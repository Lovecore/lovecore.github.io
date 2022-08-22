+++
author = "Nick"
categories = ["CTF", "NetSec"]
date = 2019-04-04T15:31:27Z
description = ""
draft = false
thumbnail = "/images/2019/01/curling.PNG"
slug = "hack-the-box-curling"
summary = "Hack the Box Machine - Curling"
tags = ["CTF", "NetSec"]
title = "Hack the Box - Curling"
url = "/hack-the-box-curling"
 

+++


Welcome back friends! This post will be about the HtB machine Curling. This machine felt more CTF like than others. Another fairly simple box to gain the flags on but that's what we expect from easier boxes. I should note that I've been posting about the easier boxes since they're a better use of my time. I wish I had more time to dedicate to cracking some of the absurdly hard boxes but unfortunately, I don't. It generally takes me 1-3 hours for an easy box & write-up. Lets dig in!

As always, we kick it off with our standard nmap scan:

```bash
nmap -sC -sV -oA 10.10.10.150
```

Lets see what we get back:

```bash
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 8a:d1:69:b4:90:20:3e:a7:b6:54:01:eb:68:30:3a:ca (RSA)
|   256 9f:0b:c2:b2:0b:ad:8f:a1:4e:0b:f6:33:79:ef:fb:43 (ECDSA)
|_  256 c1:2a:35:44:30:0c:5b:56:6a:3f:a5:cc:64:66:d9:a9 (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-generator: Joomla! - Open Source Content Management
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Home
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Looks like we have another web service running as well as SSH. The web service running seems to be Joomla. So lets poke around there and see what we have. Once we load the site up we see that it's a site about the sport Curling. When we browse through the sites posts we see them posted as Super User. What sticks out is that the first post is signed by Floris. So we know a potential username and within that same post there a comment that looks like a potential password as well. So lets try to log in as floris with the hinted password of 'curling2018'.

![](/images/2019/01/floris.PNG" caption="Hinting at a password?)

Well, no combination of curling2018 with the admin credentials seemed to work, lets view the source too see if we can find any type of potential CSRF. We didn't find anything related to CSRF but we did find this cool tidbit. A commented out filename of secret.txt.

![](/images/2019/01/secret.PNG)

So naturally, lets try to path to /secret.txt. It worked, it's given us a string.

![](/images/2019/01/string.PNG" caption="What could this be?)

Now we just need to figure out what this is for, could it be a password? Lets try it, nope not a password. When we look at it, it looks like it might be a possible base64 given that it it has 4 sections of 4 characters its simply missing the = at the end. Worth a shot, lets try. Sure enough, it is! We get the result of Curling2018!. Fairly close to what we had originally guessed. So lets try logging in as Floris using those credentials.

It worked! We are now logged into the site as Super User. After poking around to see what type of permissions we have, they do indeed seem to be actual Super User permissions. A common attack on Joomla is to shell the site so that you can browse the box via a web facing shell. What we'll do in this case is create a reverse Meterpreter session. The reason being is that I'm doing this on the free servers, so for me to give cli access via the web interface would give other CTF players an advantage.

Lets create a php backdoor using MSFPC.

![](/images/2019/01/msfpc-2.PNG" caption="We supply msfpc the php command for the type, an ip to bind to and verbose output.)

Now that we've created our payload, we're going to open the file up and copy the contents. We'll then switch back to our Joomla admin page and we are going to modify a page of the template with this back door. Navigating to the template section of the active template we're going to modify the error.php file.

![](/images/2019/01/theme.PNG" caption="The error.php page ready to be modified!)

In this case, we're just going to delete the entire contents of the page and paste in our backdoor code and save it. Normally you would want to be a bit more stealthier than that but in this case, we don't really care. Lets start up our listener by issuing the command that MSFPC gave us :

```bash
msfconsole -q -r '/root/Documents/curling/php-meterpreter-staged-reverse-tcp-443-php.rc'
```

Once that is done we will navigate to the error page in our browser and we should see our reverse listener light up. Sure enough we do! If you do take this route and you are on a public server cluster, you will inevitably spawn more Meterpreter shells as people land on the error page...

![](/images/2019/01/session.PNG" caption="One session active!)

![](/images/2019/01/heh.png" caption="I reached 189 connections before the box rebooted!)

Now we can start an interactive shell session with sessions -i 1. We're in! We run a few commands to see what we are working with.

![](/images/2019/01/info2.PNG" caption="Looks like a more current Ubuntu build.)

Looks like we're the www-data account. Lets take a look around. When we look at /home/ we only see one user home folder, floris.

![](/images/2019/01/florishome.PNG)

Inside floris' home folder we see very little a folder caled admin-area, password_backup file and user.txt. Lets try to cat user.txt and get the user flag! Operation failed damn. That would have been too easy. Lets try to navigate to admin-area, denied. Well, it does however look like I have read access to password_backup. Lets download that and see what it is.

![](/images/2019/01/download-2.PNG)

Now we get to the fun part. We have a password_backup file. When we look at he file, its a hexdump. Lets rename it and move it into its own directory.

![](/images/2019/01/bzip0.PNG)

It seems to have a header of BZh91AY&SY which means its could be bzip file type. So lets try and recompile this hexdump. The tool hexdump wont do that for us but xxd will! We will issue xxd -r password_backup > recompiled.txt.

![](/images/2019/01/bzip1.PNG)

Now we should have an ascii file out, lets take a look. Looks like it still has a bzip header spacing. If we do a 'file' on it, we should see what type it is and sure enough its a bzip file.

![](/images/2019/01/bzip2.PNG" caption=")

Lets rerun the xxd command but specify it to be a bz2 output.

![](/images/2019/01/bzip3.PNG)

Now, lets try to unzip the file using bzip2, if we want to keep the file, we can issued the -k (not that I did that...).

![](/images/2019/01/bzip4.PNG" caption="We supply bzip2 with -d to decompress (and the -k if we wanted to keep it....))

We now have an unziped file called 'decompiled', lets check the file type of it.

![](/images/2019/01/bzip5-1.PNG)

Looks like a .gzip, lets give it the .gzip extension. now lets unzip it. Before we do that we're going to rename the original decompiled file so we don't get an overwrite error.

![](/images/2019/01/bzip9-2.PNG)

It gave us an output file of decompiled, this is why we needed to rename the original. When we look at the file we end up seeing its back to a .bz2 file. When we extract that file, we get an error as well as a .out file.

![](/images/2019/01/bzip10.PNG)

When we cat the file, we see some data that looks like some user ids and a potential password?

![](/images/2019/01/bzip11.PNG)

Lets check the file type, looks like its a .tar.

![](/images/2019/01/bzip12.PNG)

Lets rename and extract it. Finally, a password.txt file!

![](/images/2019/01/bzip13.PNG)

Now the password listed is indeed the same password we saw above however following through to a final file type was the goal here. Inside we find a password, presumably for floris since it was her backup file we took. So we have a user and a password. This machine did have SSH running on it, so lets try those credentials there.

![](/images/2019/01/ssh.PNG)

Those credentials worked! Lets get our user flag:

```bash
cat user.txt
```

Now that we have the user flag, lets get the root flag. This box was one of the scenarios where I over thought the entire process for WAY longer than I should have. I ran through potential exploits, RCE and wrote some code for custom enumeration. Turns out, I simply overlooked the key...

One of the first things you should do when you have a foothold on a machine is enumerate, enumerate and then enumerate some more. In this scenario, I did my enumeration but over looked one small task, the **running** cronjobs. When we take a look with ps aux, we see a key moment in time:

![](/images/2019/01/cron1.PNG)

A cron job runs that calls a curl command using data from the 'admin-area' directory we saw earlier. When we look we have the -K which tells curl to use an input file and -o to give an output. So what we will try to do is modify this input file to get what we want. When we look at the contents of 'input' we see it's pointing to a URL of the local host.

![](/images/2019/01/input.PNG)

When we check the contents of the report file, we see the Joomla homepage. In this case we will change the local URL to point to /root/root.txt and have it output that content to the report!

![](/images/2019/01/cat2.PNG)

When the curl job runs, we then get the password output to report! We have the root flag!

Hopefully something was learned. If you found this write-up helpful, consider sending some respect my way: [Lovecore's HTB Profile](https://www.hackthebox.eu/home/users/profile/95635).

