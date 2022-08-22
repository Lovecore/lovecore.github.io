+++
author = "Nick"
categories = ["basic", "CTF", "enumeration", "NetSec", "cron", "hack the box"]
date = 2019-10-12T10:55:44Z
description = ""
draft = false
thumbnail = "/images/2019/07/info-3.png"
slug = "hack-the-box-writeup"
summary = "Welcome back! Today we're doing the box Writeup. Let's jump in!"
tags = ["basic", "CTF", "enumeration", "NetSec", "cron", "hack the box"]
title = "Hack the Box - Writeup"
url = "/hack-the-box-writeup"

+++


Welcome back! Today we're doing the box Writeup. Let's jump in!

As always, the first thing we do is run our standard nmap scan:

```
nmap -sC -sV -oA ./writeupscan 10.10.10.138
```

We get back some pretty limited results:

```
Nmap scan report for 10.10.10.138
Host is up (0.055s latency).
Not shown: 998 filtered ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.4p1 Debian 10+deb9u6 (protocol 2.0)
| ssh-hostkey: 
|   2048 dd:53:10:70:0b:d0:47:0a:e2:7e:4a:b6:42:98:23:c7 (RSA)
|   256 37:2e:14:68:ae:b9:c2:34:2b:6e:d9:92:bc:bf:bd:28 (ECDSA)
|_  256 93:ea:a8:40:42:c1:a8:33:85:b3:56:00:62:1c:a0:ab (ED25519)
80/tcp open  http    Apache httpd 2.4.25 ((Debian))
| http-robots.txt: 1 disallowed entry 
|_/writeup/
|_http-server-header: Apache/2.4.25 (Debian)
|_http-title: Nothing here yet.
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

So we head over to the port 80 and we are greeted with a basic page.

![](/images/2019/07/image-55.png)

Well, write away we are told that error 40x series will be logged and banned. So this would rule out something like dirbuster. However, we try anyway! Sure enough, it does stop parsing after maybe 20 attempts. So we aren't going to be enumerating that way. Our nmap scan did show us another directory called /writeup/, lets see what's there.

![](/images/2019/07/image-56.png)

Now we have a different page with some links. We see that there is a link to an external site and some linking to internal sites using PHP. Maybe we can break that functionality. When we take a peek at the code, we see that it's running on CMS Made Simple. So we do some quick googling on the product and see what it's all about. In my research I came across a [SQL injection script](https://packetstormsecurity.com/files/152356/CMS-Made-Simple-SQL-Injection.html) on Packet Storm. Let's give it a try. The first thing we do is copy paste the script into a .py file and save it. If we try to run it, we will get an error telling us we don't have the termcolor library. Lets fix that quickly.

The first thing to do is install python-pip. This does come packaged with Kali but if you aren't using Kali, you will want to run: ```apt install python-pip```. Once we have pip operational we issue ```pip install termcolor```. Now we should be able to run the script without error.

![](/images/2019/07/image-71.png" caption="Quite a pretty output process.)

So now we have a salt, username, email and hash. Now we have a few options. We can toss rockyou.txt at it via the script or we can toss just the hash online and see what comes back. Decrypting with rockyou takes a while, but it will return the value you want. Now if you decrypt the hash online you get this:

![](/images/2019/07/image-73.png)

What we see is actually the salt found before in front of a some text. I'm willing to bet this is the password we want to be using. We can use the credentials we found and try and log into the admin page but that doesn't work. So we can try to use them to log in via SSH and it works!

![](/images/2019/07/image-72.png)

We land right in jkr's user directory and snag the user.txt. Time to move onto root. As I was doing some basic moving around the box, I noticed that I could get to quite a few places. We'll download LinEnum and see what comes back.

We spin up a SimpleHTTPServer by issueing ```python -m SimpleHTTPServer```. Next we just wget the file while we are ssh'd into the box.

![](/images/2019/07/image-74.png)

We get a bunch of usefull data back. In addition to this, we should also run [PsPy](https://github.com/DominicBreuker/pspy) for further enumeration. With these two combined hopefully we can find a crack in the wall. Using the same method as above, we'll download pspy to the machine and run it.

![](/images/2019/07/image-75.png)

We let it run for a little bit. Another thing we should also do is attempt to SSH back into the machine while this is running to see if it catches anything on login or logout of the sessions.

Now that we've enumerated a bit, and sift through the results we can pick out some things that are interesting.

![](/images/2019/07/image-76.png)

One thing we notice is that on login / logout a pearl script is called to clean up the session. After we run LinEnum we also see that there is a cron job running  run-parts to do some things for us.  Our path also has some pretty wild path entries. I'm fairly confident we can use this to our advantage. The first thing we're going to want to do is append our malious path location to the local path.

![](/images/2019/07/image-88.png)

Next we're going to create a small little script to copy the root flag to our malicous directory.

![](/images/2019/07/image-89.png)

After that we're going to make a few copies of it and put them in accessable paths.

![](/images/2019/07/image-90.png)

These will get placed in the /usr/local/sbin/. Then when our cron job runs the run-parts from the prompt given, we will have our malicous script run instead.

![](/images/2019/07/image-91.png" caption="This is the job that is running as root we are looking to abuse. We see it specifies the path in the call.)

Once the job runs, we get what we are after!

![](/images/2019/07/image-92.png)

Now in this case, we could replace that shell code with a more malicous code, like a back door and backdoor as root rather than just copy paste the flag.

Another box down!Hopefully something was learned. If you found this write-up helpful, consider sending some respect my way: [Lovecore's HTB Profile](https://www.hackthebox.eu/home/users/profile/95635).

