+++
author = "Nick"
categories = ["hack the box", "Linux", "HA Proxy", "CVE-2019-18277", "Request Smuggling", "Insane"]
date = 2021-07-21T13:22:25Z
description = ""
draft = true
slug = "hack-the-box-sink"
tags = ["hack the box", "Linux", "HA Proxy", "CVE-2019-18277", "Request Smuggling", "Insane"]
title = "Hack the Box - Sink"
url = "/hack-the-box-sink"

+++


Welcome! Today's write-up is for the Hack the Box machine - Sink. This machine is listed as an Insane Linux machine. Let's see what's in store!

As usual we kick it off with an `nmap` scan. Here are our results:

```
Nmap scan report for 10.10.10.225
Host is up (0.050s latency).
Not shown: 65532 closed ports
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.1 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 48:ad:d5:b8:3a:9f:bc:be:f7:e8:20:1e:f6:bf:de:ae (RSA)
|   256 b7:89:6c:0b:20:ed:49:b2:c1:86:7c:29:92:74:1c:1f (ECDSA)
|_  256 18:cd:9d:08:a6:21:a8:b8:b6:f7:9f:8d:40:51:54:fb (ED25519)
3000/tcp open  ppp?
| fingerprint-strings: 
|   GenericLines, Help: 
|     HTTP/1.1 400 Bad Request
|     Content-Type: text/plain; charset=utf-8
|     Connection: close
|     Request
|   GetRequest: 
|     HTTP/1.0 200 OK
|     Content-Type: text/html; charset=UTF-8
|     Set-Cookie: lang=en-US; Path=/; Max-Age=2147483647
|     Set-Cookie: i_like_gitea=92193e16fa35e1a2; Path=/; HttpOnly
|     Set-Cookie: _csrf=LDtKdcZjmP8q7Cxz9AQDosGgdzw6MTYyNjg3NDYxNjg1NjE0Nzc4OQ; Path=/; Expires=Thu, 22 Jul 2021 13:36:56 GMT; HttpOnly
|     X-Frame-Options: SAMEORIGIN
|     Date: Wed, 21 Jul 2021 13:36:56 GMT
|     <!DOCTYPE html>
|     <html lang="en-US" class="theme-">
|     <head data-suburl="">
|     <meta charset="utf-8">
|     <meta name="viewport" content="width=device-width, initial-scale=1">
|     <meta http-equiv="x-ua-compatible" content="ie=edge">
|     <title> Gitea: Git with a cup of tea </title>
|     <link rel="manifest" href="/manifest.json" crossorigin="use-credentials">
|     <meta name="theme-color" content="#6cc644">
|     <meta name="author" content="Gitea - Git with a cup of tea" />
|     <meta name="description" content="Gitea (Git with a cup of tea) is a painless
|   HTTPOptions: 
|     HTTP/1.0 404 Not Found
|     Content-Type: text/html; charset=UTF-8
|     Set-Cookie: lang=en-US; Path=/; Max-Age=2147483647
|     Set-Cookie: i_like_gitea=0b8f838e032f43d6; Path=/; HttpOnly
|     Set-Cookie: _csrf=GJTire8w7XZ0orjnoE7t9jfnOO06MTYyNjg3NDYyMjExODg3NDU0OA; Path=/; Expires=Thu, 22 Jul 2021 13:37:02 GMT; HttpOnly
|     X-Frame-Options: SAMEORIGIN
|     Date: Wed, 21 Jul 2021 13:37:02 GMT
|     <!DOCTYPE html>
|     <html lang="en-US" class="theme-">
|     <head data-suburl="">
|     <meta charset="utf-8">
|     <meta name="viewport" content="width=device-width, initial-scale=1">
|     <meta http-equiv="x-ua-compatible" content="ie=edge">
|     <title>Page Not Found - Gitea: Git with a cup of tea </title>
|     <link rel="manifest" href="/manifest.json" crossorigin="use-credentials">
|     <meta name="theme-color" content="#6cc644">
|     <meta name="author" content="Gitea - Git with a cup of tea" />
|_    <meta name="description" content="Gitea (Git with a c
5000/tcp open  http    Gunicorn 20.0.0
|_http-server-header: gunicorn/20.0.0
|_http-title: Sink Devops
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port3000-TCP:V=7.91%I=7%D=7/21%Time=60F82005%P=x86_64-pc-linux-gnu%r(Ge
SF:nericLines,67,"HTTP/1\.1\x20400\x20Bad\x20Request\r\nContent-Type:\x20t
SF:ext/plain;\x20charset=utf-8\r\nConnection:\x20close\r\n\r\n400\x20Bad\x
SF:20Request")%r(GetRequest,2943,"HTTP/1\.0\x20200\x20OK\r\nContent-Type:\
SF:x20text/html;\x20charset=UTF-8\r\nSet-Cookie:\x20lang=en-US;\x20Path=/;
SF:\x20Max-Age=2147483647\r\nSet-Cookie:\x20i_like_gitea=92193e16fa35e1a2;
SF:\x20Path=/;\x20HttpOnly\r\nSet-Cookie:\x20_csrf=LDtKdcZjmP8q7Cxz9AQDosG
SF:gdzw6MTYyNjg3NDYxNjg1NjE0Nzc4OQ;\x20Path=/;\x20Expires=Thu,\x2022\x20Ju
SF:l\x202021\x2013:36:56\x20GMT;\x20HttpOnly\r\nX-Frame-Options:\x20SAMEOR
SF:IGIN\r\nDate:\x20Wed,\x2021\x20Jul\x202021\x2013:36:56\x20GMT\r\n\r\n<!
SF:DOCTYPE\x20html>\n<html\x20lang=\"en-US\"\x20class=\"theme-\">\n<head\x
SF:20data-suburl=\"\">\n\t<meta\x20charset=\"utf-8\">\n\t<meta\x20name=\"v
SF:iewport\"\x20content=\"width=device-width,\x20initial-scale=1\">\n\t<me
SF:ta\x20http-equiv=\"x-ua-compatible\"\x20content=\"ie=edge\">\n\t<title>
SF:\x20Gitea:\x20Git\x20with\x20a\x20cup\x20of\x20tea\x20</title>\n\t<link
SF:\x20rel=\"manifest\"\x20href=\"/manifest\.json\"\x20crossorigin=\"use-c
SF:redentials\">\n\t<meta\x20name=\"theme-color\"\x20content=\"#6cc644\">\
SF:n\t<meta\x20name=\"author\"\x20content=\"Gitea\x20-\x20Git\x20with\x20a
SF:\x20cup\x20of\x20tea\"\x20/>\n\t<meta\x20name=\"description\"\x20conten
SF:t=\"Gitea\x20\(Git\x20with\x20a\x20cup\x20of\x20tea\)\x20is\x20a\x20pai
SF:nless")%r(Help,67,"HTTP/1\.1\x20400\x20Bad\x20Request\r\nContent-Type:\
SF:x20text/plain;\x20charset=utf-8\r\nConnection:\x20close\r\n\r\n400\x20B
SF:ad\x20Request")%r(HTTPOptions,206D,"HTTP/1\.0\x20404\x20Not\x20Found\r\
SF:nContent-Type:\x20text/html;\x20charset=UTF-8\r\nSet-Cookie:\x20lang=en
SF:-US;\x20Path=/;\x20Max-Age=2147483647\r\nSet-Cookie:\x20i_like_gitea=0b
SF:8f838e032f43d6;\x20Path=/;\x20HttpOnly\r\nSet-Cookie:\x20_csrf=GJTire8w
SF:7XZ0orjnoE7t9jfnOO06MTYyNjg3NDYyMjExODg3NDU0OA;\x20Path=/;\x20Expires=T
SF:hu,\x2022\x20Jul\x202021\x2013:37:02\x20GMT;\x20HttpOnly\r\nX-Frame-Opt
SF:ions:\x20SAMEORIGIN\r\nDate:\x20Wed,\x2021\x20Jul\x202021\x2013:37:02\x
SF:20GMT\r\n\r\n<!DOCTYPE\x20html>\n<html\x20lang=\"en-US\"\x20class=\"the
SF:me-\">\n<head\x20data-suburl=\"\">\n\t<meta\x20charset=\"utf-8\">\n\t<m
SF:eta\x20name=\"viewport\"\x20content=\"width=device-width,\x20initial-sc
SF:ale=1\">\n\t<meta\x20http-equiv=\"x-ua-compatible\"\x20content=\"ie=edg
SF:e\">\n\t<title>Page\x20Not\x20Found\x20-\x20\x20Gitea:\x20Git\x20with\x
SF:20a\x20cup\x20of\x20tea\x20</title>\n\t<link\x20rel=\"manifest\"\x20hre
SF:f=\"/manifest\.json\"\x20crossorigin=\"use-credentials\">\n\t<meta\x20n
SF:ame=\"theme-color\"\x20content=\"#6cc644\">\n\t<meta\x20name=\"author\"
SF:\x20content=\"Gitea\x20-\x20Git\x20with\x20a\x20cup\x20of\x20tea\"\x20/
SF:>\n\t<meta\x20name=\"description\"\x20content=\"Gitea\x20\(Git\x20with\
SF:x20a\x20c");
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

So we have ports `5000`,`3000` and `22`. It looks that both of the high en ports are hosting some kind of web service. `Gunicorn` on `5000` and `Gitea` on `3000`. `Gitea` is running version 1.12.6. A quick `searchsploit` for the product and we don't have anything for that version at first glance. Next we check on `Gunicorn`, also nothing at first glance there either. Let's start enumerating the sites to see what we might be able to dig up, it's possible there are some `vhosts` available if we can sniff out a hostname.

While attempting to enumerate potential directories, we kept getting rate limited while using `ffuf`, this would lead you to think there is some kind of WAF or proxy in front of the services. Let's open up `Burpsuite` and see what we can find.

We'll start with port `5000` since it seems to have a much more rich attack surface given user accounts can be created. With `Burpsuite` running, we'll attempt to create an account:

![](/images/2021/07/image-43.png)

While viewing the requests, we see that the page is being served up via `haproxy`

![](/images/2021/07/image-44.png)

Once logged in, we take a look around. We have a comment field, a search field and a hostname - `sink.htb`. We'll add this to our hosts file and check for `vhosts`.

Command:
`ffuf -u http://sink.htb -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-110000.txt -H "Host: FUZZ.sink.htb" -c`

This was met with mostly errors. There is a notes section with the ability to save there as well. We test this field out to spot any noticable vulnerabilities - nothing found. The site is pretty basic and straight forward, let's look at the `Gitea`to see what we might be able to find here.

After sifting through `Gitea` I wasn't able to find anything the stood out. Back to the drawing board. The noticable difference in these two sites is that the site hosted on `5000` is leveraging `haproxy` and `3000` is not. Searching around we see that `haproxy` has had a [few vulernabilities](https://www.cvedetails.com/product/22372/Haproxy-Haproxy.html?vendor_id=11969) in it's lifetime. I googled 'determine ha proxy version via burp suite' and found this [YouTube video](https://www.youtube.com/watch?v=nq0ndhkfV_M). This seems like it could be a path forward for us.

The main cuplrit here is [CVE-2019-18277](https://nvd.nist.gov/vuln/detail/CVE-2019-18277). The catch is, we need to use this technique to obtain something of use.



