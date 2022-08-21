+++
author = "Nick"
categories = ["hack the box", "openBSD", "medium", "cve-2019-19520", "cve-2019-19522"]
date = 2020-12-12T15:00:00Z
description = ""
draft = false
thumbnail = "/images/2020/11/info.png"
slug = "hack-the-box-openkeys"
tags = ["hack the box", "openBSD", "medium", "cve-2019-19520", "cve-2019-19522"]
title = "Hack the Box - OpenKeyS"

+++


Welcome back! Today we are doing the Hack the Box machince - OpenKeyS. This is an OpenBSD machine with a difficulty of Medium. Let's dive in!

As always, `nmap` it: `nmap -sC -sV -p- -oA allscan 10.10.10.199`

Here are our results:
```
Nmap scan report for 10.10.10.199
Host is up (0.047s latency).
Not shown: 65533 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.1 (protocol 2.0)
| ssh-hostkey: 
|   3072 5e:ff:81:e9:1f:9b:f8:9a:25:df:5d:82:1a:dd:7a:81 (RSA)
|   256 64:7a:5a:52:85:c5:6d:d5:4a:6b:a7:1a:9a:8a:b9:bb (ECDSA)
|_  256 12:35:4b:6e:23:09:dc:ea:00:8c:72:20:c7:50:32:f3 (ED25519)
80/tcp open  http    OpenBSD httpd
|_http-title: Site doesn't have a title (text/html).

```

That's a pretty limited set of results! Looks like we'll head over to see what's being hosted on port 80. 

We are greated with a login page. Before we attempt to login, we'll do some basic enumeration with `gobuster` to see what we can find.

Command:
`gobuster dir -u 10.10.10.199 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php -t 50`

We pretty much see our standard items:

![](/images/2020/11/image-20.png)

When we browse to the `includes` location, we see two files. `auth.php` and `auth.php.swp`. When we try and load the `.swp` file we get some info. A user of Jennifer and a hostname of openkeys.htb. We'll add the hostname to our hosts file and try some enumeration for subdomains.

Command:
`ffuf -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-110000.txt -u http://FUZZ.openkeys.htb -fc 404 -c`

There's nothing to see in that enumeration. So we'll download the `auth.php` file and the 'auth.php.swp` file as well.

Command:
`wget http://10.10.10.199/includes/auth.php`
`wget http://10.10.10.199/includes/auth.php.swp`

When we try to view the contents of the `auth.php` file, there's nothing inside. When we view the contents of the `.swp` it'. We can use `strings` to help visualize it a bit better.

Command:
`strings auth.php.swp`

Nothing new in the code at all. Time to go back a level to `/auth_helpers/` as the code suggests and look at the files there. Inside we have a binary file called `check_auth`. Now when we try to run it, no dice. 

We need to determine a way forward, which seemingly is the web interface. So when we stop and think about what the site does and how it verifies. The know the site is trying to check if the user is logged in. Now is this checking against the system, yes, but there could also be some functionlity checking against the web client as well. So we'll try to do just that, make the site think we are Jennifer so we can get a proper session. 

We create a new cookie called username and give it a value of jennifer. Try to login with some junk credentials, no dice.

![](/images/2020/11/image-22.png)

Doing some research on OpenBSD and web authentications leads me to this article: [schallenge](https://www.secpod.com/blog/openbsd-authentication-bypass-and-local-privilege-escalation-vulnerabilities/). Now given this information, we could potentially use a username of `-schallenge` and a junk password in conjunction with our cookie to log in.

![](/images/2020/11/openkey.gif)

Sure enough, it works! We now have an `SSH` key for Jennifer! We save the file, give it the correct permissions and try to log in.

![](/images/2020/11/image-23.png)

Now we can snag our `user.txt` flag! Once we have that we start out internal enumeration. We download `linpeas` to the machine and let it run and see what it comes back with. Nothing great. However, we know from our previous research that CVE-2019-19520 has local auth bypass. Some googling arounds leads us to a [PoC](https://github.com/bcoles/local-exploits/blob/master/CVE-2019-19520/openbsd-authroot).

All we have to do is potentially run the script!

![](/images/2020/11/root.gif)

There we have it, our root flag!



