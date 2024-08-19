---
title: "Golden Ticket"
date: 2024-08-18T17:18:56-07:00
tags: ["ctfs", "math"]
---

# Golden Ticket (idek 2024)
So we're given the following:

```py
from Crypto.Util.number import *

#Some magic from Willy Wonka
def chocolate_generator(m:int) -> int:
    p = 396430433566694153228963024068183195900644000015629930982017434859080008533624204265038366113052353086248115602503012179807206251960510130759852727353283868788493357310003786807
    return (pow(13, m, p) + pow(37, m, p)) % p

#The golden ticket is hiding inside chocolate
flag = b"idek{REDACTED}"
golden_ticket = bytes_to_long(flag)
flag_chocolate = chocolate_generator(golden_ticket)
chocolate_bag = []

#Willy Wonka is making chocolates
for i in range(golden_ticket):
    chocolate_bag.append(chocolate_generator(i))

#And he put the golden ticket at the end
chocolate_bag.append(flag_chocolate)

#Augustus ate lots of chocolates, but he can't eat all cuz he is full now :D
remain = chocolate_bag[-2:]

#Can you help Charles get the golden ticket?
print(remain)

#[88952575866827947965983024351948428571644045481852955585307229868427303211803239917835211249629755846575548754617810635567272526061976590304647326424871380247801316189016325247, 67077340815509559968966395605991498895734870241569147039932716484176494534953008553337442440573747593113271897771706973941604973691227887232994456813209749283078720189994152242]
```

So the `chocolate_generator` function gives us

$$
cg(x) = 13^x + 37^x \mod p
$$

And we're given the following values of it, `flag-1` and `flag`. So we have

$$
\begin{matrix}
cg(flag-1) &=& 13^{flag-1} &+& 37^{flag-1} \\\\
cg(flag) &=& 13^{flag} &+& 37^{flag} \\\\
\end{matrix}
$$

So we multiply the top by 37 and 13 to make everything have the same powers

$$
\begin{matrix}
37\times 13\times cg(flag-1) &=& 37\times 13^{flag} &+& 13\times 37^{flag} \\\\
cg(flag) &=& 13^{flag} &+& 37^{flag} \\\\
\end{matrix}
$$

And then subtract the bottom times 37 from the top to get

$$
\begin{matrix}
37\times 13\times cg(flag-1) - 37\times cg(flag) &=& -24\times 37^{flag} \\\\
\end{matrix}
$$

Now $p-1$ is the size of the multiplication group of $p$, and it turns out it is entirely composed of small
factors. So Pohlig-Helman applies here, and the discrete log can be computed quickly in $p$. So in sage we can
simply do (since it will automatically apply Pohlig-Helman when solving in $p$)

```py
p=396430433566694153228963024068183195900644000015629930982017434859080008533624204265038366113052353086248115602503012179807206251960510130759852727353283868788493357310003786807
a,b = [88952575866827947965983024351948428571644045481852955585307229868427303211803239917835211249629755846575548754617810635567272526061976590304647326424871380247801316189016325247, 67077340815509559968966395605991498895734870241569147039932716484176494534953008553337442440573747593113271897771706973941604973691227887232994456813209749283078720189994152242]
e_msg = (37*13*a-37*b) * pow(-24,-1,p)
msg = GF(p)(e_msg).log(37)
print(msg.to_bytes(128, 'big').replace(b"\x00",0))
```