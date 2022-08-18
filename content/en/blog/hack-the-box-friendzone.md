+++
author = "Nick"
categories = ["CTF", "NetSec", "metasploit", "python", "netcat", "smb", "web services", "reverse shell"]
date = 2019-07-13T15:12:00Z
description = ""
draft = false
image = "__GHOST_URL__/content/images/2019/07/infocard.png"
slug = "hack-the-box-friendzone"
summary = "Now I had already started on this box a while back but had to stop due to time constraints, so now it's time to pickup where we left off and finish it up!"
tags = ["CTF", "NetSec", "metasploit", "python", "netcat", "smb", "web services", "reverse shell"]
title = "Hack The Box - FriendZone"

+++


Welcome back everyone! Today we'll be doing FriendZone. This is a last minute write up since I've been meaning to get to doing a box recently and just noticed that this box will be retired in a few hours! Now I had already started on this box a while back but had to stop due to time constraints, so now it's time to pickup where we left off and finish it up!

We kick it off with our normal nmap scan:

```bash
nmap -sC -sV -oA FriendZone/Scan 10.10.10.123
```

We see some interesting results:

21/tcp  open  ftp         vsftpd 3.0.3
22/tcp  open  ssh         OpenSSH 7.6p1 Ubuntu 4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 a9:68:24:bc:97:1f:1e:54:a5:80:45:e7:4c:d9:aa:a0 (RSA)
|   256 e5:44:01:46:ee:7a:bb:7c:e9:1a:cb:14:99:9e:2b:8e (ECDSA)
|_  256 00:4e:1a:4f:33:e8:a0:de:86:a6:e4:2a:5f:84:61:2b (ED25519)
53/tcp  open  domain      ISC BIND 9.11.3-1ubuntu1.2 (Ubuntu Linux)
| dns-nsid: 
|_  bind.version: 9.11.3-1ubuntu1.2-Ubuntu
80/tcp  open  http        Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Friend Zone Escape software
139/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
443/tcp open  ssl/http    Apache httpd 2.4.29
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: 404 Not Found
| ssl-cert: Subject: commonName=friendzone.red/organizationName=CODERED/stateOrProvinceName=CODERED/countryName=JO
| Not valid before: 2018-10-05T21:02:30
|_Not valid after:  2018-11-04T21:02:30
|_ssl-date: TLS randomness does not represent time
| tls-alpn: 
|   http/1.1

445/tcp open  netbios-ssn Samba smbd 4.7.6-Ubuntu (workgroup: WORKGROUP)
Service Info: Hosts: FRIENDZONE, 127.0.1.1; OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

So we point our browser over to the box and see whats on port 80. Pretty static site, with an email a picture that's about it. Nothing in the source, so we take a look on port 443 and nothing there. What is curious that we see a reference to friendzoneportal.red. So lets add friendzoneportal and friendzoneportal.red to our hosts list. Now lets try to resolve friendzoneportal.red and see what we get.

{{< figure src="__GHOST_URL__/content/images/2019/07/image-3.png" >}}

That takes us to a new page? Lets check the source. Nothing really. Lets change those entries to friendzone.red and see what we get.

{{< figure src="__GHOST_URL__/content/images/2019/07/image-4.png" caption="" >}}

Great another page with a .gif. Lets check this source. We see a code comment with a directory listing. Lets see where that takes us.

{{< figure src="__GHOST_URL__/content/images/2019/07/image-5.png" >}}

Now we have a page thats has what looks like an encoded base64 string on it.

{{< figure src="__GHOST_URL__/content/images/2019/07/image-6.png" >}}

Lets see if it converts to anything, nothing, hmmm. We see the code comment on this page saying "don't stare too much , you will be smashed ! , it's all about times and zones", this leads me to believe its a dynamic string based on time? Lets keep digging.

We see that port 53 is open for DNS which we can possibly use to enumerate later on. We'll also quickly try to use smbmap to see what is available on the host.

{{< figure src="__GHOST_URL__/content/images/2019/07/image.png" >}}

We see that we have general as read only and Development as read / write. Awesome, lets see whats there. We connect with smbclient to general and list the contents.

{{< figure src="__GHOST_URL__/content/images/2019/07/image-1.png" caption="creds.txt?!" >}}

Cool, a file called 'creds.txt'. This might be handy! We download the file and we cat the contents.

{{< figure src="__GHOST_URL__/content/images/2019/07/image-2.png" >}}

Next we'll try to use those credentials to connect to the SMB shares. That doesn't seem to work though, every time we connect, we simply connect as the guest account. Next we connect to the Development share and take a peek around. Unfortunately for us, there's nothing there.

