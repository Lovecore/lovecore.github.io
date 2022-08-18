+++
author = "Nick"
categories = ["hack the box", "Hard", "Linux", "burpsuite", "python", "cryptography", "ffuf", "gobuster", "php filters", "LFI"]
date = 2020-07-04T14:55:00Z
description = ""
draft = false
image = "__GHOST_URL__/content/images/2020/06/info.png"
slug = "hack-the-box-forward-slash"
summary = "Hey everyone, we are back with another Hack the Box machine. This write-up is for the machine Forward Slash. It's listed as a Hard Linux machine, let's jump in!"
tags = ["hack the box", "Hard", "Linux", "burpsuite", "python", "cryptography", "ffuf", "gobuster", "php filters", "LFI"]
title = "Hack the Box - Forward Slash"

+++


Hey everyone, we are back with another Hack the Box machine. This write-up is for the machine Forward Slash. It's listed as a Hard Linux machine, let's jump in!

As usual, we start with our `nmap` scan: `nmap -sC -sV -p- -oA allscan 10.10.10.183`

Here are the results:
```
Nmap scan report for 10.10.10.183
Host is up (0.045s latency).
Not shown: 65533 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 3c:3b:eb:54:96:81:1d:da:d7:96:c7:0f:b4:7e:e1:cf (RSA)
|   256 f6:b3:5f:a2:59:e3:1e:57:35:36:c3:fe:5e:3d:1f:66 (ECDSA)
|_  256 1b:de:b8:07:35:e8:18:2c:19:d8:cc:dd:77:9c:f2:5e (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Did not follow redirect to http://forwardslash.htb
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

We'll, it looks like the path forward is on the web interface somewhere, let's look. We do see that forwardslash.htb is noted in our scan so we'll add that to our hosts file. We are greated with this awesome landing page which seems like the site has been defaced.

{{< figure src="__GHOST_URL__/content/images/2020/06/image-25.png" >}}

We see some hints about FTP logins and XML. So we'll start with a `gobuster` on the site to see what might turn up.

Command:
`gobuster dir -u http://forwardslash.htb -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php,txt,xml -t 50`

We immediatly see a `note.txt` file saying there is a backup and should be fine. With some luck we'll be able to find that backup and assess it for further exploitation. We re-run the enumeration process looking for `.zip` and `.bak` files but don't find anything.

Generally if `Gobuster` doesn't show much, we'll want to enumerate subdomains on the target. We'll use [`ffuf`](https://github.com/ffuf/ffuf) for this.

Command:
`ffuf -c -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt -u http://forwardslash.htb -H "Host: FUZZ.forwardslash.htb" -o ffuff_result.txt -of csv`

I had to run this a few times with a few lists to catch what we're looking for. In our results we see `backup.forwardslash.htb` gives us an error of 302. If we were hiding those results, we might have missed it. We can see that the size tells us a different story.

{{< figure src="__GHOST_URL__/content/images/2020/06/image-26.png" >}}

Now if we had this virtual host to our hosts file and navigate to the page. We are taken to a login screen.

{{< figure src="__GHOST_URL__/content/images/2020/06/image-27.png" >}}

We should probably enumerate this subdomain as well.

Command:
`gobuster dir -u backup.forwardslash.htb -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -t 30 -x txt,zip,php,bak`

We see an interesting list of objects found as well:

{{< figure src="__GHOST_URL__/content/images/2020/06/image-29.png" >}}

We look at thes files and there is nothing in them aside from a note saying they've removed the option to upload pictures. The `/dev` seems promising but we can't get there just yet. When we login we have a small dashboard.

{{< figure src="__GHOST_URL__/content/images/2020/06/image-28.png" >}}

While browsing we see that the function for changing a profile picture is disabled, like we noted earlier. If we modify the page to allow us to use it, we might be able to leverage it further. We are able to read files locally using `file://`.

{{< figure src="__GHOST_URL__/content/images/2020/06/image-30.png" >}}

Maybe we can read files using `backup.forwardslash.htb`. We try to read the `config.php` file and it fails. We can try to use the same original approach to reading the file directly in `Burpsuite`. When we send our `POST` request to repeater and use a filter on the post request we get a response back!

{{< figure src="__GHOST_URL__/content/images/2020/06/image-31.png" >}}

Now we can use this method to gain some basic structure knowledge. We still want the config file, we're going to assume it's in the basic `/var/www/` location. Nope. Since it's a virtual host, it could be located in the virtual hosts name.

{{< figure src="__GHOST_URL__/content/images/2020/06/forwardslash_web_Cred.gif" >}}

This works and we get our `config.php` content back. Inside we have some credntials for the `www-data` user. While we're at it we'll grab the conents of `api.php` and `hof.php`.

{{< figure src="__GHOST_URL__/content/images/2020/06/image-33.png" >}}

