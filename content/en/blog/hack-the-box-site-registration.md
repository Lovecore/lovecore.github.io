+++
author = "Nick"
categories = ["CTF", "NetSec", "invite", "Javascript", "api", "hack the box"]
date = 2019-05-31T13:20:01Z
description = ""
draft = false
thumbnail = "/images/2019/05/HTB.png"
slug = "hack-the-box-site-registration"
summary = "So it's been a while since I've posted something here. That's mostly due to the machine's that I've completed are not yet able to have walk through's released. So in the mean time, why not do a quick write up on how to actually get obtain a membership to Hack The Box"
tags = ["CTF", "NetSec", "invite", "Javascript", "api", "hack the box"]
title = "Hack The Box - Site Registration"
url = "/hack-the-box-site-registration"

+++


So it's been a while since I've posted something here. That's mostly due to the machine's that I've completed are not yet able to have walk through's released. So in the mean time, why not do a quick write up on how to actually get obtain a membership to Hack The Box.

If you aren't familiar with how [Hack The Box](https://www.hackthebox.eu) works, here's a quick rundown. You have to 'hack' the site to obtain a registration code. Once you've done that you are able to create an account and do all the fun things inside. I'll go over how we do that, so warning, if you want to do this yourself without any help, stop reading now!

When we head over to the Hack The Box site, we see a few things. A login options, info about the site and a join button. So we click the button to join and we are sent to a page that shows the following:

![](/images/2019/05/image.png" caption="Hack our way in? Well this should be fun!)

So the first thing anyone should do is to check our page source and see what we have listed in it. As we sift through the page source, we see some nothing things like CSRF tokens, WOT tokens and some things that seems a little different. We see a inviteapi.min.js and a calm.js. Lets see what those are about, the former seems pretty promising.

![](/images/2019/05/image-1.png)

So since it's a Javascript file, we can freely load it in our browser. So we drop in the whole path to the file to load it up. When we do, we see our JS file. As we parse through it, we see what seems to be some functions:

![](/images/2019/05/image-2.png" caption="verifyInviteCode, makeInviteCode are two functions we'll want to try.)

So if we open up our developer tools in our browser, we should be able to call this function. We load up the Developer tools and head to the console tab  and simply call the functions we want, in this case it's makeInviteCode().

![](/images/2019/05/image-3.png)

If all goes well, we should get a 200 response status and a token! We see the value 'enctype' which means encryption type. In this case an easy Base64, so lets decode that. There are a few ways to do this, the fastest is to probably head over to [https://base64decode.org](https://base64decode.org) and let the site do the magic decode for you. However, since we chose a path of learning in this career, lets put together a quick decode method in the browser to do this for us.

JavaScript has two functions already built to decode Base64 for us, easy right? We can use atob() and btoa(). So all we have to do is assign a variable for our token be got above and pass it into the atob() function to get what it says.

```js
var token = "SW4gb3JkZXIgdG8gZ2VuZXJhdGUgdGhlIGludml0ZSBjb2RlLCBtYWtlIGEgUE9TVCByZXF1ZXN0IHRvIC9hcGkvaW52aXRlL2dlbmVyYXRl"
```
We assign our token value to a variable called token.
```js
var decoded = atob(token);
```
We then call the atob function on the token and store the result in a variable called decoded.
```js
console.log(decoded)
```
We then output the decoded value to the console. Once we've done that we get the following.

![](/images/2019/05/image-4.png)

It's giving us a path to an API endpoint. There are a few ways to do this too, we can fire up Postman (which is generally what I use for API development and poking) or we can do it in a terminal (which is way easier).

In our terminal session we're going to use curl to make the request, all we do is issue curl -xpost https://www.hackthebox.eu/api/invite/generate. If that's been successful we get back another JSON with Base64 in it. We can then decode that again, using whatever method you want as posted above and you now have your invite code!

