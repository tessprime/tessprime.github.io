---
title: "The SHA of this tweet"
date: 2023-09-11T13:28:30-07:00
math: true
---

So there's this cool tweet:
[![Sha Tweet](/images/sha_tweet.png)](https://twitter.com/lauriewired/status/1700982575291142594)

So an obvious question is: "How likely is it that a tweet of this form exists for a given
length?"

To be precise, we define our sentence to be of the form:

```python
rawchars = list("0123456789abcdef")
digits = ["zero", "one", "two", "three", "four", "five", "six", "seven", "eight", "nine"]
char_map = { a:b for (a,b) in zip(rawchars, digts + "abcdef")}
def sentence(hash_prefix, sentence_prefix=SENTENCE_PREFIX):
    return sentence_prefix + ", ".join([char_map[p] for p in hash_prefix]) + "."
```

Where `SENTENCE_PREFIX` is `"The SHA256 for this sentence begins with: "`.

We can easily brute force this with:

```python
def find_n_prefix(n, sentence_prefix=SENTENCE_PREFIX):
    for p in product(*([rawchars] * n)):
        s = sentence(p, sentence_prefix)
        h = hashlib.sha256(s.encode("ascii")).hexdigest()
        found = True
        for c, ch in zip(p,h):
            if c != ch:
                found = False
                break
        if found:
            print(s, h)
```

So what is the actual probability? Well, for any given sentence, the 
probabilty of it having any given hash prefix of length $l$ is $\frac{1}{16^l}$.
But we *also* have $16^l$ hashes. So we're basically rolling an $n$-sided die $n$
times. Which means that our probability of *not* getting a hash is
$$p(n)=(1-\frac{1}{n})^n$$
which for large $n$ (which we have very quickly) is just $e^{-1}$. Thus the probability
of such a hash existing is $1-e^{-1}$ or about 2/3rds.

So now that we know what the probability of a *fixed* sentence form is, we can estimate
that we only need about 5 different variations of the original sentence to ensure 99%
chance of us finding a desirable hash.

What's somewhat counterintuitive is that this probability is completely independent of
the number of characters we're "predicting".