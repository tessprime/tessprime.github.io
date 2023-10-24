---
title: "Systems of Xor"
subtitle: "How likely is a random matrix over $\\mathbb{Z}_2$ to be invertible?"
date: 2023-10-23T10:46:45-07:00
math: true
---
In a recent CTF, we were given the ability to obtain the signatures of
arbitrary messages, and we were given the signature of the flag.
<!--more-->

In this particular case, a critical property of the signature function
allowed us to recover the flag fairly easily, without worrying about
the details of the signature function (vastly simplifying the analysis).

That detail was linearity under xor and bitwise multiplication.

$$ L(a \oplus b\otimes c) = L(a) \oplus (L(b) \otimes c) $$


One very useful property of this is that since we can express
any n bit vector as a xor-sum of vectors which have exactly
one 1-bit, we can easily compute the signature for a given
$x$.

$$
\begin{matrix}
L(x) = L(x \otimes (\dots 001))
\oplus L(x \otimes (\dots 010))
\oplus \dots
\oplus L(x \otimes(010\dots))
\oplus L(x \otimes(100\dots))
\end{matrix}
$$

Note, that $x \otimes (\dots 0{b_i}0 \dots)$ will be zero unless
$x$ has the $i$-th bit set. So we just need to know the
values of $L$ on the 1-bit basis.

Now let's look at the individual components

$$
\begin{matrix}
s_1 &=& x_1 L_1(b_1) \oplus x_2 L_1(b_2) \oplus \dots \oplus x_n L_1(b_n) \\\\
s_2 &=& x_1 L_2(b_1) \oplus x_2 L_2(b_2) \oplus \dots \oplus x_n L_2(b_n) \\\\
&\dots&\\\\
s_n &=& x_1 L_n(b_1) \oplus x_2 L_n(b_2) \oplus \dots \oplus x_n L_n(b_n) \\\\
\end{matrix}
$$

If $x$ is known, we can compute the signature, but if the signature is known,
we can compute $x$ by solving the matrix.

However, this doesn't always work. When I ran my attack script against a local
version, it failed at first. Running it a few more times revealed that it
worked *sometimes*. In retrospect this should be expected as the rows of
the matrix are essentially chosen at random, there's no reason to assume
that they will be linearly independent, but the algorithm used by sage will
still produce *some* value after it peforms elimination.

So, the question then becomes, if the signature function is basically random,
what is the probability that the resulting matrix is solveable?

Well, if we think of the rows as adding a new vector each time, then when
we add the final vector, assuming that we are linearly independent up until
then, what's the probability that the new vector is linearly independent?

Since the new vector can be paired with every other vector in the previously
paired space, and this will then span the entire space, we argue that the
remaining space not containing the vector must be half the size of the original
space. So we see that we have a 50% chance of picking a vector from remaining
space.

With a little bit of basis remanipulation, we can see this will make the probability
of picking a redundant vector double each time. Working backwards, this means that the
second vector will have a probability of $2^{-(n-1)}$ of being linearly dependent on the
first (this is not suprising given that there is clearly exactly two linearly
independent vectors after the first, the original vector and zero). The probability
thus of them being **independent** will be $1-2^{-n-1}$

We can put together a simple python routine to compute this for large $n$

```py
acc = 1
for i in range(1, 1024):
    acc *= 1 - 2**(-i)
print(acc)
```

This saturates to a value of 28.87%. This doesn't scream any obvious
approximation, and I only have emprical evidence to suggest that this actually
converges to a fixed number rather than *very* slowly converge to zero.

Time to dust off ye olde calculus series tests.

<!-- TODO: Prove that the answer converges quickly -->
