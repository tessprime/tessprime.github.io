---
title: "'Easy' HackTheBox Challenge"
date: 2023-09-20T17:25:58-07:00
tags: ["hack the box", "hacking"]
---

Well it turns out that I'm bad at even 'easy' HackTheBox Challenges. :)
<!--more-->
Did the cozyhosting, and ran into two things that I checked solutions for:

1. Proper enumeration. Tried gobuster. Didn't work. Tutorials use dirsearch.
1. Dehashing password. Tried to use john with rockyou, but didn't work.

Not too bad I suppose. Was able to, slowly, work my way through the whole exfiltration
parts. But tutorials seem to suggest I was mostly going in the right direction.

Going to check out some of the other tutorials to see if they use different approaches
or tools. Once the `josh` user is discovered, `hydra` could have been used directly
with rockyou (and possibly faster) to just brute force the ssh connection, so I'm
wondering if one of the tutorials just did that.
