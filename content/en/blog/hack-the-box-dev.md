+++
author = "Nick"
date = 2022-03-01T18:38:12Z
description = ""
draft = true
slug = "hack-the-box-dev"
title = "Hack the Box - Devzat"

+++


Welcome back! Today we're going to walk through the Hack the Box machine - Devzat. This machine is listed as a Medium Linux machine. Let's jump in!

First we kick it off with our standard `nmap` scan. Here are our results:

```
Nmap scan report for 10.10.11.118
Host is up (0.055s latency).
Not shown: 65532 closed ports
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.2 (Ubuntu Linux; protocol 2.0)
80/tcp   open  http    Apache httpd 2.4.41
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: Did not follow redirect to http://devzat.htb/
8000/tcp open  ssh     (protocol 2.0)
| fingerprint-strings:
|   NULL:
|_    SSH-2.0-Go
| ssh-hostkey:
|_  3072 6a:ee:db:90:a6:10:30:9f:94:ff:bf:61:95:2a:20:63 (RSA)
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port8000-TCP:V=7.80%I=7%D=3/1%Time=621E67FD%P=x86_64-pc-linux-gnu%r(NUL
SF:L,C,"SSH-2\.0-Go\r\n");
Service Info: Host: devzat.htb; OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

We see a basic set of ports open, `SSH` and `HTTP`. Let's take a look at what's being hosted on `80` and `8000`.



