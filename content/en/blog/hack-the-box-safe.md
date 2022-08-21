+++
author = "Nick"
categories = ["hack the box", "python", "Buffer Overflow", "keypass", "john", "scp", "kpcli", "ropstar", "ssh", "ssh-keygen"]
date = 2019-10-26T15:02:05Z
description = ""
draft = false
thumbnail = "/images/2019/10/info.png"
slug = "hack-the-box-safe"
summary = "Welcome back! Today we are going to be doing the box Safe on Hack the Box. The box is listed as an easy box. Lets jump in!"
tags = ["hack the box", "python", "Buffer Overflow", "keypass", "john", "scp", "kpcli", "ropstar", "ssh", "ssh-keygen"]
title = "Hack the Box - Safe"

+++


Welcome back! Today we are going to be doing the machine Safe on Hack the Box. The box is listed as an easy box. Lets jump in!

We always start our enumeration with the standard nmap scan: ```nmap -sC -sV -oA safe_scan 10.10.10.147```. We get back a limited set of results. 
```
Nmap scan report for 10.10.10.147
Host is up (0.055s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.4p1 Debian 10+deb9u6 (protocol 2.0)
| ssh-hostkey: 
|   2048 6d:7c:81:3d:6a:3d:f9:5f:2e:1f:6a:97:e5:00:ba:de (RSA)
|   256 99:7e:1e:22:76:72:da:3c:c9:61:7d:74:d7:80:33:d2 (ECDSA)
|_  256 6a:6b:c3:8e:4b:28:f7:60:85:b1:62:ff:54:bc:d8:d6 (ED25519)
80/tcp open  http    Apache httpd 2.4.25 ((Debian))
|_http-server-header: Apache/2.4.25 (Debian)
|_http-title: Apache2 Debian Default Page: It works
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 9.72 seconds
```
So we will scan it again against all ports, ```nmap -sC -sV -T5 -p- -oA safe_all_ports 10.10.10.147```.

