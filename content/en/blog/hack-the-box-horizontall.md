+++
author = "Nick"
categories = ["hack the box", "easy", "Linux", "Laravel", "strapi", "reverse shell"]
date = 2022-02-05T16:04:52Z
description = ""
draft = false
image = "images/2021/10/horizontall.png"
slug = "hack-the-box-horizontall"
tags = ["hack the box", "easy", "Linux", "Laravel", "strapi", "reverse shell"]
title = "Hack the Box - Horizontall"

+++


Welcome back! Today we are going to be doing the Hack the Box machine - Horizontall. This is listed as an easy Linux machine. Let's jump in!

As usual, we kick it off with an `nmap` scan. Here are the results:

```
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 ee:77:41:43:d4:82:bd:3e:6e:6e:50:cd:ff:6b:0d:d5 (RSA)
|   256 3a:d5:89:d5:da:95:59:d9:df:01:68:37:ca:d5:10:b0 (ECDSA)
|_  256 4a:00:04:b4:9d:29:e7:af:37:16:1b:4f:80:2d:98:94 (ED25519)
80/tcp open  http    nginx 1.14.0 (Ubuntu)
|_http-server-header: nginx/1.14.0 (Ubuntu)
|_http-title: Did not follow redirect to http://horizontall.htb
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

It looks like the only way in is on port 80. Let's see what's being hosted. Since this box looks to be web access, we'll load up `Burpsuite` and work through that.

We see the request come throught to `horizontall.htb`, so we'll add that to our host file.

![](/images/2021/10/image.png)

Once we add this to the host file, we get a basic web page. Nothing is usable on the page, so we start our enumeration with `ffuf`.

Command:
`ffuf -u http://horizontall.htb/FUZZ -w /usr/share/seclists/Discovery/Web-Content/raft-medium-words-lowercase.txt -t 50`

Nothing comes back on this URL using multiple word lists. Now we can try and check against virutal hosts. After using `Gobuster` and `ffuf` I decided to check the source of the javascript. We find a virtual host listing:

{{< figure src="__GHOST_URL__/content/images/2021/10/image-1.png" >}}

We can add this host to our hostfile and see what it resolves to. When we request the link above, we get back a `json` response, as we would expect from an API endpoint.

{{< figure src="__GHOST_URL__/content/images/2021/10/image-2.png" >}}

Knowing this is a live endpoint, we can start to enumerate other potential endpoints.

Command:
`ffuf -u http://api-prod.horizontall.htb/FUZZ -w /usr/share/seclists/Discovery/Web-Content/api/actions-lowercase.txt`

We find an endpoint called `admin`.

{{< figure src="__GHOST_URL__/content/images/2021/10/image-3.png" >}}

Following this URL, takes us to a login page for Strapi.

{{< figure src="__GHOST_URL__/content/images/2021/10/image-4.png" >}}

A quick Google shows there are some RCE's available for this product. We need to check the version. We can do that by checking `/admin/init` endpoint as shown in our PoC code.

{{< figure src="__GHOST_URL__/content/images/2021/10/image-6.png" >}}

Now we some more research we find this post  on how the patch for this vulnerability was not properly verified. [https://thatsn0tmysite.wordpress.com/2019/11/15/x05/](https://thatsn0tmysite.wordpress.com/2019/11/15/x05/)

We take this PoC code and run it successfully!

{{< figure src="__GHOST_URL__/content/images/2021/10/image-5.png" >}}

We now try to login as Admin with our new password and we get in! Now that we have a foothold on the website, we have to find a way to gain server access. Some Googling shows us that there is an authenticated RCE to gain a remote shell - https://www.exploit-db.com/exploits/50238. There is a nice PoC listed [here](https://github.com/diego-tella/CVE-2019-19609-EXPLOIT) on Github.

We supply the requirements and fire it off!

{{< figure src="__GHOST_URL__/content/images/2021/10/image-7.png" >}}

Now we have a foothold on the server, we can try to enumerate and gain system access. The `strapi` user has the ability to read from the `devloper`'s home directory, and we snag the `user.txt` flag!

{{< figure src="__GHOST_URL__/content/images/2021/10/image-8.png" >}}

We copy over `linpeas` and start our internal enumeration. We see two additional ports being hosted - `8000` and `3306`.

{{< figure src="__GHOST_URL__/content/images/2021/10/image-9.png" >}}

As we see with additional enumeration, we can `ssh` in as `strapi`

{{< figure src="__GHOST_URL__/content/images/2021/10/image-10.png" >}}

We also have some passwords shown for the `mysql` database:

{{< figure src="__GHOST_URL__/content/images/2021/10/image-11.png" >}}

First, we will try to SSH in as `developer` using the credentials we just found - `#J!:F9Zt2u`. No dice. We'll just try and take the `strapi` key and use that to log in.

We copy out the RSA key located in `opt/strapi/.ssh` to our machine and connect with it.

Command:
`ssh -i id_rsa strapi@horizontall.htb`

{{< figure src="__GHOST_URL__/content/images/2021/10/image-12.png" >}}

We access the database with the given credentials but there is nothing additional to gain. We next `curl` the local host to see what's running on port 8000. 

Command:
`curl localhost:8000`

We see a platform called Larvel is running.

{{< figure src="__GHOST_URL__/content/images/2021/10/image-13.png" >}}

Another quick Google shows there are plenty of exploits for this platform. In order to actually leverage these exploits we need to forward the ports from the remote machine to our machine since the Laravel platform is only exposed internally.

Command:
`ssh -i id_rsa -L 8000:127.0.0.1:8000 strapi@horizontall.htb `

{{< figure src="__GHOST_URL__/content/images/2021/10/image-14.png" >}}

When we browse to port 8000 on our local machine we can now see the contents of the internally hosted Laravel system. Now we just need to pick an exploit and fire away!

Some Googling finds us this exloit - https://github.com/nth347/CVE-2021-3129_exploit. We will `clone` it onto our system and run it with the `ifconfig` command to see if it works.

Command:
`python3 exploit.py http://localhost:8000 Monolog/RCE1 ifconfig`

{{< figure src="__GHOST_URL__/content/images/2021/10/image-15.png" >}}

It does work! So now we can just capture the flag or attempt to create a remote session. I'm a simple man, I like getting the flag as fast as possible.

Command:
`python3 exploit.py http://localhost:8000 Monolog/RCE1 "cat /root/root.txt"`

{{< figure src="__GHOST_URL__/content/images/2021/10/image-16.png" >}}

