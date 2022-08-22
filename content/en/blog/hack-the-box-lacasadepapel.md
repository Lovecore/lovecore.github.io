+++
author = "Nick"
categories = ["CTF", "enumeration", "reverse shell", "web services"]
date = 2019-07-27T15:43:38Z
description = ""
draft = false
thumbnail = "/images/2019/07/info.png"
slug = "hack-the-box-lacasadepapel"
summary = "Welcome back to another HTB write up. Today we'll be doing the box LaCasaDePapel or the House of Paper. Lets jump in!"
tags = ["CTF", "enumeration", "reverse shell", "web services"]
title = "Hack The Box - LaCasaDePapel"
url = "/hack-the-box-lacasadepapel"

+++


Welcome back to another HTB write up. Today we'll be doing the box LaCasaDePapel or the House of Paper. Lets jump in!

As always, we kick it off with an nmap.

```bash
nmap -sV -sC -oA ./scan 10.10.10.131
```

Here's what we get back, a fairly normal port scan.

````bash
Host is up (0.22s latency).
Not shown: 996 closed ports
PORT    STATE SERVICE  VERSION
21/tcp  open  ftp      vsftpd 2.3.4
22/tcp  open  ssh      OpenSSH 7.9 (protocol 2.0)
| ssh-hostkey: 
|   2048 03:e1:c2:c9:79:1c:a6:6b:51:34:8d:7a:c3:c7:c8:50 (RSA)
|   256 41:e4:95:a3:39:0b:25:f9:da:de:be:6a:dc:59:48:6d (ECDSA)
|_  256 30:0b:c6:66:2b:8f:5e:4f:26:28:75:0e:f5:b1:71:e4 (ED25519)
80/tcp  open  http     Node.js (Express middleware)
|_http-title: La Casa De Papel
443/tcp open  ssl/http Node.js Express framework
| ssl-cert: Subject: commonName=lacasadepapel.htb/organizationName=La Casa De Papel
| Not valid before: 2019-01-27T08:35:30
|_Not valid after:  2029-01-24T08:35:30
|_ssl-date: TLS randomness does not represent time
| tls-nextprotoneg: 
|   http/1.1
|_  http/1.0
Service Info: OS: Unix

```

Let's take a look at the running web services. Browse to the site and we are greeted with a QR code.. interesting. Let's peek the source. In the source we see the URL for the QR Token and that's about it.

![](/images/2019/07/image-23.png" caption="I've never dealt with QR codes and token's before. Better get to googling...)

When we look at what is running on 443, we are greeted with an image that says Certificate Error. So we need some type of pair to continue maybe? Lets keep looking at what we have. A quick searchsploit for vsftpd shows that we do have 1 exploit to potentially try.

![](/images/2019/07/image-24.png" caption="The -w will print out the URL for the exploit as well.)

[S](http://10.10.10.131/qrcode?qrurl=/root/)o we give it a shot. When this is exploit is run, it opens a debugging port on 6200. So we telnet to it and see what we have.

![](/images/2019/07/image-77.png" caption="Some type of PHP shell.)

We take a peek around but we only see one item. A variable called Tokyo. When we view it we get some code that references a certificate.

![](/images/2019/07/image-78.png)

We can try and see what the ca.key has in store by using edit. ```bash edit -e /home/nairobica.key```

Now that we have a private key, what good is it to us? We know that from enumerating before we have content on port 80 and 443. Port 80 seems like a rabbit hole, however, port 443 asks us for a valid certificate to enter. This could be our opening!

What we'll need to do is create a public cert from the private key we just obtained. We will issue the following: ```bash openssl req -key /privatekey.txt -new -x509 -days 365 -out casa.crt```. ONce we've done that, we get our .crt, however, we need to convert this to a PKCS format from PEM. So we do the following ```bash openssl pkcs12 -inkey casa.crt -export -out new_casa.pfx```. Now that we have a valid certificate that FireFox will take, we import it and head back to the site.

![](/images/2019/07/image-79.png)

We are now greeted with a new landing. Season 1 and Season 2. When we look at the links, we see that Season-2 is using ?PATH=. This would lead me to believe we can use directory traversal with ../../../ lets see what we get. Sure enough we get the output of the /etc directory! Using this we can path around and see what we have and we do see a .ssh file in /berlin/ but how can we view it? If you took a look around the site you will notice that each episode of Season 2 has what looks like base64 encoding:

![](/images/2019/07/image-82.png)

So it's possible we can just create an echo command then convert it to base63 and plug it into our web browser!

![](/images/2019/07/image-81.png" caption="We want the contents of the id_rsa file!)

![](/images/2019/07/image-83.png)

Now that we have a key, who does it work with? We can use the same method as above to get the contents of the passwd file, this should give us some insite.

![](/images/2019/07/image-84.png)

We have a few users, berlin, dali and professor. We try to SSH for each user and see what works. Turns out, its professor.

![](/images/2019/07/image-85.png)

We run run LinEnum and PsPy to get an idea of whats going on in the box. We see a cron job executing as root calling memcached.ini.

![](/images/2019/07/image-87.png)

We modify that .ini to forward out a netcat connection and wait...

![](/images/2019/07/image-86.png)

We snag our root flag and we are done!

I actually could not find the user.txt on this box. I'm pretty sure this is the down side of free services. Someone nuked it...

Edit: Sure enough, I went switched to the EU machine pool and was able to get the user.txt via the base64 file traversal method above.

Hopefully something was learned. If you found this write-up helpful, consider sending some respect my way: [Lovecore's HTB Profile](https://www.hackthebox.eu/home/users/profile/95635).

