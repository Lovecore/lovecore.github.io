+++
author = "Nick"
categories = ["hack the box", "wfuzz", "nmap scripts", "PHP", "enumeration", "magic method", "netcat", "reverse shell", "cron", "pspy64", "LinEnum", "Linux", "bash", "www-data", "gobuster"]
date = 2020-01-18T05:01:00Z
description = ""
draft = false
image = "__GHOST_URL__/content/images/2019/10/info-1.png"
slug = "hack-the-box-player"
summary = "Welcome! Today we are doing the machine Player on Hack the Box. Let's jump in!"
tags = ["hack the box", "wfuzz", "nmap scripts", "PHP", "enumeration", "magic method", "netcat", "reverse shell", "cron", "pspy64", "LinEnum", "Linux", "bash", "www-data", "gobuster"]
title = "Hack the Box - Player"

+++


Welcome! Today we are doing the machine Player on Hack the Box. Let's jump in!

Like we normally do with every CTF box, start with ```nmap -sC -sV -oA player_scan```. The results that come back are fairly small:

```
Nmap scan report for 10.10.10.145
Host is up (0.051s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 6.6.1p1 Ubuntu 2ubuntu2.11 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   1024 d7:30:db:b9:a0:4c:79:94:78:38:b3:43:a2:50:55:81 (DSA)
|   2048 37:2b:e4:31:ee:a6:49:0d:9f:e7:e6:01:e6:3e:0a:66 (RSA)
|   256 0c:6c:05:ed:ad:f1:75:e8:02:e4:d2:27:3e:3a:19:8f (ECDSA)
|_  256 11:b8:db:f3:cc:29:08:4a:49:ce:bf:91:73:40:a2:80 (ED25519)
80/tcp open  http    Apache httpd 2.4.7
|_http-server-header: Apache/2.4.7 (Ubuntu)
|_http-title: 403 Forbidden
Service Info: Host: player.htb; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 9.42 second
```

So we'll run the scan again but this time against all ports; ```nmap -sC -sV -T4 -p- -oA player_all_ports```.

```
Not shown: 65532 closed ports
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 6.6.1p1 Ubuntu 2ubuntu2.11 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   1024 d7:30:db:b9:a0:4c:79:94:78:38:b3:43:a2:50:55:81 (DSA)
|   2048 37:2b:e4:31:ee:a6:49:0d:9f:e7:e6:01:e6:3e:0a:66 (RSA)
|   256 0c:6c:05:ed:ad:f1:75:e8:02:e4:d2:27:3e:3a:19:8f (ECDSA)
|_  256 11:b8:db:f3:cc:29:08:4a:49:ce:bf:91:73:40:a2:80 (ED25519)
80/tcp   open  http    Apache httpd 2.4.7
|_http-server-header: Apache/2.4.7 (Ubuntu)
|_http-title: 403 Forbidden
6686/tcp open  ssh     OpenSSH 7.2 (protocol 2.0)
Service Info: Host: player.htb; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 27.12 seconds
```

These results just show one additional port open, ```6686```. We aren't able to ```SSH``` into these boxes. So we'll have to do some further enumeration. We can use the ```virtual host``` scanning function of ```nmap```.

We are going to assume that the hostname of the target machine has been set to ```player.htb```, since we didn't see it leak anywhere. We'll add it to our host file to make our lives easier. Next we'll fire off ```nmap``` with the ```vhosts``` script.

```
nmap -p 80 --script http-vhosts --script-args http-vhosts.domain=player.htb 10.10.10.145
```

We get back some much better results than a standard ```nmap``` scan.

```
Nmap scan report for 10.10.10.145
Host is up (0.052s latency).

PORT   STATE SERVICE
80/tcp open  http
| http-vhosts: 
| dev.player.htb : 200
| chat.player.htb : 200
|_125 names had status 403

Nmap done: 1 IP address (1 host up) scanned in 6.92 seconds
```

We see two vhosts listed, ```dev``` and ```chat```. Let's see what these are serving. ```dev``` has a login page using ```POST``` requests. ```chat``` has a chat transcript. We see that they make mention of a staging location that has some items exposed. Probably another virtual host. We'll need to do a better job of enumerating. Let's use ```wfuzz``` for this.

```
wfuzz -c -w /usr/share/wordlists/SecLists/Discovery/DNS/subdomains-top1million-110000.txt --hc 400,404,403 -H "Host: FUZZ.player.htb" -u http://player.htb -t 100
```

Lets break down this command a bit.
```-c``` for use colors.
```-w``` for specifying a wordlist.
```--hc``` will hide the responses we list, so in this case ```404```,```400```,```403```.
```-H``` tells ```wfuzz``` to use headers. So in this case we are fuzzing on the headers rsponse for vhosts.
```-u``` is our URL.
```-t``` dictates the amount of concurrent connections.

{{< figure src="__GHOST_URL__/content/images/2019/10/image-9.png" >}}

After we let it run, we see that there are three vhosts found. Just like we thought, there is a ```staging``` vhost as well. So we will now enumerate all four hostnames we have: ```player.htb```, ```dev.player.htb```, ```staging.player.htb``` and ```chat.player.htb```. We'll add that hostname to our ```hosts``` file and kick off the enumeration.