```
Nmap scan report for 10.10.10.147
Host is up (0.055s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.4p1 Debian 10+deb9u6 (protocol 2.0)
| ssh-hostkey: 
|   2048 6d:7c:81:3d:6a:3d:f9:5f:2e:1f:6a:97:e5:00:ba:de (RSA)
|   256 99:7e:1e:22:76:72:da:3c:c9:61:7d:74:d7:80:33:d2 (ECDSA)
|_  256 6a:6b:c3:8e:4b:28:f7:60:85:b1:62:ff:54:bc:d8:d6 (ED25519)
80/tcp open  http    Apache httpd 2.4.25 ((Debian))
|_http-server-header: Apache/2.4.25 (Debian)
|_http-title: Apache2 Debian Default Page: It works
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 9.72 seconds
┌─[user@parrot]─[~/Documents/htb/safe]
└──╼ $nmap -sC -sV -T5 -p- -oA safe_all_ports 10.10.10.147
Starting Nmap 7.80 ( https://nmap.org ) at 2019-09-30 15:01 EDT
Warning: 10.10.10.147 giving up on port because retransmission cap hit (2).
Segmentation fault
┌─[✗]─[user@parrot]─[~/Documents/htb/safe]
└──╼ $nmap -sC -sV -T4 -p- -oA safe_all_ports 10.10.10.147
Starting Nmap 7.80 ( https://nmap.org ) at 2019-09-30 15:09 EDT
Nmap scan report for 10.10.10.147
Host is up (0.054s latency).
Not shown: 65532 closed ports
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 7.4p1 Debian 10+deb9u6 (protocol 2.0)
| ssh-hostkey: 
|   2048 6d:7c:81:3d:6a:3d:f9:5f:2e:1f:6a:97:e5:00:ba:de (RSA)
|   256 99:7e:1e:22:76:72:da:3c:c9:61:7d:74:d7:80:33:d2 (ECDSA)
|_  256 6a:6b:c3:8e:4b:28:f7:60:85:b1:62:ff:54:bc:d8:d6 (ED25519)
80/tcp   open  http    Apache httpd 2.4.25 ((Debian))
|_http-server-header: Apache/2.4.25 (Debian)
|_http-title: Apache2 Debian Default Page: It works
1337/tcp open  waste?
| fingerprint-strings: 
|   DNSStatusRequestTCP: 
|     15:10:45 up 6:41, 0 users, load average: 0.47, 0.68, 0.63
|   DNSVersionBindReqTCP: 
|     15:10:40 up 6:41, 0 users, load average: 0.51, 0.69, 0.63
|   GenericLines: 
|     15:10:28 up 6:41, 0 users, load average: 0.52, 0.70, 0.63
|     What do you want me to echo back?
|   GetRequest: 
|     15:10:35 up 6:41, 0 users, load average: 0.56, 0.70, 0.64
|     What do you want me to echo back? GET / HTTP/1.0
|   HTTPOptions: 
|     15:10:35 up 6:41, 0 users, load average: 0.56, 0.70, 0.64
|     What do you want me to echo back? OPTIONS / HTTP/1.0
|   Help: 
|     15:10:50 up 6:41, 0 users, load average: 0.43, 0.67, 0.63
|     What do you want me to echo back? HELP
|   NULL: 
|     15:10:28 up 6:41, 0 users, load average: 0.52, 0.70, 0.63
|   RPCCheck: 
|     15:10:35 up 6:41, 0 users, load average: 0.56, 0.70, 0.64
|   RTSPRequest: 
|     15:10:35 up 6:41, 0 users, load average: 0.56, 0.70, 0.64
|     What do you want me to echo back? OPTIONS / RTSP/1.0
|   SSLSessionReq, TerminalServerCookie: 
|     15:10:50 up 6:41, 0 users, load average: 0.43, 0.67, 0.63
|     What do you want me to echo back?
|   TLSSessionReq: 
|     15:10:51 up 6:41, 0 users, load average: 0.43, 0.67, 0.63
|_    What do you want me to echo back?
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port1337-TCP:V=7.80%I=7%D=9/30%Time=5D9252FD%P=x86_64-pc-linux-gnu%r(NU
SF:LL,3E,"\x2015:10:28\x20up\x20\x206:41,\x20\x200\x20users,\x20\x20load\x
SF:20average:\x200\.52,\x200\.70,\x200\.63\n")%r(GenericLines,63,"\x2015:1
SF:0:28\x20up\x20\x206:41,\x20\x200\x20users,\x20\x20load\x20average:\x200
SF:\.52,\x200\.70,\x200\.63\n\nWhat\x20do\x20you\x20want\x20me\x20to\x20ec
SF:ho\x20back\?\x20\r\n")%r(GetRequest,71,"\x2015:10:35\x20up\x20\x206:41,
SF:\x20\x200\x20users,\x20\x20load\x20average:\x200\.56,\x200\.70,\x200\.6
SF:4\n\nWhat\x20do\x20you\x20want\x20me\x20to\x20echo\x20back\?\x20GET\x20
SF:/\x20HTTP/1\.0\r\n")%r(HTTPOptions,75,"\x2015:10:35\x20up\x20\x206:41,\
SF:x20\x200\x20users,\x20\x20load\x20average:\x200\.56,\x200\.70,\x200\.64
SF:\n\nWhat\x20do\x20you\x20want\x20me\x20to\x20echo\x20back\?\x20OPTIONS\
SF:x20/\x20HTTP/1\.0\r\n")%r(RTSPRequest,75,"\x2015:10:35\x20up\x20\x206:4
SF:1,\x20\x200\x20users,\x20\x20load\x20average:\x200\.56,\x200\.70,\x200\
SF:.64\n\nWhat\x20do\x20you\x20want\x20me\x20to\x20echo\x20back\?\x20OPTIO
SF:NS\x20/\x20RTSP/1\.0\r\n")%r(RPCCheck,3E,"\x2015:10:35\x20up\x20\x206:4
SF:1,\x20\x200\x20users,\x20\x20load\x20average:\x200\.56,\x200\.70,\x200\
SF:.64\n")%r(DNSVersionBindReqTCP,3E,"\x2015:10:40\x20up\x20\x206:41,\x20\
SF:x200\x20users,\x20\x20load\x20average:\x200\.51,\x200\.69,\x200\.63\n")
SF:%r(DNSStatusRequestTCP,3E,"\x2015:10:45\x20up\x20\x206:41,\x20\x200\x20
SF:users,\x20\x20load\x20average:\x200\.47,\x200\.68,\x200\.63\n")%r(Help,
SF:67,"\x2015:10:50\x20up\x20\x206:41,\x20\x200\x20users,\x20\x20load\x20a
SF:verage:\x200\.43,\x200\.67,\x200\.63\n\nWhat\x20do\x20you\x20want\x20me
SF:\x20to\x20echo\x20back\?\x20HELP\r\n")%r(SSLSessionReq,64,"\x2015:10:50
SF:\x20up\x20\x206:41,\x20\x200\x20users,\x20\x20load\x20average:\x200\.43
SF:,\x200\.67,\x200\.63\n\nWhat\x20do\x20you\x20want\x20me\x20to\x20echo\x
SF:20back\?\x20\x16\x03\n")%r(TerminalServerCookie,63,"\x2015:10:50\x20up\
SF:x20\x206:41,\x20\x200\x20users,\x20\x20load\x20average:\x200\.43,\x200\
SF:.67,\x200\.63\n\nWhat\x20do\x20you\x20want\x20me\x20to\x20echo\x20back\
SF:?\x20\x03\n")%r(TLSSessionReq,64,"\x2015:10:51\x20up\x20\x206:41,\x20\x
SF:200\x20users,\x20\x20load\x20average:\x200\.43,\x200\.67,\x200\.63\n\nW
SF:hat\x20do\x20you\x20want\x20me\x20to\x20echo\x20back\?\x20\x16\x03\n");
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 124.86 seconds
```

