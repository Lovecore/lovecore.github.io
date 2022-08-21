+++
author = "Nick"
categories = ["hack the box", "gobuster", "nosql", "sqli", "SQL Injection", "burpsuite", "ssh", "linpeas", "GTFObins"]
date = 2020-04-18T13:53:15Z
description = ""
draft = false
thumbnail = "/images/2019/12/mango.png"
slug = "hack-the-box-mango"
tags = ["hack the box", "gobuster", "nosql", "sqli", "SQL Injection", "burpsuite", "ssh", "linpeas", "GTFObins"]
title = "Hack the Box - Mango"

+++


Welcome back! Today we are going to be doing the Hack the Box machine - Mango. Mango is a medium Linux box. Let's jump in!

As usual we start with our ```nmap``` scan: ```nmap -sC -sV -T4 -p- -oA all_ports 10.10.10.162```

Here are our results:
```
Nmap scan report for 10.10.10.162
Host is up (0.054s latency).
Not shown: 65532 closed ports
PORT    STATE SERVICE  VERSION
22/tcp  open  ssh      OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 a8:8f:d9:6f:a6:e4:ee:56:e3:ef:54:54:6d:56:0c:f5 (RSA)
|   256 6a:1c:ba:89:1e:b0:57:2f:fe:63:e1:61:72:89:b4:cf (ECDSA)
|_  256 90:70:fb:6f:38:ae:dc:3b:0b:31:68:64:b0:4e:7d:c9 (ED25519)
80/tcp  open  http     Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: 403 Forbidden
443/tcp open  ssl/http Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Mango | Search Base
| ssl-cert: Subject: commonName=staging-order.mango.htb/organizationName=Mango Prv Ltd./stateOrProvinceName=None/countryName=IN
| Not valid before: 2019-09-27T14:21:19
|_Not valid after:  2020-09-26T14:21:19
|_ssl-date: TLS randomness does not represent time
| tls-alpn: 
|_  http/1.1
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

We see there is a hostname leaked in our enumeration. So we'll add ```mango.htb``` and ```staging-order.mango.htb``` to our hosts list and visit port 80.

![](/images/2019/12/image-33.png)

We are greeted with a Google clone. We take a look at the source code we see ```analytics.php```. So knowing this we'll point ```gobuster``` at the site and see what results might come back. While that runs we take a look at ```staging-order.mango.htb```. We are greeted with a login page. Looking at the source there doesn't seem to be much. During our fuzzing of the login field we see that it does seem vulnerable to [```noSQL``` injection](hhttps://book.hacktricks.xyz/pentesting-web/nosql-injection). Here is a quick list of [```noSQL injections```](https://github.com/cr0hn/nosqlinjection_wordlists/blob/master/mongodb_nosqli.txt).

When we use ```[$ne]``` to bypass the login page, we get an under construction landing page. We can assume this is the page we get when we have access, since we did bypass it. So now that we know a way forward, we need to extract some data. Based on the link above we have a way of doing that.

![](/images/2019/12/mango_extract_nosql.png)

As we can see, we can guess each character of the username and password. Based on the box, I'd guess our username is either mango or admin. So we'll set our username to admin and use ```Burpsuite's``` sniper function to brute these characters one at a time.  A script might have been a bit faster but this works as well.

![](/images/2019/12/mango_burp_sniper.png)

Error 200 means that the character was NOT accepted. While a 302 error means it was correct. We just keep appending our chracters as they are found to create a full password for the admin user.

![](/images/2019/12/mango_burp_extract_2.png)

As you can see in the above image, we've appended the previously found 't' to our url. Rerun the attack and our next character is 9. We'll repeat this process until we have a full password: ```t9KcS3>!0B#2```. We repeat the process for user ```mango``` as well. Turns out there is a user with password of ```h3mXK8RhU~f{]f5H```.

We now have two pairs of credentials. The only other place we found that we can use them is ```SSH```. Turns out the credentials for ```mango``` work.

![](/images/2019/12/mango_ssh.gif)

Once we log in as ```mango``` we take a peek for ```user.txt``` but we don't see it. We look at the ```/home``` directory and see we only have two users, as we saw before. We issues a ```su admin``` and enter the admin password and poof, we are admin!

![](/images/2019/12/mango_su.gif)

Now we have our userflag. We need to start enumerating. We spin up our ```SimpleHTTPServer``` and get ```linenum``` and ```linpeas``` onto our target. A quick run of ```linpeas``` shows us our path forward!

![](/images/2019/12/mano_jjs.png)

A quick look over at [GTFObins](https://gtfobins.github.io/#jj) has our answer. We can use the read file portion to snag ```root.txt```!

![](/images/2019/12/mano_read_root.gif)

We can also give ourselves a root shell while we're at it. We can do this by copying ```bash``` to ```/tmp```. Setting the SUID bit with ```chmod +s``` then calling bash with ```-p``` as per GTFO. We just need to do this in a few separate commands.

![](/images/2019/12/mango_root_suid.gif)

Hopefully something was learned. If you found this write-up helpful, consider sending some respect my way: [Lovecore's HTB Profile](https://www.hackthebox.eu/home/users/profile/95635).

