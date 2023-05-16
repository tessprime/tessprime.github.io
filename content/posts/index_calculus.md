---
title: "Index Calculus"
date: 2023-05-16T02:33:39-07:00
math: true
---
Alright, so it turns out that Pari doesn't use just pohlig helman, or rather
it doesn't proceed use Baby Step Giant Step or Pohlard Rho for the discrete
log step on the reduced prime (I'll probably do a write up for those later,
they're cool!)

Rather, it uses the Index Calculus.

So, recall that when we attempt to solve the discrete log of a target number $h$ modulo a prime $p$,
in a base $g$, so what we're trying to do is find an $x$ such that $g^x=h\mod p$.

So anyhow the Index Calculus says:

**Index Calculus**: Okay, suppose that your number $h$ is $r$-smooth,
that is, it doesn't have any factors bigger than the $r$-th
prime. (Technically, this is an abuse of terminology, normally $r$-smooth
means that $r$ is the largest prime factor, rather than the $r$-th prime, but
we'll care more about the $r$-th prime here rather than its actual value).

**Me**: How is $r$ chosen?

**Index Calculus**: Will explain that in a bit. So anyhow, if $h$ isn't r-smooth, just reroll
it.

**Me**: *WHAT?!*

**Index Calculus**: We can always multiply $h$ by powers of $g$. Everytime this crosses the
$p$ boundary (remember we're dealing with this thing modulo $p$) we will get a new
number which has essentially a new random factorization. Eventually, we'll get a r-smooth
number.

**Me**: *blinks* We can **do** that?

**Index Calculus**: Sure, if the thing you're looking for is common enough.

**Me**: How common are smooth numbers?

**Index Calculus**: That's given by the $\psi$ function, which can be computed
using Dickman's $\rho$ function. (for some reason $\psi$ doesn't have a name,
but Dickman's $\rho$ does).

https://en.wikipedia.org/wiki/Dickman_function

**Me**: *blinks*. Okay. So we have spent a bunch of time rerolling $h$ into
a form where it's r-smooth. Now what?

**Index Calculus**: Okay, so if we take the log of this thing (assume
we found the modified $h$ after multiplying by $g$ s times)

$$
g^s h = 2^{a_1} 3^{a_2} \ldots p(r)^{a_r}
$$

(Where $p(r)$ is the $r$-th prime.). So applying log we obtain

$$
\log_g s + \log_g h = a_1 \log_g 2 + a_2 \log_g a_2 \ldots a_r \log_g p(r)
$$

Which is just a linear equation in $r$ variables (each $\log_g p_i$ is a variable).

**Me**: Wait...

**Index Calculus**: Now we just generate powers of $g$, select the ones that are $r$-smooth
until we have a linear independent set of equations. And we solve for each $\log p_i$. We
can now apply that to our equation for $\log_g h$.

Since Dickman-$\rho$ grows like the square root of the number of *bits* we do significantly
less work than Baby Step Giant Step or Pollard Rho, which only reduces the bits by 
half.

So that brings us to the selection of r. If we make r very large, we end up spending
more time factoring the numbers, but if we make it too small, than we spend more time
generating numbers. So we basically select r based how to manage these two constraints.
Also, as r increases, the step "then do linear algebra" may become more expensive (note
though, we're going to be dealing with *sparse* equations, so it'll be more efficient
than the generic N^3 situation). If we're memory constrained, like on GPU's, then
we're going to want to spend more time generating equations that have fewer variables.

Here's some nice notes from Andrew Sutherland

https://math.mit.edu/classes/18.783/2021/LectureNotes10.pdf

**Me**: Woah. Thanks Index Calculus!

******

So one of the really cool aspects of this is the idea that we don't need to use algorithms
that are guaranted to work when factoring numbers, if we get unlucky and get a number which
doesn't work with Pollard's $p$-1 algorithm, it doesn't matter! We can just try again
and get a more reasonable number. 

Anyhow, `pari` doesn't quite use the above explanation. Its doing something weird in
it's generating equation step that I still don't understand.
