+++
author = "Nick"
categories = ["hack the box", "Linux", "osTicket", "ssh", "mattermost", "mysql", "hashcat", "john"]
date = 2021-05-23T11:36:50Z
description = ""
draft = false
thumbnail = "/images/2021/05/delivery.png"
slug = "hack-the-box-delivery"
tags = ["hack the box", "Linux", "osTicket", "ssh", "mattermost", "mysql", "hashcat", "john"]
title = "Hack the Box - Delivery"

+++


Welcome back! Today we are going to walk through the Hack the Box machine - Delivery. This box is listed as an Easy Linux machine, let's jump in!

As always, we kick it off with an `nmap` scan: nmap -sC -sV -p- -oA allscan 10.10.10.222

Here are the results:
```
Host is up (0.048s latency).
Not shown: 65532 closed ports
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey: 
|   2048 9c:40:fa:85:9b:01:ac:ac:0e:bc:0c:19:51:8a:ee:27 (RSA)
|   256 5a:0c:c0:3b:9b:76:55:2e:6e:c4:f4:b9:5d:76:17:09 (ECDSA)
|_  256 b7:9d:f7:48:9d:a2:f2:76:30:fd:42:d3:35:3a:80:8c (ED25519)
80/tcp   open  http    nginx 1.14.2
|_http-server-header: nginx/1.14.2
|_http-title: Welcome
8065/tcp open  unknown
| fingerprint-strings: 
|   GenericLines, Help, RTSPRequest, SSLSessionReq, TerminalServerCookie: 
|     HTTP/1.1 400 Bad Request
|     Content-Type: text/plain; charset=utf-8
|     Connection: close
|     Request
|   GetRequest: 
|     HTTP/1.0 200 OK
|     Accept-Ranges: bytes
|     Cache-Control: no-cache, max-age=31556926, public
|     Content-Length: 3108
|     Content-Security-Policy: frame-ancestors 'self'; script-src 'self' cdn.rudderlabs.com
|     Content-Type: text/html; charset=utf-8
|     Last-Modified: Wed, 05 May 2021 14:40:04 GMT
|     X-Frame-Options: SAMEORIGIN
|     X-Request-Id: tbkykq9no7yqzmw46oepnhfy8c
|     X-Version-Id: 5.30.0.5.30.1.57fb31b889bf81d99d8af8176d4bbaaa.false
|     Date: Wed, 05 May 2021 20:46:01 GMT
|     <!doctype html><html lang="en"><head><meta charset="utf-8"><meta name="viewport" content="width=device-width,initial-scale=1,maximum-scale=1,user-scalable=0"><meta name="robots" content="noindex, nofollow"><meta name="referrer" content="no-referrer"><title>Mattermost</title><meta name="mobile-web-app-capable" content="yes"><meta name="application-name" content="Mattermost"><meta name="format-detection" content="telephone=no"><link re
|   HTTPOptions: 
|     HTTP/1.0 405 Method Not Allowed
|     Date: Wed, 05 May 2021 20:46:01 GMT
|_    Content-Length: 0
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port8065-TCP:V=7.91%I=7%D=5/5%Time=6093017C%P=x86_64-pc-linux-gnu%r(Gen
SF:ericLines,67,"HTTP/1\.1\x20400\x20Bad\x20Request\r\nContent-Type:\x20te
SF:xt/plain;\x20charset=utf-8\r\nConnection:\x20close\r\n\r\n400\x20Bad\x2
SF:0Request")%r(GetRequest,DF3,"HTTP/1\.0\x20200\x20OK\r\nAccept-Ranges:\x
SF:20bytes\r\nCache-Control:\x20no-cache,\x20max-age=31556926,\x20public\r
SF:\nContent-Length:\x203108\r\nContent-Security-Policy:\x20frame-ancestor
SF:s\x20'self';\x20script-src\x20'self'\x20cdn\.rudderlabs\.com\r\nContent
SF:-Type:\x20text/html;\x20charset=utf-8\r\nLast-Modified:\x20Wed,\x2005\x
SF:20May\x202021\x2014:40:04\x20GMT\r\nX-Frame-Options:\x20SAMEORIGIN\r\nX
SF:-Request-Id:\x20tbkykq9no7yqzmw46oepnhfy8c\r\nX-Version-Id:\x205\.30\.0
SF:\.5\.30\.1\.57fb31b889bf81d99d8af8176d4bbaaa\.false\r\nDate:\x20Wed,\x2
SF:005\x20May\x202021\x2020:46:01\x20GMT\r\n\r\n<!doctype\x20html><html\x2
SF:0lang=\"en\"><head><meta\x20charset=\"utf-8\"><meta\x20name=\"viewport\
SF:"\x20content=\"width=device-width,initial-scale=1,maximum-scale=1,user-
SF:scalable=0\"><meta\x20name=\"robots\"\x20content=\"noindex,\x20nofollow
SF:\"><meta\x20name=\"referrer\"\x20content=\"no-referrer\"><title>Matterm
SF:ost</title><meta\x20name=\"mobile-web-app-capable\"\x20content=\"yes\">
SF:<meta\x20name=\"application-name\"\x20content=\"Mattermost\"><meta\x20n
SF:ame=\"format-detection\"\x20content=\"telephone=no\"><link\x20re")%r(HT
SF:TPOptions,5B,"HTTP/1\.0\x20405\x20Method\x20Not\x20Allowed\r\nDate:\x20
SF:Wed,\x2005\x20May\x202021\x2020:46:01\x20GMT\r\nContent-Length:\x200\r\
SF:n\r\n")%r(RTSPRequest,67,"HTTP/1\.1\x20400\x20Bad\x20Request\r\nContent
SF:-Type:\x20text/plain;\x20charset=utf-8\r\nConnection:\x20close\r\n\r\n4
SF:00\x20Bad\x20Request")%r(Help,67,"HTTP/1\.1\x20400\x20Bad\x20Request\r\
SF:nContent-Type:\x20text/plain;\x20charset=utf-8\r\nConnection:\x20close\
SF:r\n\r\n400\x20Bad\x20Request")%r(SSLSessionReq,67,"HTTP/1\.1\x20400\x20
SF:Bad\x20Request\r\nContent-Type:\x20text/plain;\x20charset=utf-8\r\nConn
SF:ection:\x20close\r\n\r\n400\x20Bad\x20Request")%r(TerminalServerCookie,
SF:67,"HTTP/1\.1\x20400\x20Bad\x20Request\r\nContent-Type:\x20text/plain;\
SF:x20charset=utf-8\r\nConnection:\x20close\r\n\r\n400\x20Bad\x20Request");
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 110.71 seconds
```

