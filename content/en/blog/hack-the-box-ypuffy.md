+++
author = "Nick"
categories = ["CTF", "NetSec"]
date = 2019-02-14T17:53:46Z
description = ""
draft = false
thumbnail = "/images/2019/02/infocard.PNG"
slug = "hack-the-box-ypuffy"
summary = "We are back with another HtB walk through. I just finished this box maybe 12 hours before it went into retired, so that was awesome! This box is a pretty cool box with a fairly common entry and a common escalation path"
tags = ["CTF", "NetSec"]
title = "Hack the Box - Ypuffy"

+++


We are back with another HtB walk through. I just finished this box maybe 12 hours before it went into retired, so that was awesome! This box is a pretty cool box with a fairly common entry and a common escalation path. I've actually seen this sort of entry point in the field before, which is (very) scary. It seems to be much more common in small to medium businesses and even some schools. Lets dive in!

Our standard nmap scan shows some fairly common things and one thing that is an instantly stands out to me. Anonymous bind on the ldap port.

```bash
nmap scan report for 10.10.10.107
Host is up (0.13s latency).
Not shown: 995 closed ports
PORT    STATE SERVICE     VERSION
22/tcp  open  ssh         OpenSSH 7.7 (protocol 2.0)
| ssh-hostkey: 
|   2048 2e:19:e6:af:1b:a7:b0:e8:07:2a:2b:11:5d:7b:c6:04 (RSA)
|   256 dd:0f:6a:2a:53:ee:19:50:d9:e5:e7:81:04:8d:91:b6 (ECDSA)
|_  256 21:9e:db:bd:e1:78:4d:72:b0:ea:b4:97:fb:7f:af:91 (ED25519)
80/tcp  open  http        OpenBSD httpd
139/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: YPUFFY)
389/tcp open  ldap        (Anonymous bind OK)
445/tcp open  netbios-ssn Samba smbd 4.7.6 (workgroup: YPUFFY)
Service Info: Host: YPUFFY

Host script results:
|_clock-skew: mean: 1h40m00s, deviation: 2h53m12s, median: 0s
| smb-os-discovery: 
|   OS: Windows 6.1 (Samba 4.7.6)
|   Computer name: ypuffy
|   NetBIOS computer name: YPUFFY\x00
|   Domain name: hackthebox.htb
|   FQDN: ypuffy.hackthebox.htb
|_  System time: 2019-02-14T09:59:24-05:00
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2019-02-06 09:59:23
|_  start_date: N/A

I poked around on port 80 for a bit to see what was there. Nothing really, I checked it out via web as well as in BurpSuite and still didn't see any data. Sending it bad data just gave us error 400's back. It seems like a black hole to me, so lets skip that and circle back around to it.

Anonymous bind on the LDAP should yield some kind of information. We can actually use nmap to get some information about the baseDN or we can use a tool called [ldapsearch](https://linux.die.net/man/1/ldapsearch) to get some info from the box. I've spent years as a Windows System Engineer so I'm a bit more comfortable with nmap here. Also, if you're reading this and didn't know that nmap has this functionality, take a look here to learn more about the power of nmap!

![](/images/2019/02/nmap-ldap.png" caption="We issue the script name ad well as -Pn to disable host discovery because we've already done that.)

We get back a ton of info on our domain, the key part is right here though:

![](/images/2019/02/baseDN-enum.png" caption="A NT Password hash?! Lets pass it around!)

We see a NT hash listed as well as a username. We want to leverage these but how? Well we know from our nmap scan that SMB service was running, we should try to leverage that service since its known to like valid NTLM hashes for authentication. We can use a tool called SMBmap to do just that.

![](/images/2019/02/smbmap1.png" caption="We give the NT hash in and point it to the box and poof!)

We now see the alice share that we have read and write to. We also see the admin share which has no access, usually meaning that the user doesn't have any administrative rights. So lets tell smbmap to list recursively. To do that we simply add -R on the end of our command.

![](/images/2019/02/smbmap2.png" caption="We see a key file! Lets snag it!)

We see a private key! There's a really good chance we can use that to SSH with. Lets get it. We can use --downloads flag to get the file we want.

![](/images/2019/02/smbmap3.png" caption="Yoink!)

Now that we have a key file, lets get connected via SSH. First attempt to connect gave me the old 'change your permissions on the key file' response. Ok, fine, 400 it is, lets try again. Nope, invalid format?!

![](/images/2019/02/ssh1-1.png" caption="What the?!)

Well, that's weird, let try to open the file and see what's going on, maybe it needs to be converted or something.

When we cat the file, the first line gives us the answer... PuTTY file type.

![](/images/2019/02/putty.png" caption="Looks like we're install putty tools today....)

Ok, lets apt install putty-tools to get the things we need. We could alternately just jump onto a windows box and use it there. In an attempt to try and keep things in one machine, we'll just convert the key over.

Once those are installed we can use puttygen to convert the key over.

![](/images/2019/02/image.png" caption="We issue the -O for our new key type and the -o for the output filename)

Now that we have a new key, we can change its permissions and attempt to ssh as alice.

![](/images/2019/02/image-1.png" caption="Looks like it worked, lets get our user flag!)

A quick ls and we see our user.txt file! While we're at it, lets uname -a on the system too.

![](/images/2019/02/image-3.png)

![](/images/2019/02/image-19.png" caption="Shit.. OpenBSD, I hate this OS.)

Now that we're in lets see what users we have here. A quick 'cd ..; ls' shows we have 3 users. We can also get entries on the passwd file. OpenBSD is a real pain in my ass... Normally we can just cat or less a file and pipe it into awk or sed and grep results from there...

![](/images/2019/02/image-4.png" caption="cd..;ls results)

![](/images/2019/02/image-5.png" caption="We pipe the get entry command to grep and we give the -v for invert match and tell it to filter out any names with underscores.)

Now that we have an idea of what's system we have, lets look up what type of CVE's we might have for OpenBSD 6.3. Turns out, there is [one](https://techblog.mediaservice.net/2018/10/cve-2018-14665-exploit-local-privilege-escalation-on-openbsd-6-3-and-6-4/)! A quick copy past into the shell and we are off!

![](/images/2019/02/image-20.png" caption="Root!)

After our 2 minute wait, we see our prompt has changed from a standard user ($) to a root user (#). Easy mode! Turns out this exploit was released well after the box, so that's no fun. Lets find the actual exploit path required.

I poked around for a while. I was able to read a sshauth.sql file in Bob's directory but that's really all that was there. The userca had a file or two listed there as well, key files. Lets take a look at processes using ps -aux and see whats there. There's quite a few entries listed here for sshauthd. Lets see whats there.

![](/images/2019/02/image-6.png)

Permission denied to /var/appsrv. Damn. Nothing of use seemingly in /var/log either. At this point I bashed my head around all sorts of items in the box for a few days. At this point, it was time to get back to basics, maybe a service was misconfigured. Let start by looking at the sshd_config file to see what might be there. We see two odd entries here:

![](/images/2019/02/image-7.png" caption="Authorized curl commands?)

Lets try and run them and see what we  get back. When we substitute our username in the %u of the curl we get back a key!

![](/images/2019/02/image-8.png" caption="Well then)

Well, we can take then and create a key maybe... I wonder if that can work for other users. Lets try the other command listed, AuthorizePricipalsCommand. When we issue that command with alice1978, we simply get back, alice1978. Maybe we can feed users and see what we get in return.

![](/images/2019/02/image-10.png" caption="Well, it does seem to return some values, even one that looks fun!)

Given the command structure, I would guess that it is returning a sshauth key or password of some sort back for the entered user.

OpenBSD uses a command called doas for Super User functions rather than sudo. So to check that we can view our doas permisions by getting the content of doas.conf.

![](/images/2019/02/image-11.png" caption="Looks like alice has the ability to run ssh-keygen.)

I would say that hunch lined up perfectly. So we have the ability to run ssh-keygen and we have what could be a key for the root user.  Lets create a keypair.

![](/images/2019/02/image-12.png)

Now that we have a key, lets try to ssh as root. Denied.

![](/images/2019/02/image-13.png)

Well, we do seem to have the ability to run ssh-keygen as certca user, lets create an entire new keypair as that user. After a few attempts at creating the key, because it needed a key id (-I) and then userca need read write to my directory, I was finally able to create a keypair!

![](/images/2019/02/image-14.png" caption="We now have a signed cert!)

Lets ssh into the localhost as root and see what we get.

![](/images/2019/02/image-18.png" caption="Easy money!)

We are in! I spent more time looking up openBSD commands than anything else in this box

Hopefully something was learned. If you found this write-up helpful, consider sending some respect my way: [Lovecore's HTB Profile](https://www.hackthebox.eu/home/users/profile/95635).

