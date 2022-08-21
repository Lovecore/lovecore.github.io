+++
author = "Nick"
categories = ["CTF", "windows", "enumeration", "smb"]
date = 2019-09-07T15:28:20Z
description = ""
draft = false
image = "__GHOST_URL__/content/images/2019/07/infocard-2.png"
slug = "hack-the-box-bastion"
summary = "We are back with another Hack the Box walk through, this time we are doing the box Bastion. Lets dive in!"
tags = ["CTF", "windows", "enumeration", "smb"]
title = "Hack The Box - Bastion"

+++


We are back with another Hack the Box walk through, this time we are doing the box Bastion. Lets dive in!

As always, we kick it off with our standard nmap scan:

```
nmap -sC -sV -oA ./bastionscan 10.10.10.134
```

We get back the standard things:

```
Nmap scan report for 10.10.10.134
Host is up (0.055s latency).
Not shown: 996 closed ports
PORT    STATE SERVICE      VERSION
22/tcp  open  ssh          OpenSSH for_Windows_7.9 (protocol 2.0)
| ssh-hostkey: 
|   2048 3a:56:ae:75:3c:78:0e:c8:56:4d:cb:1c:22:bf:45:8a (RSA)
|   256 cc:2e:56:ab:19:97:d5:bb:03:fb:82:cd:63:da:68:01 (ECDSA)
|_  256 93:5f:5d:aa:ca:9f:53:e7:f2:82:e6:64:a8:a3:a0:18 (ED25519)
135/tcp open  msrpc        Microsoft Windows RPC
139/tcp open  netbios-ssn  Microsoft Windows netbios-ssn
445/tcp open  microsoft-ds Windows Server 2016 Standard 14393 microsoft-ds
Service Info: OSs: Windows, Windows Server 2008 R2 - 2012; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: mean: -39m34s, deviation: 1h09m15s, median: 24s
| smb-os-discovery: 
|   OS: Windows Server 2016 Standard 14393 (Windows Server 2016 Standard 6.3)
|   Computer name: Bastion
|   NetBIOS computer name: BASTION\x00
|   Workgroup: WORKGROUP\x00
|_  System time: 2019-07-18T20:06:46+02:00
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2019-07-18 14:06:47
|_  start_date: 2019-07-18 10:35:23

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 18.45 seconds

```

We see that SMB is open, so lets try to connect with a null and map some shares.

![](/images/2019/07/image-37.png)

As we see there are a few shares available. Lets try to connect to the Backup share since it's not hidden.

![](/images/2019/07/image-38.png)

Lets switch to the WindowsImageBackup directory and see what we have

![](/images/2019/07/image-39.png)

We see a directory called L4mpje-PC and when we path to it, we see a backup directory. In here there are a few things of note.

![](/images/2019/07/image-40.png)

We obviously are interesting in the .vhd, or virtual hard drive's. One is 5+GB and the other is only 37 MB. Lets get that one. We can open it up and take a peek, we see its the System Reserve portion of the windows drive. In this case, I want to snag the stored hashes on it, if there are any. So to do this we need to mount the .VHD, we'll use a tool called guest mount. Now there are other ways of doing this, however, if I can mount the file directly from the network resource, thats good. First we'll mount the remote path our local host.

```
mkdir /mnt/remote; mount -t cifs //10.10.10.134/Backups -o user=guest,password= /mnt/remote
```
This command will create a remote directory in mnt and then mount the remote share to it. After that we'll mount it with [guestmount](http://libguestfs.org/guestmount.1.html):
```
mkdir vhd; guestmount --add /mnt/remote/WindowsImageBackup/L4mpje-PC/Backup\ 2019-02-22\ 124351/9b9cfbc4-369e-11e9-a17c-806e6f6e6963.vhd --inspector --ro /mnt/vhd
```

Now we can change to the vhd directory and see what we have.

![](/images/2019/07/image-41.png)

Since we're after a NTLM dump, we'll path to Windows/System32/config and run [samdump2](https://linux.die.net/man/1/samdump2).

![](/images/2019/07/image-42.png" caption="Sweet, some hashes)

We pipe those into a text file for cracking.

```
samdump2 SYSTEM SAM > ~/Documents/bastion/hash.txt
```

Now we see that the Administrator account and the Guest account are disabled. Leaving us with just L4mpje. we'll take the second half of the string and put it into a new document and run hashcat against it. In this case I didn't have enough resources to run it on my VM. So in this case we used [hashkiller](https://hashkiller.co.uk/Cracker/NTLM) or [crackstation](https://crackstation.net/) to do some of the dirty work for us.

![](/images/2019/07/image-43.png" caption="We have a match!)

Now maybe we can use that to SSH in.

![](/images/2019/07/image-44.png)

Sure enough, it works!

![](/images/2019/07/image-45.png)

We path to the Desktop and get the user.txt flag.

![](/images/2019/07/image-47.png)

Now onto the root flag. We need to further enumerate the box, we can do that by heading to the Program Files and Program Files (x86) directories.

![](/images/2019/07/image-48.png" caption="mRemoteNG is suspect...)

Between mRemoteNG and OpenSSH there isn't much here. Some googling around shows the mRemoteNG had an issue where passwords were being saved in the clear, but encrypted. So lets see what config files we can find, normally they're stored in AppData.

![](/images/2019/07/image-49.png)

We see that we have some files here. Lets take a peek at them, starting from the top... Lucky for us, in the first file we see the Administrator password stored in an encrypted format, resembling base64.

![](/images/2019/07/image-50.png)

Now, it's not base 64 but again, some digging around led me to this: [https://github.com/kmahyyg/mremoteng-decrypt](https://github.com/kmahyyg/mremoteng-decrypt). Someone has done the dirty work already! We'll just wget the python script and pass the hash through.

![](/images/2019/07/image-51.png" caption="Damn, lets try the jar file.)

Luckily the jar file worked.

![](/images/2019/07/image-52.png)

Now we have a password for the Administrator. I wonder if this password matches the local administrator password?

![](/images/2019/07/image-53.png" caption="It sure did!)

Now we have admin access lets snag our flag!

![](/images/2019/07/image-54.png)

This was another fairly quick box. The hardest part was the stability of the VPN to the box. It took me a few nmap scans to actually get back results.

Hopefully something was learned. If you found this write-up helpful, consider sending some respect my way: [Lovecore's HTB Profile](https://www.hackthebox.eu/home/users/profile/95635).