Denied. We can use `php://filter`, noted [here](https://github.com/foobarto/redteam-notebook) to obtain the info.

{{< figure src="__GHOST_URL__/content/images/2020/06/image-34.png" >}}

When we look at the `api.php` code, we see our `session_start()` is required. We repeat these steps for all fo the directories / indexies we've found. The one of note is from `/dev/index.php`. We get our source back and see there are some credentials hardcoded.

{{< figure src="__GHOST_URL__/content/images/2020/06/image-35.png" >}}

We can now login via `SSH` as Chiv. When we do we see we have a low level account. It looks like our next goal is to get to `Pain`'s account.

{{< figure src="__GHOST_URL__/content/images/2020/06/image-36.png" >}}

We'll download `linpeas` from our machine onto our target and start our internall enumeration. One item on the standard output shows interest:

{{< figure src="__GHOST_URL__/content/images/2020/06/image-37.png" >}}

This application owned by our target. When we cat the application we get some useful data.

{{< figure src="__GHOST_URL__/content/images/2020/06/image-38.png" >}}

{{< figure src="__GHOST_URL__/content/images/2020/06/image-39.png" >}}

When we try to run the `backup` application we get the following:

{{< figure src="__GHOST_URL__/content/images/2020/06/image-41.png" >}}

Every time we run this application we get a new MD5 Sum back. Assumably linked to the time run. When we look back at our `linpeas` results we see a backup file owned by `Pain` as well.

{{< figure src="__GHOST_URL__/content/images/2020/06/image-42.png" >}}

So the thought here is that if we feed it this backup file with the appropriate MD5, we can 'decrypt' the backup. We need to get the MD5 of the current time and link that to our target file:
```bash
t="$(date +%H:%M:%S | tr -d '\n' | md5sum | tr -d ' -')"
ln -s /var/backups/config.php.bak /home/chiv/$t
backup
```

A quick breakdown:
The first line is to give the current date / time and MD5 Sum.
Next we link the target of `config.php.bak` to that time variable.
Then we call the backup tool.

We run it and get our password:

{{< figure src="__GHOST_URL__/content/images/2020/06/image-44.png" >}}

Now that we have a password, we can simply `su` to `Pain`'s account.

{{< figure src="__GHOST_URL__/content/images/2020/06/image-43.png" >}}

Now that we're another users, we'll rerun our enumeration script and see if anything new appears. While it runs, we'll also look at the files in the home directory.

We see that root owns these files but we can still read them. Here's the code for the `encrypter.py`.

```python
def encrypt(key, msg):
    key = list(key)
    msg = list(msg)
    for char_key in key:
        for i in range(len(msg)):
            if i == 0:
                tmp = ord(msg[i]) + ord(char_key) + ord(msg[-1])
            else:
                tmp = ord(msg[i]) + ord(char_key) + ord(msg[i-1])

            while tmp > 255:
                tmp -= 256
            msg[i] = chr(tmp)
    return ''.join(msg)

def decrypt(key, msg):
    key = list(key)
    msg = list(msg)
    for char_key in reversed(key):
        for i in reversed(range(len(msg))):
            if i == 0:
                tmp = ord(msg[i]) - (ord(char_key) + ord(msg[-1]))
            else:
                tmp = ord(msg[i]) - (ord(char_key) + ord(msg[i-1]))
                tmp += 256
            while tmp < 0:
            msg[i] = chr(tmp)
    return ''.join(msg)


print encrypt('REDACTED', 'REDACTED')
print decrypt('REDACTED', encrypt('REDACTED', 'REDACTED'))
```

We can brute force this code with the following:



And we get this message:

> ey: ttttttttttttttttt, Msg: Hl��vF��;�������&you liked my new encryption tool, pretty secure huh, anyway here is the key to the encrypted image from /var/backups/recovery: `cB!6%sdH8Lj^@Y*$C2cf`

We now have a password to this recovery image! We saw in our previous enumeration that `Pain` can run some software with privileges.

{{< figure src="__GHOST_URL__/content/images/2020/06/image-45.png" >}}

Perfect, now all we have to do is use `luksOpen` to mount the file.

Command:
`sudo /sbin/cryptsetup luksOpen /var/backups/recovery/encrypted_backup.img backup`

Enter the password, then mount it.

Command:
`sudo /bin/mount /dev/mapper/backup ./mnt/`

{{< figure src="__GHOST_URL__/content/images/2020/06/image-47.png" >}}

Once we mount the device, we are given an `id_rsa` file owned by root. We'll copy the file to our machine. Change its permissions and use it to log in as root!

We log in as root and snag our flag! Box completed! 

Hopefully you enjoyed this machine as much as I did. Feel free to send some respect my way on [HTB](https://www.hackthebox.eu/home/users/profile/95635) if you found it helpful!