There is some interesting items here. We see that port 80 and 8065 are open. We'll browse these ports to see what they have.

![](/images/2021/05/image-7.png" caption="Landing page for port 80)

We see a generic page for some kind of IT support service. The first thing we notice is that the link points to helpdesk.delivery.htb. Viewing the source also shows us http://delivery.htb:8065 as well. We'll add these to our hosts list. After we add them, we click the link that takes us to helpdesk.delivery.htb.

We are now greeted with a support center ticketing system. This system is run on `osTicket`. It looks like we can create a ticket without an account. We can also go back and view the ticket without an account. When we try to register for an account, it emails us a link to activate.

Now we'll head over to the items being hosted on 8065 to see if they require eachother.

![](/images/2021/05/image-8.png)

Here it looks like we are able to create an account for delivery.htb, or atleast some service associated with it. We create an account and it tells us to check our inbox for the link.

How do we get this registration link? After toying around, it looks like we can actually create a Mattermost account associated to the ticket email. Then, when the ticket is created, the email is sent to the ticket account and appended to the currently open ticket.

![](/images/2021/05/image-9.png)

We can then copy the verification link out of the ticket, and verify our account.

![](/images/2021/05/image-10.png)

Now we can login with the credentials we made.

![](/images/2021/05/image-11.png)

We can then click internal and view some internal systems. When we get logged in we see some credentials posted in the clear `maildeliverer:Youve_G0t_Mail!`. We try to leverage these via `ssh` and we are in!

![](/images/2021/05/image-12.png)

Snag the `user.txt` flag and start enumerating! We download `linpeas.sh` to our target and start enumerating. Nothing really sticks out, so we'll have to manually dig around. One item that is of interest is `/opt/mattermost`

![](/images/2021/05/image-13.png)

Atleast we have a starting point. Sifting through the config files, we find some credentials: `elastic:changeme` and `mmuser:Crack_The_MM_Admin_PW`. Now we can try to connect to this SQL instance with these credentials.

Command:
`mysql -u mmuser -p'Crack_The_MM_Admin_PW'`

![](/images/2021/05/image-14.png)

Awesome, a database connection. Now we can enumerate within it.

Commands:
```mysql
show databases;
use mattermost;
show tables;
describe Users;
select * from Users\G;
```

![](/images/2021/05/sql_delivery.gif)

Now we have a `root` username and a password hash. Now we know from gaining entry to the internal site that this password is not on the `RockYou` password list. However, it does say that if we are clever enough, we can use `hashcat` rules to easily crack variations. Here are two good resources for `hashcat` rules: [one](https://notsosecure.com/one-rule-to-rule-them-all/) and [two](https://www.4armed.com/blog/hashcat-rule-based-attack/).

Using a rule within `hashcat` is pretty simple. We just supply the `-r` flag. We also need to specify the `--stdout` option and send that out content to a text file. So in all this is what we have:

`cat password.txt | ./hashcat -r OneRuleToRuleThemAll.rule --force --backend-ignore-cuda --stdout > root_combo.lst`

There are some other flags in here since running `hashcat` in a VM on new Ryzen silicone was a pain.

Now with our new wordlist, we simply supply it to `John` and let it do it's work.

Command:
`john -w=root_combo.lst hash.out`

![](/images/2021/05/image-15.png)

Now we have a password for the `root` `sql` user. What are the chances there is some password reuse?

Command:
`su root`

![](/images/2021/05/image-16.png)

There we have it, the `root` flag! I hope everyone found this box fun, be sure to send Ippsec some love for all his contributions to the community!

Also, if you found this write-up useful, send me some respect over on HTB:
https://app.hackthebox.eu/profile/95635