We'll use ```gobuster``` to enumeration each of the sites:
```
gobuster dir -r -x php -t 100 -u http://player.htb -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -o /home/user/Documents/htb/player/player_gobust.txt
```

We'll repeat this command for each of the four hosts. We get the following results

player.htb

{{< figure src="__GHOST_URL__/content/images/2019/10/image-13.png" caption="player.htb results" >}}

dev.player.htb

{{< figure src="__GHOST_URL__/content/images/2019/10/image-14.png" caption="dev.player.htb results" >}}

chat.player.htb

{{< figure src="__GHOST_URL__/content/images/2019/10/image-15.png" caption="chat.player.htb results" >}}

staging.player.htb

{{< figure src="__GHOST_URL__/content/images/2019/10/image-16.png" >}}

Now that we've done some enumeration, lets see what the staging site is hosting.

{{< figure src="__GHOST_URL__/content/images/2019/10/image-10.png" >}}

We are seemingly viewing production application, just being held in the staging location before deployment. We see two links, ```product updates``` and ```contact core team```. The ```Product Updates``` page seems to loads data from internal services every 5 seconds. We know that it's using ```PHP```. When we try to use the contact form we are redirected to an error page.

{{< figure src="__GHOST_URL__/content/images/2019/10/image-11.png" >}}

Time to load up ```Burpsuite``` and start looking at what's going on behind the scenes. When we look at the response that the contact form is giving us, we see some new information.

{{< figure src="__GHOST_URL__/content/images/2019/10/image-12.png" >}}

```Fix.php``` is a new page that didn't show up on our initial enumeration. When we ```curl``` that resource we get nothing back. There doesn't seem like there's too much going on in the staging page. Let's visit the ```/launcher``` path we found with our enumeration and see what that has going on.

{{< figure src="__GHOST_URL__/content/images/2019/10/image-17.png" >}}

This directs us to a countdown page. When we look at what the page is doing in ```Burpsuite``` we see that every time we submit an email we get a different request.

{{< figure src="__GHOST_URL__/content/images/2019/10/image-18.png" >}}

