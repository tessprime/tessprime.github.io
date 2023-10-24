---
title: "Snake"
date: 2023-05-11T19:45:22-07:00
tags: ["hacking"]
---

Well this is interesting. CISA has published an article
detailing information about the SNAKE malware, which they
have explicitly declared was created by Center 16 and is
controlled by the Russian FSB

https://www.cisa.gov/news-events/cybersecurity-advisories/aa23-129a

<!--more-->

Of note, they mention that they were able to compromise the network
due to the authentication protocol using a 128 bit prime for the Diffie
Hellman exchange.

Now, 128 bits is bad, but one might ask "How bad is it exactly?"

So the main algorithm that gets used is the Pohlig Hellman algorithm.

https://en.wikipedia.org/wiki/Pohlig%E2%80%93Hellman_algorithm

However, looking at the description on Wikipedia leads to the following issue:
Pohlig Hellman shouldn't be any better than Baby Step Giant Step when `p-1` is
the product of two and a prime. Thus, based on this, we would expect that
a 128 bit safe prime (a prime that is 1 more than twice another prime) would
still take `2**64` operations. This is big, but only about 31 years or so
of compute time (based on the assumption of 1 GFLOPs). This can be cracked
on the order of days with a decently funded operation.

However, `sage` takes only 3 minutes to pull this off on a single core machine.

Looking into the basic algorithm, sage doesn't seem to be doing anything other
than Pohlig Hellman.

So currently I'm digging into `sage` and `pari` to figure out what the heck is
going on. It seems pretty clear that I have missed something very fundamental
about how Pohlig Hellman works.
