+++
author = "Nick"
categories = ["CTF", "NetSec"]
date = 2019-04-20T16:41:00Z
description = ""
draft = false
image = "__GHOST_URL__/content/images/2019/04/netmon_card.png"
slug = "hack-the-box-netmon"
summary = "I was pretty fortunate that this box is VERY easy to crack, which was great because I have been pretty time limited, especially on a holiday weekend. So, Lets dive into the box called 'Netmon'!"
tags = ["CTF", "NetSec"]
title = "Hack The Box - Netmon"

+++


It's been a while since I've been able to do a HTB. So I figured why not try to get one in really quick over this Easter weekend?! I was pretty fortunate that this box is VERY easy to crack, which was great because I have been pretty time limited, especially on a holiday weekend. So, Lets dive into the box called 'Netmon'!

We run our normal nmap scan

```bash
nmap -sC -sV -oA 10.10.10.152
```

We get back some pretty juicy stuff, anon FTP, SMB and HTTP. This could be smooooth sailing!

```bash
Starting Nmap 7.70 ( https://nmap.org ) at 2019-04-20 11:40 EDT
Nmap scan report for 10.10.10.152
Host is up (0.14s latency).
Not shown: 995 closed ports
PORT    STATE SERVICE      VERSION
21/tcp  open  ftp          Microsoft ftpd
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| 02-03-19  12:18AM                 1024 .rnd
| 02-25-19  10:15PM       <DIR>          inetpub
| 07-16-16  09:18AM       <DIR>          PerfLogs
| 02-25-19  10:56PM       <DIR>          Program Files
| 02-03-19  12:28AM       <DIR>          Program Files (x86)
| 02-03-19  08:08AM       <DIR>          Users
|_02-25-19  11:49PM       <DIR>          Windows
| ftp-syst: 
|_  SYST: Windows_NT
80/tcp  open  http         Indy httpd 18.1.37.13946 (Paessler PRTG bandwidth monitor)
|_http-server-header: PRTG/18.1.37.13946
| http-title: Welcome | PRTG Network Monitor (NETMON)
|_Requested resource was /index.htm
|_http-trane-info: Problem with XML parsing of /evox/about
135/tcp open  msrpc        Microsoft Windows RPC
139/tcp open  netbios-ssn  Microsoft Windows netbios-ssn
445/tcp open  microsoft-ds Microsoft Windows Server 2008 R2 - 2012 microsoft-ds
Service Info: OSs: Windows, Windows Server 2008 R2 - 2012; CPE: cpe:/o:microsoft:windows

Host script results:
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2019-04-20 11:41:08
|_  start_date: 2019-04-20 11:32:40

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 33.02 seconds
```

Lets try the FTP port since we can log in anonymously. We log into the box and take a peek around.

{{< figure src="__GHOST_URL__/content/images/2019/04/ftp_anon1-1.png" >}}

We log in and head over the the Users directory and see what we have at first glance. Administrator and Public, ok cool. Lets look at Public desktop, maybe we have a User hash.

{{< figure src="__GHOST_URL__/content/images/2019/04/ftp_anon2.png" caption="Sure enough our user hash is hanging out in the Public space!" >}}

Well, that was... unexpectedly easy. Lets get our Admin hash now! Try the normal things, like accessing the Admin account directory, denied. Check other core access privileges, also denied. Lets drop the FTP session and see what we can see on port 80.

We point our browser to the ip and we get the PRTG Network Monitor login page.

{{< figure src="__GHOST_URL__/content/images/2019/04/netmon-1.png" caption="" >}}

Looks like we're running version 18.1.37. Lets try to log in with the default admin credentials of prtgadmin / prtgadmin. Nope, denied. Well, lets google around for some known exploits for PRTG.

We seem to find a few, one in particular that seems great, CVE-[2018-9276](https://www.exploit-db.com/exploits/46527). The problem with this particular CVE is it requires you to be authenticated already. Damn. A bit more googling comes up with some particularly interesting System Administrator posts talking about how this particular software backups where storing credentials in plain text! Now it seems that the effected version aren't our versions, but maybe they did a few in-place upgrades since the issue was disclosed. Lets see what we can find.

We jump back onto the FTP and try to path to ProgramData/Paessler/Configuration Auto-Backups/. You guessed it, this is where PRTG stores its automatic backups. If we're lucky we might have an old backup to parse through with some plaintext goodies in it.

{{< figure src="__GHOST_URL__/content/images/2019/04/ftp_ls_programData.png" >}}

We see there are 3 backup files listed. the first two are the same size, so we'll grab a copy of one. The .old.bak file is a bit smaller so we'll grab that as well. Once we've downloaded them we will just grep through them for the word 'password'.

We'll grep the .old file first. We'll use the -A for trailing lines to see what will come after the word 'password'
```bash
grep -A1 password 'PRTG Configuration.old'
```
Turns out everything in the .old file is followed by <encrypted>. Lets try the same set of commands on the old.bak.
![db_password-1](__GHOST_URL__/content/images/2019/04/db_password-1.png)
    
This time we found some credentials! Looks like the prtgadmin account password from the oldest backup we have.

So time to head over to the web interface and try the credentials. Failed. Damn it. Well, these are old credentials, maybe the user was required to change the password. Lets try Prtg@dmin2019. BINGO!

{{< figure src="__GHOST_URL__/content/images/2019/04/netmon-2.png" >}}

This was actually the hardest part. I was thinking that I needed to brute force the box but in actuality, I just needed to think as a user. What would a user do? Change the year...

Now that we have admin access we can go back to CVE-2018-976. The exploit requires our cookies from cache. So we load up developer options and snag the data.

{{< figure src="__GHOST_URL__/content/images/2019/04/cookie1.png" >}}

{{< figure src="__GHOST_URL__/content/images/2019/04/cookie2-1.png" >}}

{{< figure src="__GHOST_URL__/content/images/2019/04/cookie3.png" >}}

Now we just need to run the exploit with the appropiate switches.
```bash
bash prtg-exploit.sh -u http://10.10.10.10 -c "_ga=GA1.4.1727477117.1555948138; _gid=GA1.4.1609818662.1555948138; OCTOPUS1813713946=ezM4QUUyMDVCLUNBNkEtNEEwQS1BNjc4LUEzMzVEMTM1Qjk1MX0%3D; _gat=1"
```

{{< figure src="__GHOST_URL__/content/images/2019/04/cve_run-1.png" caption="We now have an admin account on the server!" >}}

Now that we have an admin account, we can use SMBCLIENT to connect to the C drive and get the admin hash.

{{< figure src="__GHOST_URL__/content/images/2019/04/smb3_C_pentest.png" >}}

{{< figure src="__GHOST_URL__/content/images/2019/04/smb4_C_pentest.png" caption="There the root flag!" >}}

On the Administrator desktop is the root.txt, we win!

All in all this took about an hour to finish. I would really recommend this box to anyone that is just getting into the field of pentesting.

Hopefully something was learned. If you found this write-up helpful, consider sending some respect my way: [Lovecore's HTB Profile](https://www.hackthebox.eu/home/users/profile/95635).