Right about here I was getting a little curious as to where to go next, as I reviewed my nmap scan for something I might have missed, I thought well why not DNS? I didn't enumerate at the beginning thinking it was a potential rabbit hole. Since we seem to have two separate hostnames going on friendzone.red and friendzoneportal.red, we might be able to use dig to get some more info with a zone transfer. More about [zone transfers here](https://digi.ninja/projects/zonetransferme.php).

So we give it a dig and see that we have administrator1.friendzone.red, hr.friendzone.red and uploads.frienzone.red.

{{< figure src="__GHOST_URL__/content/images/2019/07/image-7.png" >}}

Lets check friendzoneportal.red and see whats there as well.

{{< figure src="__GHOST_URL__/content/images/2019/07/image-8.png" >}}

We have admin, files, imports and vpn records. Well then, that's a lot of records. I'll append the names we found to our host file so that we can resolve them. After that, we visit each of the pages to see whats there and whats not.

We see that administrator1 has a login page, lets use the credentials from before to see if they work. Sure enough they do, and we get a message.

{{< figure src="__GHOST_URL__/content/images/2019/07/image-9.png" >}}

So, we do what it says and head over to dashboard.php and see what's in store. We find a page that has to deal with images and fetching images that have been uploaded. We just happen to have a domain called uploads.freindzone.red, we might be able to execute some type of remote .php shell if it allows us to upload.

{{< figure src="__GHOST_URL__/content/images/2019/07/image-11.png" >}}

In this case, I tried to uploads a .txt file and it worked! When we upoad a file to the uploads page, we get back a stamp of some type.

{{< figure src="__GHOST_URL__/content/images/2019/07/image-12.png" >}}

Now back on the admin page, we the image_id and pagename params given to try and load the image, or in this case, the .txt file. It seems to work, we get back a broken image and some error info.

{{< figure src="__GHOST_URL__/content/images/2019/07/image-13.png" >}}

So know that we that we can upload something we can probably also execute something as well. We can try to get the site to read a file we've put somewhere on the server, lucky for us, we have the ability to read and write on the Development share. So lets upload a reverse php shell there and maybe we can get this page to load it for us!

We'll use the fairly standard php-reverseshell.php thats comes in the Kali distro to see if that works. If it doesn't we can create one with MSVenom and get a Meterpreter session.

We connect back to the SMB share and upload our shell.php and start our Netcat listener on our port.

{{< figure src="__GHOST_URL__/content/images/2019/07/image-14.png" >}}

We head over to our admin page and load the path from /etc/Development/shell and see if it works.

{{< figure src="__GHOST_URL__/content/images/2019/07/image-15.png" >}}

And it does! So what should be noted is that the share for Development is listed in the same location as the general share. You can see that if you enumerate the smb shares properly. Now lets grabs some creds!

We are running the shell as the www daemon, so we'll need some way to pivot. In this case, we're going to see what we have in the www directories first. Since this is a CTF and not a live red team exercise ;). We see in www there is a mysql_data.conf. When we view it we get a username and password.

{{< figure src="__GHOST_URL__/content/images/2019/07/image-16.png" >}}

So lets try to switch user and see what we get. Su must be run from a terminal. Looks like we'll need to spawn a TTY session. We can do this by leveraging Python that is built into the OS. For more cool shell tricks, [see here](https://netsec.ws/?p=337).

Once we've spawned a python shell, we can su to the Friend account and get creds. We' snag the user.txt file and head onto root! While we were browsing around the files for the www daemon we saw a directory called server_admin, in there is a reporter.py. Lets peek the code:

{{< figure src="__GHOST_URL__/content/images/2019/07/image-17.png" caption="Hmm, possibly exploitable." >}}

We might be able to do some root level cron job type things with this. We'll load up LinEnum to enumerate the box and see what we can do. Oddly enough after it runs, we don't see this python script being called in the cron jobs.

{{< figure src="__GHOST_URL__/content/images/2019/07/image-18.png" >}}

But we do see os.py as a world writable and we know that our script calls it as well. This is our path forward! We append a small backdoor to the core os.py code and wait. Now, in actuallity, you could have just written a python command to copy the root.txt file to tmp or something but I enjoy backdoor trickery!  Sure enough a few minutes later our Netcat lights up and we have a shell back as root!

{{< figure src="__GHOST_URL__/content/images/2019/07/image-19.png" >}}

Now all we do is cat the root.txt and we get the hash!

{{< figure src="__GHOST_URL__/content/images/2019/07/image-20.png" >}}

Hopefully something was learned. If you found this write-up helpful, consider sending some respect my way: [Lovecore's HTB Profile](https://www.hackthebox.eu/home/users/profile/95635).

