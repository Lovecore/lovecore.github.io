+++
author = "Nick"
categories = ["basic", "CTF", "enumeration", "netcat", "python", "reverse shell", "web services"]
date = 2019-09-28T15:50:08Z
description = ""
draft = false
image = "__GHOST_URL__/content/images/2019/09/info.png"
slug = "hack-the-box-swagshop"
summary = "Welcome back! Today we're going to tackle the box SwagShop on Hack the Box. Lets get to it!"
tags = ["basic", "CTF", "enumeration", "netcat", "python", "reverse shell", "web services"]
title = "Hack The Box - SwagShop"

+++


Welcome back! Today we're going to tackle the box SwagShop on Hack the Box. Lets get to it!

As always, we kick it off with our standard nmap.

```
nmap -sC -sV -oA ./swagshop 10.10.10.140
```

```
Nmap scan report for 10.10.10.140
Host is up (0.18s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 b6:55:2b:d2:4e:8f:a3:81:72:61:37:9a:12:f6:24:ec (RSA)
|   256 2e:30:00:7a:92:f0:89:30:59:c1:77:56:ad:51:c0:ba (ECDSA)
|_  256 4c:50:d5:f2:70:c5:fd:c4:b2:f0:bc:42:20:32:64:34 (ED25519)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Home page
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
```

We see http and ssh, so lets see whats on port 80. We are greeted with a Magento install. Looks to be an old one at that. It's fairly well known that Magento has some good exploits running around out there. Lets use searchsploit to find some.

![](/images/2019/07/image-63.png" caption="We see a few.)

We see one that has a remote code execution that will create a normal user. Lets give that a shot.

![](/images/2019/07/image-64.png" caption="We can use the -m and the path to copy that file to our current directory.)

We'll need to modify the code a little to get it to work on our target. Once we've done that we can run it against our target to see if it works.

![](/images/2019/07/image-61.png)

It does! Now that we're a  user, lets see how we can create a host session. There are some remote authenticated code executions we could use, but we want simple and persistant. Some googling around lead me to [LavaMagentoBD](https://github.com/lavalamp-/LavaMagentoBD). Lets give it a shot.

![](/images/2019/07/image-57.png)

Now that we've done that, we need to modify the IndexController.php to give ourselves a backdoor. In this case we'll just put in the classic reverse shell php file.

We'll need to change our hash in the xml and repackage it.

![](/images/2019/07/image-58.png)

Then we head over to the downloader page and upload our new package as a module.

![](/images/2019/07/image-62.png)

We start up our netcat to catch our incoming connections. Then head over to our uploaded path to kick the backdoor. [http://10.10.10.140/app/code/community/Lavalamp/Connector/controllers/IndexController.php](http://10.10.10.140/app/code/community/Lavalamp/Connector/controllers/IndexController.php). Once we do, we see our Netcat light up!

![](/images/2019/07/image-65.png)

Now we can path to home to see what we have. We see an account called haris, we move to it and cat the contents of user.txt!

![](/images/2019/07/image-66.png)

So how do we escalate from here? We should probably run [LinEnum](https://github.com/rebootuser/LinEnum) but before I do that, I usually test what we can do. I can't sudo, invoke crons or any other fairly common task.

![](/images/2019/07/image-67.png)

However, what I can do is call Vi on items in /var/www/html/! You can use Vi to gain the root.txt contents.

![](/images/2019/07/image-68.png)

So here we issue the command:
```
sudo vi /var/www/html/index.php -c ':!/bin/sh' /dev/null
```
This will load vi as a super user. The -c tells vi we want to issue a command as well. We tell it that we want to spawn a bash shell. Once we enter this, we see an error about Term entry and some build error, but at the bottom we see our shell prompt! To confirm it worked, we issue ``` whoami ```. Sure enough, we are root!

![](/images/2019/07/image-69.png)

From here we just cat /root/root.txt and bingo! CTF complete.

![](/images/2019/07/image-70.png)

All in all this was a pretty straight forward box. I did have some issues with the magecart plugin's not working correctly, so it took me a few reboots until they seemed to get flushed out, oddly buggy, but still fun!

Hopefully something was learned. If you found this write-up helpful, consider sending some respect my way: [Lovecore's HTB Profile](https://www.hackthebox.eu/home/users/profile/95635).

