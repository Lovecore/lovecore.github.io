+++
author = "Nick"
categories = ["hack the box", "Easy"]
date = 2023-04-29T11:00:00.000Z
publishDate = 2023-04-29T11:00:00.000Z
description = " "
summary = ""
draft = false
slug = "hack-the-box-topology"
tags = ["hack the box", "Easy", "Linux", "Gobuster", "Sanitation", "RCE", "CVE-2002-0739", "WPScan", "John", "Passpie"]
title = "Hack the Box - Topology"
url = "/hack-the-box-topology"
thumbnail = "/images/topology/logo.png"
+++

Welcome! Today we are going to be doing the Hack the Box machine Topology. This machine is listed as an Easy Linux machine so let's see what's in store!

As always, start off with some basic enumeration.

`rustscan -a $TARGET -- -sV -sC`

Here are our results:

```bash
PORT   STATE SERVICE REASON  VERSION
22/tcp open  ssh     syn-ack OpenSSH 8.2p1 Ubuntu 4ubuntu0.7 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    syn-ack Apache httpd 2.4.41 ((Ubuntu))
|_http-title: Miskatonic University | Topology Group
| http-methods: 
|_  Supported Methods: HEAD GET POST OPTIONS
|_http-server-header: Apache/2.4.41 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

Well, we only see two options here. Let's see what's being hosted on port 80. We land and are greated with a page hosting some information. Taking a quick peek at the source we see a subdomain listed - `latex.topology.htb`. So we'll add the root domain and subdomain to our hostfile. After we've done that we're going to try and enumerate some other potential subdomains. In this case we'll use `gobuster`.

`gobuster dns -t 40 -w /usr/share/seclists/Discovery/DNS/bitquark-subdomains-top100000.txt -d topology.htb`

I ran this with a bunch of lists and came up with the following results:

```
latex.topology.htb
dev.topology.htb
stats.topology.htb
```

While that ran, we checked into the `latex` subdomain. We see a file listing:

![](/images/topology/top1.png)

Some interesting things listed here, like the `equationtest.log` that leaks a small bit of internal server info and the use of `pdfTeX`. Now we also have a demo site we can run. So we try to run some basic enumeration commands within `Latex` and are told we cannot because an Illegal Command Detected.

![](/images/topology/top2.png)

Since we see that we can only leverage one liners for this, it might not be helpful. Digging around we see there is a [`payloadallthethings` entry](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/LaTeX Injection) for `Latex`. The interesting part is that a the top it says we might need to wrap our commands.

```
You might need to adjust injection with wrappers as \[ or $.
```

Well since our first set of commands didn't run and gave us some detection, let's try wrapping the same ones again. Going back down the list we are able to get one command to read the `/etc/passwd` file!

`$\openin\file=/etc/passwd$` resulted in an output with data! 

![](/image/topology/top4.png)

So now we have a user on the box - `vdaisley`. Let's try to snag any `SSH` keys in the users home directory.

The command `$\lstinputlisting{/home/vdaisley/.ssh/id_rsa}$` didn't work. Now, circling back to the other subdomains we found. `dev.topology.htb` is password protected and `stats.topology.htb` is the standard stats page with `Apache`. Some quick Googling around shows how to setup [Authentication on directories](https://ubiq.co/tech-blog/password-protect-directory-apache/). So we need to gget the `.htpasswd` file from the `dev` site. According to the documents that path should be `/var/www/<SITE>`, let's try to read the file from that location. Here's our Latex command:

`$\lstinputlisting{/var/www/dev/.htpasswd}$`

Sure enough, that gives us a username and password!

![](/image/topology/top4.png)

`vdaisley:$apr1$1ONUB/S2$58eeNVirnRDB5zAIbIxTY0`

Now we can take that hash, slap it into as file called `hash` and give it to`john` and see what comes back.

Command:
`john --wordlist=/usr/share/wordlists/rockyou.txt ./hash`

![](/image/topology/top5.png)

There we go, `calculus20` is our password. Once we're in, we are greated with another landing page.

![](/image/topology/top6.png)

Nothing really in the source of the page but we should check this password against the `SSH` service on the machine... and it works!

![](/image/topology/user.png)

Perfect, we grab our `user.txt` flag and look for ways to escalate! Nothing is shown under the standard `sudo -l` so we'll copy over some `linpeas` and start enumerating.

In our `linpeas` directory we start a `simple HTTP server`:

`python3 -m http.server 80`

Then on our compromised machine, we use `wget` to pull it down. Change its permissions with `chmod +x` and let it run! There are a few things listed here but nothing that jumps right out. The next step is to launch `pspy64`. We copy it over in the same method as above and start to analyze its output.

There's a lot of data here but the item that really sticks out is the amount of times `GNUPlot` is being run and called.

![](/images/topology/top7.png)

Quick Googling returns [this](https://exploit-notes.hdks.org/exploit/linux/privilege-escalation/gnuplot-privilege-escalation/). We setup the PoC as shown and we indeed get a connection back as `vdaisley`.

![](/images/topology/top8.png)

So now, we need to leverage that by getting our script to run via the 'hidden job'. In this case we are able to simply copy our above PoC to the `/opt/gnuplot/` directory and wait for it to run!

Command:
`cp shell.plt /opt/gnuplot/shell.plt`

![](/image/topology/top9.png)

The reason this works is because of this line here:

![](/image/topology/why.png)

This is simply looking for things that end in `.plt` within the `/opt/gnuplot/` directory and calling `gnuplot` on them. 

We snag our `root.txt` flag and we are done!