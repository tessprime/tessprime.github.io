---
title: "Solving equations in discrete arithmetic"
date: 2023-05-18T10:04:15-07:00
math: true
draft: true
---
For some reason, most word problems involving dealing with
remainders primarily around pirates divvying up coconuts.
Perhaps Number Theorists feel left out by having Calculus
having all the obvious applications to maritime traditions?

Anyhow, lets roll with it.

**A pile of coconuts is being divided up by 13 pirates. Each
coconut is subdivided into 6 pieces. After each pirate gets
their share, there is one piece left over which the Captain
takes. What is the smallest number of initial coconuts?**

So we want to find a number C such that

$$6 C % 13 = 1$$

That is

$$ 6 C = 1 \mod 13 $$

Now how can we solve this? Well we could try doing trial
multiplication 
