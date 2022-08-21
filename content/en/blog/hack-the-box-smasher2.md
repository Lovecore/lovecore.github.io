+++
author = "Nick"
categories = ["enumeration", "nmap", "dig", "gobuster", "dns", "c", "custom exploit", "reverse shell", "log enumeration", "hack the box", "burpsuite", "JSON"]
date = 2019-12-14T15:25:31Z
description = ""
draft = false
image = "__GHOST_URL__/content/images/2019/10/info-2.png"
slug = "hack-the-box-smasher2"
summary = "Welcome back to anther Hack the Box write up. In this post we're going to go through the box Smasher2."
tags = ["enumeration", "nmap", "dig", "gobuster", "dns", "c", "custom exploit", "reverse shell", "log enumeration", "hack the box", "burpsuite", "JSON"]
title = "Hack the Box - Smasher2"

+++


Welcome back to anther Hack the Box write up. In this post we're going to go through the box Smasher2. I did not have a chance to do the original box, I might go back and do that. Off we go!

Like we do with every box, our standard ```nmap``` scan: ```nmap -sC -sV -T4 -oA smasher2 10.10.10.135```. We see a small set of results.

```
Nmap scan report for 10.10.10.135
Host is up (0.052s latency).
Not shown: 997 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 23:a3:55:a8:c6:cc:74:cc:4d:c7:2c:f8:fc:20:4e:5a (RSA)
|   256 16:21:ba:ce:8c:85:62:04:2e:8c:79:fa:0e:ea:9d:33 (ECDSA)
|_  256 00:97:93:b8:59:b5:0f:79:52:e1:8a:f1:4f:ba:ac:b4 (ED25519)
53/tcp open  domain  ISC BIND 9.11.3-1ubuntu1.3 (Ubuntu Linux)
| dns-nsid: 
|_  bind.version: 9.11.3-1ubuntu1.3-Ubuntu
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: 403 Forbidden
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 15.82 seconds
``` 
We'll repeat the scan but against all ports. The same set of results came back.

We see that port ```53``` is open as well as ```80```. We'll enumerate the web interface with ```gobuster``` and see what it might show. 
```
gobuster dir -u http://10.10.10.135 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
```


While that runs we can run ```nslookup``` against the machine to see what it might return. We see that ```smasher2.htb``` is indeed the hostname so we'll add that to our hosts file. We'll also want to ```dig``` against the host and see what data comes back.

![](/images/2019/10/image-48.png)

A few more hostnames come back, we'll add those as well. We see that the root vhost has an ```apache2``` landing page. We'll enumerate this host as well. We see the same thing for both sites: ```backup```, ```server-status```, ```p99-readfile``` and ```readdir```. The first thing we'll try to do is visit the ```backup``` location. We are greeted with a password authentication.

Once we've bruteforced our way into the directory. We get a list of the directories content.

![](/images/2019/10/image-49.png)

Inside the directory we have two files. A ```python``` script and a ```.so``` file. We download both and start looking at the script. It seems to be a script for internal web service on port 5000. We also see this portion of code:

![](/images/2019/10/image-50.png)

This can let us craft a POST payload to the API endpoint to gain some information. We can use the format ```{"schedule":"Command"}``` to do this. We also want to look at the ```.so``` file. ```.so``` extension often refers to a ```shared-object```. Essentially it's an executable we can decompile and look at the code. Now that we have an understanding of these two files, lets explore the ```wonderfullsessionmanager``` address.

We do some manual enumeration of the site while we are on it. We try to login using admin / admin and get a failure box which happens to return ```JSON```. This is likley the ```auth.py``` webservice.

![](/images/2019/10/image-51.png)

![](/images/2019/10/image-53.png)

I tried to brute force this web login and forgot about the 2 minute lockout. So just changed our ```hydra``` values to slow that down a bit and got in with ```Administrator```.

![](/images/2019/10/image-54.png)

Now that we have a sucessful key to the ```API``` we can try and craft a ```POST``` payload. At first I was unable to use our payload as descibed as above to function. I changed the HTTP request type to ```OPTION``` and sent it after that changed it back to ```POST``` and everything was functioning fine. I'm still unsure on how this was functioning.

Out crafted payload looks like this:

![](/images/2019/10/image-55.png)

This request didn't work. This could be that there is a WAF or other means of filtering on the requests. A fairly standard evasion technique for this is to punctuate the command with ```' '```. So our new command looks like this:

![](/images/2019/10/image-56.png)

And our response is:

![](/images/2019/10/image-57.png)

It works! Now we need to get ourselves a shell using this method. There are a few options. We can copy our ```SSH``` key to the ```authorized_keys``` file. We could generate a payload with ```MSFPC``` and host it via ```SimpleHTTPServer```. We seem to be unable to get a ```netcat``` command to run via this as well. So we'll use the second method and the quick and dirty ```Python``` ```reverse shell``` from [Pentest Monkey](http://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet). We host it via ```SimpleHTTPServer``` and tell our payload to ```wget``` the file.

![](/images/2019/10/image-58.png)

We can also verify this worked because our webserver recorded the download.

![](/images/2019/10/image-59.png)

Now we just need to execute the payload via the same method.

![](/images/2019/10/image-60.png)

I kept getting this same response from the server. So we'll craft another reverse shell, this time in ```perl```.

![](/images/2019/10/image-61.png)

This time it worked. Now we have a shell. We should see what kind of permissions we have so that we can add our key to the ```authorized_keys``` file for future use. There is no ```.ssh``` file so we will create one as well as the ```authorized_keys``` file.

Now we can start enumerating the system. We start with ```LinEnum``` and then ```psps64``` as usual. The ```LinEnum``` results doesn't give us any insight, nor does ```pspy64```. Looks like we're going to do it by hand. So we'll check out a few locations first. We want to view the log files, see what might be in our web directories, check our bin and sbin locations. Those are often good starting places to see if there's anything there we can leverage or that can lead us somewhere else. As we are sifting through the logs there seems to be an odd entry. a kernal driver is loaded.

![](/images/2019/10/image-62.png)

Now that we know where the driver is and that it is a 3rd party driver. Lets see what we can get from it. First we'll run ```strings``` on it to see if that gives us anything useful.

![](/images/2019/10/image-63.png)

Well it looks like we are on the right path. We see there is the ```mmap``` function which is used for memory mapping. After doing some searching I came across [this white paper](https://labs.f-secure.com/assets/BlogFiles/mwri-mmap-exploitation-whitepaper-2017-09-18.pdf). There is a very large section on leveraging ```mmap``` starting on page 13 of the white paper. After reading it and then re-reading it. I had a better understanding of what was going on. We can then take the code they've provided and start using and modifying it.

![](/images/2019/10/image-64.png)

We do indeed see that it is vulnerable. Now we can craft (copy paste) the exploit according to the white paper.

![](/images/2019/10/image-65.png)

This was pretty wild. If there wasn't a white paper on this with the code to copy, I would have considered this a much harder box than Kryptos!

Hopefully something was learned. If you found this write-up helpful, consider sending some respect my way: [Lovecore's HTB Profile](https://www.hackthebox.eu/home/users/profile/95635).

