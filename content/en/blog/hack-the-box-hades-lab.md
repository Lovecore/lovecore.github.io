+++
author = "Nick"
date = 2020-06-24T17:11:53Z
description = ""
draft = true
slug = "hack-the-box-hades-lab"
title = "Hack the Box - Hades Lab"

+++


Welcome everyone! This post will be different than the normal one machine posts that are here. This time I've written up the Hades lab. I'm sure it will be retired at some point! If you're reading this, that time is now!

Here we go!

We know there are only three machines on the network - `HADES-DC1`, `HADES-DEV`  and `HADES-WEB`. Regardless, I will scanned the entire subnet.

Command:
`nmap -oA hades_subnet 10.13.38.1/24` 

This way I can have the target's identified and then explicitly scan those three hosts after.



