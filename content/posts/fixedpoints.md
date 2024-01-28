---
title: "Fixed Points for RSA"
date: 2024-01-27T22:42:18-08:00
subtitle: You've forgotten the essence of factoring.
---

Suppose that you were merrily encrypting your data and sending
them to your friend whose public key he gave you and you happen
to find some data that remains unchanged after exponentiation.

That is, you have found a number $x$ such that $x^e=x mod n$.

You can now factor their number.

Now if you're like me, you may find this surprising.
After all, all this seems to have revealed nothing super interesting.
Sure, you now know that $\phi$ has $e-1$ as a divisor, but you also
knew that $\phi$ had the small factor of $2$ and nobody lost any sleep about that.

Perhaps you think, that there's some way to treat this thing as a group
generator, express it as power of your plaintext, and then attempt to factor
$\phi$ using a discrete log modulo $e-1$?

Oh, no, no, no, you're a smart guy, clearly picked up some flashy tricks,
but you made one crucial mistake. You forgot about the essence of factoring.
It's all about the *ring*.

$$
x^e = x mod n
x^e - x = 0 mod n
x*(x^{e-1} - 1 = 0 mod n
$$

That last expression is the product of two numbers, both of which we know,
that when multiplied together yield a multiple of $n$. This is somewhat
of importance in the factoring $n$.

I spent way too much time puzzled about how this works. *sigh*.