This might be useful. After looking through all of the enumeration we've done so far, there wasn't anything that particularly stuck out to me. When I went back to read the chat again, there was mention of a source code disclosure. Researching that we come across a [PHP Temporary File Source Code Disclosure](https://www.rapid7.com/db/vulnerabilities/http-php-temporary-file-source-disclosure). So if we try to access the temporary backup version of the code using a ```~```. We might be able to obtain more information.

{{< figure src="__GHOST_URL__/content/images/2019/10/image-19.png" >}}

We ```curl``` the temporary URL and we indeed get some PHP back!

{{< figure src="__GHOST_URL__/content/images/2019/10/image-20.png" >}}

We see the site is making use of ```JWT``` to give access. Lets look at what that token actually shows. We snag the ```cookie``` that we were getting via ```burpsuite``` previously and stick it into [JWT.io](http://jwt.io).

{{< figure src="__GHOST_URL__/content/images/2019/10/image-21.png" >}}

Now we have a a decoded token that matches what the source we saw earlier. So we should try to modify our token's validation signature to match the one listed that will grant us access. To do this we use the ```verify signature``` portion of JWT.io. If we enter our ```key``` found earlier (```_S0_R@nd0m_P@ss_```) into the verify signature it should change our token. When we do this we see that our token value returns to the original JWT values?! What we can take away from this is that if we take the ```access_code``` given in the leaked ```PHP``` source (```0E76658526655756207688271159624026011393```) and feed it into this JWT, we should get a new JWT value back.

{{< figure src="__GHOST_URL__/content/images/2019/10/image-22.png" >}}

Now that we have a new token value, lets set that value in ```Burpsuite``` and see where it gets us.

{{< figure src="__GHOST_URL__/content/images/2019/10/image-23.png" >}}

We get a location back in our response. lets visit that location and see what it might be.

{{< figure src="__GHOST_URL__/content/images/2019/10/image-26.png" >}}

An upload page for secure media. I uploaded some text files and they came back as 'file could not be found'. When we upload an image it does work. Probably some media filtering going on. When looking at the file we see the following:

{{< figure src="__GHOST_URL__/content/images/2019/10/image-27.png" >}}

Some googling around show a few ```FFMpeg``` exploits. [This](https://github.com/neex/ffmpeg-avi-m3u-xbin) and [this](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Upload%20Insecure%20Files/CVE%20Ffmpeg%20HLS) both seem like they might fit the bill (since they are practically the same thing). We can use this exploit to obtain files from the remote system. A good start might be ```/var/www/backup/service_config```.

{{< figure src="__GHOST_URL__/content/images/2019/10/image-28.png" >}}

We  generate the malicious AVI and upload it to the site to get our desired content. Our 'Buffed' media comes back as follows:

{{< figure src="__GHOST_URL__/content/images/2019/10/image-29.png" >}}

We now have a ```username``` and ```password```. We will continue to use this method to obtain the ```PHP``` source of ```dev```, ```fix.php``` and the ```apache``` sites locations. We can't get any contents of the ```fix.php``` most likley due to permissions. We do get the contents of the ```apache``` sites config.

{{< figure src="__GHOST_URL__/content/images/2019/10/image-30.png" >}}

We see another location called ```demo``` and it seems to route to ```dev```. Let's get the content of this directory ```index.php```. As we parse through the content we see a file called ```users.php```.

{{< figure src="__GHOST_URL__/content/images/2019/10/image-31.png" >}}

We'll get the content of that as well.

{{< figure src="__GHOST_URL__/content/images/2019/10/image-32.png" >}}

We now have another ```username``` and a hashed or encrypted ```password```. This time for another user, ```peter```.

Neither set of credentials work on the ```dev``` login page. So now we'll use them on our ```6686``` ```SSH``` port. Looks like the ```telegen``` credentials work out.

{{< figure src="__GHOST_URL__/content/images/2019/10/image-33.png" caption="" >}}

On login we get some data about our session. We see that we are in ```lshell``` or ```limited shell```. Most commands are forbidden in this shell scenario. We know that we're running OpenSSH 7.2. Some searching around shows that we have a relevant vulnerability. ```CVE-2016-3115```. [Here](https://www.exploit-db.com/exploits/39569) is an exploitDB posting and this is the [corresponding GitHub](https://github.com/tintinweb/pub/tree/master/pocs/cve-2016-3115). We can use the POC provided in the GitHub to test if it is indeed vulnerable.

{{< figure src="__GHOST_URL__/content/images/2019/10/image-34.png" >}}

It indeed is vulnerable. So we'll use it to snage our users.txt file. We'll use this process to enumerate files on the system. After enumerating other system files we will look at the ```fix.php``` file.

{{< figure src="__GHOST_URL__/content/images/2019/10/image-35.png" >}}

We now have some credentials for Peter. They don't seem to work for ```SSH```. They might work in our ```dev``` login portal.

{{< figure src="__GHOST_URL__/content/images/2019/10/image-36.png" >}}

They do indeed work. We are greated with a web based IDE called [Codiad](https://github.com/Codiad/Codiad/wiki). After reading the documentation a bit, we will create a new project in the IDE and set its path to the ```tmp``` directory. When we try to do this, we get a promp that says it should be hosted in ```/var/www/demo/home```. So we make that the path and create the project.

{{< figure src="__GHOST_URL__/content/images/2019/10/image-37.png" >}}

From here we can create a new ```PHP``` file which will be our ```reverse shell```.

{{< figure src="__GHOST_URL__/content/images/2019/10/image-38.png" >}}

{{< figure src="__GHOST_URL__/content/images/2019/10/image-39.png" >}}

We save our configurations for our shell file. Start up a ```netcat``` listener and browse to our newly created file.

{{< figure src="__GHOST_URL__/content/images/2019/10/image-41.png" >}}

We get a response back as ```www-data``` as expected. Now we can start enumerating the system and see what we can leverage. ```LinEnum``` shows a cron tab running ```PHP5``` frequently. I didn't see any further usage of it in the report so we'll launch ```pspy64``` to see if we can catch what it might be.

{{< figure src="__GHOST_URL__/content/images/2019/10/image-42.png" >}}

This looks to be the process being called. When we look at the source of the file we see that a database call is being processed every time someone uploads a file to the media system. Since this file is launched as ```root``` we should be able to append a ```reverse shell``` onto the ```PHP``` file in place already and obtain a ```reverse shell``` as ```root``` when it fires off.

We also see that there is an ```unserialized``` function in the ```buff.php``` file.

{{< figure src="__GHOST_URL__/content/images/2019/10/image-43.png" >}}

The above is considered a ```magic method```. You can read more about them [here](https://www.php.net/manual/en/language.oop5.magic.php). If we feed the ```___wakeup()``` function serialized data, it will run it. In this case we can get the function to run the content we put into ```merge.log```.

{{< figure src="__GHOST_URL__/content/images/2019/10/image-44.png" caption="Here is our payload." >}}

The above payload will add the ```telegen``` user to the ```sudoers``` file with all the access we need. next we need to convert it to a ```serialized string```. Once we do that, we will add the code to the ```merge.log```. When we try to do it as ```www-data``` we see that we don't have permissions but ```telegen``` does. We can switch users by calling a new shell as ```telegen```: ```su -s /bin/bash -c '/bin/bash' telegen```. Once we've switched our user we can ```echo``` our command into the ```merge.log``` file.

{{< figure src="__GHOST_URL__/content/images/2019/10/image-45.png" >}}

Now you can upgrade our shell with the python upgrade trick: ```python -c 'import pty;pty.spawn("/bin/bash")'```. We should now be able to ```sudo su``` to ```root```.

{{< figure src="__GHOST_URL__/content/images/2019/10/image-46.png" >}}

There we have it, our root flag!

Hopefully something was learned. If you found this write-up helpful, consider sending some respect my way: [Lovecore's HTB Profile](https://www.hackthebox.eu/home/users/profile/95635).