It looks like port 1337 is echoing back our nmap command with the text ```what do you want me to echo back?```. Interesting, let's see what is being hosted on port 80.

We see the default apache2 install page. So we'll kick off a ```dirb``` scan and enumerate the site more. ```dirb http://10.10.10.147``` will run the common word list against the site. While that run's we will check the page source on the apache install to see what might be there.

![](/images/2019/09/image-83.png)

We now know that we can download ```myapp``` from this location. Lets ```wget``` get it and see what happens.

![](/images/2019/09/image-86.png)

Now that we have the file. Lets test against it. When we launch the app and enter any anything, it is echo'd back to us.

![](/images/2019/09/image-87.png)

There is a good chance this app will have a ```buffer overflow```. So lets tell ```python``` to end it 150 characters and see what our result is.

![](/images/2019/09/image-88.png)

Looks like we have a fault! So now we need to identify the offset where the overflow actually occurs. Before we do that we will check the security on the file by using ```checksec``` within ```peda```. If you are unfamiliar with PEDA check [it out here](https://github.com/longld/peda).

![](/images/2019/09/image-89.png)

This tells us the PIE is disabled. PIE are Position Independent Executables which means that Address Space Layout Randomization (ASLR) is off. So we can reliably find our offset and call it accordingly.

Now, there are two ways (probably more) forward from here. The first time I did the box, I used [ropstar](https://github.com/xct/ropstar) to get user because I was curious to how well the program functions (turns out, quite well). We'll do this method first because its one liner and you get user pretty damn fast.

The second more manual method will be outlined further down in this post.

First we need to clone the repo. ```git clone https://github.com/xct/ropstar```. Once that is done, install the requirements. ```pip install -r requirements.txt```. Then change the permissions on ```ropstar.py```. ```chmod +x ropstar.py```. We are now good to go! We are going to run the following command: 

```python ~/Tools/ropstar/ropstar.py myapp -rhost 10.10.10.147 -rport 1337```. 

Heres the breakdown:
The ```myapp``` argument is the name of the binary.
The ```-rhost``` is the remote host running the binary.
The ```-rport``` is the remote port that the application is using.

Once we run it we get some fun Hackerman content on the screen and are eventually greeted with a prompt.

![](/images/2019/09/image-90.png)

We now have an interactive remote session on the box! We head over to ```/home/user/``` and snag the user.txt file. Onto root!

We'll want create a new ssh key pair so that we can remote into the box reliably. We'll use ```ssh-keygen``` and save the file to our working directory. Next we'll create the ```authorized_keys``` file on the remote box with. Issuing ```touch /home/user/.ssh/authorized_keys```. Next we'll ```echo``` our key into the file. ```echo "KEY HERE" > /home/user/.ssh/authorized_keys```.

Now that we have a reliable ```SSH``` shell, we can enumerate a bit. We see a file in the users home directory called ```MyPasswords.kdbx```

![](/images/2019/10/image.png)

There are a bunch of JPG files and a ```KeePass``` database. First we'll download the the ```KeePass``` database. We will use ```scp``` for this. On our local host, we issue:

```scp -r -i id_rsa user@10.10.10.147:/home/user/ /home/user/Documents/htb/safe/```. 

Here's the breakdown of the command:
The ```-r``` tells scp to get all the files.
The ```-i``` tells scp what identity file to use.
The ```:/home/user``` tells scp what file(s) to get on the remote host.
The ```/home/user/Documents/htb/safe/``` is the local path to download the files.

Now if we try to open the ```MyPasswords.kdbx``` file directly we are prompted for the master key.

![](/images/2019/10/image-1.png)

We know that we need a master key file. After poking around, it seems that we can use the many JPG files we found in the users home directory. We will use ```keepass2john``` to convert the JPG file to a hash and try and crack it with ```john```. So for each image we issue ```keepass2john -k user/IMGFILE user/MyPasswords.kdbx > IMG_XXXX_Hash```. This will take the image file specified, convert it to a hash and save it. We will do this will all 6 of the image files.

Once we've converted all of our images files over to hashes, we will brute force the hashes with ```john```. We issue ```john IMGFILEHERE --wordlist=/usr/share/wordlists/rockyou.txt``` for each hash file we've created. Eventually we get a list of passwords to try against the ```KeePass``` database.

![](/images/2019/10/image-2.png)

Now that we have our passwords, we can user ```kpcli``` to interact with the database. You could also just use the GUI interface as well. We launch ```kpcli```. We then use ```open``` to open the ```MyPassword.kdbx``` and also specify the master keyfile.

![](/images/2019/10/image-3.png)

Now that we have entered the database, we see that there is a directory called MyPasswords. We navigate into it and see there is an entry called ```Root password```. We can view it by issuing ```show -f 0```. This will show the contents of entry ```0```. The ```-f``` will unmask the ```Pass``` field.

![](/images/2019/10/image-4.png" caption="show 0 vs show -f 0)

We now have a root password! Lets ```SSH``` back into the box as root. We are unable to SSH into the box as root. So we will ```SSH``` back in as user and ```su root``` to see if we can escalate.

![](/images/2019/10/image-5.png)

We can! We will ```cat /root/root.txt``` and get our flag!

![](/images/2019/10/image-7.png)

Box complete. The use of the image files as master keys was pretty CTF like. This box is a good resource for learning Buffer Overflows.

Hopefully something was learned. If you found this write-up helpful, consider sending some respect my way: [Lovecore's HTB Profile](https://www.hackthebox.eu/home/users/profile/95635).

