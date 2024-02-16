+++
author = "Nick"
categories = ["hack the box", "Easy", "Linux"]
date = 2023-04-29T11:00:00.000Z
publishDate = 2023-04-29T11:00:00.000Z
description = " "
summary = ""
draft = false
slug = "hack-the-box-keeper"
tags = ["hack the box", "Easy", "Linux", "KeePass", "dotnet", "CVE-2023-32784", "Request Tracker", "PuTTY"]
title = "Hack the Box - Keeper"
url = "/hack-the-box-keeper"
thumbnail = "/images/keeper/logo.png"
+++

Welcome back! Today we are going to be doing the Hack the Box seasonal machine - Keeper. This machine is listed as a easy Linux machine. So let's see what's in store. 

As usual, we start with a `rustscan` of the target. Here are the results:

```
PORT   STATE SERVICE REASON  VERSION
22/tcp open  ssh     syn-ack OpenSSH 8.9p1 Ubuntu 3ubuntu0.3 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    syn-ack nginx 1.18.0 (Ubuntu)
|_http-title: Site doesn't have a title (text/html).
| http-methods: 
|_  Supported Methods: GET HEAD
|_http-server-header: nginx/1.18.0 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

Not much to see here, let's take a look at port `80`. Right away we see a redirect link pointing to a hostname of `tickets.keeper.htb`. So, we'll add both `keeper.htb` and `tickets.keeper.htb` to our `host` file. After we do that, we'll start a `gobuster` to enumerate subdomains as well.

Command:
`gobuster dns -u http://keeper.htb -w /usr/share/wordlists/seclists/Pick/a/list -t 80`

Now while that runs we'll see whats on `tickets.keeper.htb`. We land on a login page. 

![](/images/keeper/keeper1.png)

This is a [`Request Tracker` portal](https://duckduckgo.com/?q=request+tracker+exploit&ia=web). Using the default username and password of `root`:`password`, we are able to log in. Browsing around the portal we are able to obtain some information from the pending ticket. Now we see that there was a crash dump file attached to a ticket, however that file has been removed and placed in the users `home` directory.

![](/images/keeper/keeper2.png)

We continue to look through the portal and are able to obtain some additional info about users. We have two users for the system `inorgaard` and `root`.

![](/images/keeper/keeper3.png)

When we click on `inorgaard` we see a `password` for initial use set - `Welcome2023`. Well, I wonder if there's a `password reuse` issue here. We try the same password for `SSH`? Well, sure enough, they do. We are able to login via `SSH` and obtain the `user.txt` flag.

![](/images/keeper/user.png)

Now we can access that `crash dump` from earlier as well. We can `unzip` the files and copy them to our local machine for extraction and examination. When we extract the life we get a `KeePassDumpFull.dmp`. Now if the version of `KeePass` running is below `2.54`, we can [extract the master password](https://github.com/vdohney/keepass-password-dumper) from this dump. Maybe there's a `root` password or `ssh key` in there. 

First we need to get `.NET 7` installed. 

Commands:
`wget https://packages.microsoft.com/config/debian/12/packages-microsoft-prod.deb -O packages-microsoft-prod.deb`
`sudo dpkg -i packages-microsoft-prod.deb`
`rm packages-microsoft-prod.deb`
`sudo apt-get update && sudo apt-get install -y dotnet-sdk-7.0`

Next we pull down our repo.

Commands:
`git clone https://github.com/vdohney/keepass-password-dumper`

Then we can run the `PoC`.

Command:
`dotnet run ~/htb/keeper/KeePassFullDump.dmp`

![](/images/keeper/keeper4.png)

Awesome! We now have an output with one wildcard in it - `●ødgrød med fløde`. Now we can try each of the suggested characters in the wildcard space in order to unlock the `database`. I actually DDG'd for the password without the wildcard and it came back with this:

![](/images/keeper/keeper5.png)

It looks delicious! That most likely solved our problem. `R` is the character we are looking for. Now we can use `KeePassxc` to load this database.

Command:
`sudo apt install keepassxc`
`keepassxc`

![](/images/keeper/keeper6.png)

Then we select the `passcodes.kdbx`.

![](/images/keeper/keeper8.png)

Enter the password we've discovered and we're in!

![](/images/keeper/keeper9.png)

Now, we see there's an `RSA` key used with `PuTTY` inside, as well as the password to match. In this case, we can simply copy the contents of the file out in to an `id_rsa` file. Next we can install `PuTTY`.

Command:
`sudo apt install putty`
`putty`

Then we launch putty and add our key file.

![](/images/keeper/keeper10.png)

Then we hit `open` and we are in!

![](/images/keeper/root.png)

We are able to log in as `root` and get the `root.txt` flag! 