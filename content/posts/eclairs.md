---
title: "Eclairs"
date: 2023-10-17T19:55:41-07:00
math: true
---

# Eclairs Writeup

A problem from TCP1P Ctf.

<!--more-->

We are given the following Python code:

```py
from Crypto.Util.number import getPrime, bytes_to_long
from sympy.ntheory.modular import crt
from libnum.ecc import *
import random
import time

while (p:=getPrime(256)) % 4 != 3: pass
while (q:=getPrime(256)) % 4 != 3: pass
e = 3
n = p*q
a = getPrime(256)
b = getPrime(256)
E = Curve(a, b, n)
flag = bytes_to_long(open("flag.txt", "rb").read())

def sqrt_mod(a):
    assert p % 4 == 3
    assert q % 4 == 3
    r = int(crt([p,q],[pow(a,(p+1)//4,p), pow(a,(q+1)//4,q)])[0])
    n = p*q
    if pow(r,2,n) == a % n:
        return r
    return False

def lift_x(x):
    y = sqrt_mod(x**3 + a*x + b)
    if y:
        return (x, y)
    return False


def find_coordinates(x):
    P = lift_x(x)
    if P:
        x,y = P
        return (pow(x,e,n), pow(y,e,n))
    return False

def captcha():
    while True:
        x = random.randint(1, n)
        P = lift_x(x)
        if P : break
    k = random.randint(1,n)
    print("HOLD UP!!!!")
    print("YOU ARE ABOUT TO DO SOMETHING VERY CONFIDENTIAL")
    print("WE NEED TO MAKE SURE THAT YOU ARE NOT A ROBOT")
    print(f"Calculate {k} X {P}")
    ans = input("Answer: ")
    return ans == str(E.power(P,k))
    

while True:
    print("1. Check out my cool curve")
    print("2. Get flag")
    print("3. Exit")
    choice = input(">> ")

    if choice == "1":
        print("This point is generated using the following parameter:")
        # encrypted because I don't want anyone to steal my cool curve >:(
        print(pow(a,e,n))
        print(pow(b,e,n))
        x = int(input("x: "))
        P = find_coordinates(x)
        if P:
            print(P)
        else:
            print("Not found :(")

    elif choice == "2":
        if captcha():
            print(pow(flag, e, n))
        else:
            print("GO AWAY!!!")
            exit()
    elif choice == "3":
        exit()
    else:
        print("??")
        exit()
```

The basic idea is that we are given a Elliptic Curve oracle which, when given a the x-coordinate, will compute 
$\left(x^3,y^3\right)$ where $\left(x,y\right)$ is the point on the curve. However, we don't know the curve, or even the value of $n$,
the Ring modulo $n$ that we're working with.

The first thing we can do is request a number of points. which will give us a number of $\left(x_i^3, y_i^3\right)$ points. Since we know
the original $x_i$ points, we can compute the cube of that (not modulo $n$) and subtract the modulo'd version. We know that this should
be zero mod n, so we the number we get is a multiple of $n$. We can thus do this several times and compute the gcd of them all. This should
give us with high probability the number $n$.

Next, we just need to determine $a$, and $b$. Now, we know that each point on the curve satisifies

$$y^2 = x^3 + a x + b$$

Since we're given $y^3$, we can instead cube both sides of this equation

$$(y^3)^2 = (x^3 + a x + b)^3$$

Now, we could say "Hey, we have two unknowns, and two equations, so
we should be able to resolve the values of $a$ and $b$ with two 
equations." Proceeding with this idea in sage we'd obtain:

```py
data = obtain_data()
def curve_point(x, y3, a,b )
    return (x^3 + a * x + b) ^ 3 - y3**2
P.<a,b> = PolynomialRing(Zmod(n))
p1 = curve_points(inputs[0]["x"], inputs[0]["y"], a, b)
p2 = curve_points(inputs[0]["x"], inputs[0]["y"], a, b)
I = ideal([p1,p2])
a = I.elimination_ideal(b)
a.roots()
```

But this won't work since we'd have to compute the roots in $\mathbb{Z}_n$, which is
basically just trying to solve RSA with an exponent of 9. This could potentially
be done if we knew the high bits of the values of $a$ and $b$ but we don't.

However, we don't have to just restrict ourselves to *two* points, we can get as
many as we want. Then we can say "Hey, let's rethink this, expanding the original
cubed value of y we'd get

```bc
x^9  + 3*a*x^7   + 3*a^2*x^5 + 3*b*x^6 + 
       a^3*x^3   + 6*a*b*x^4 + 3*a^2*b*x^2 +
       3*b^2*x^3 + 3*a*b^2*x + b^3
- y^6  = 0
```

Then we blur our eyes a bit (and remember that x is a known value)

```bc
3 * z1 * x^7 + 3*z2*x^5 + 3*z3*x^6  +
z4*x^3       + 6*z5*x^4 + 3*z6 *x^2 + 
3*z7*x^3     + 3*z8 * x + z9
- x^9 - y^6  = 0
```

Which is just a *linear* equation in 9 variables. So we can just obtain 9 more of these equations
and solve for the nine unkonws. Then, we just pick the ones corresponding to mulitples of $a$ and $b$.

Once this is obtained, we are asked to compute $k*P$ where $k$ is a random number, and $P$ is a point
on the curve. This is easily done in sage

```py
C = EllipticCurve(Zmod(n), [a,b])
P = C(point)
print(k*P)
```

And we spit that back (making sure that we have it in the right format). However, we aren't given
the flag at this point, rather, we are given the *cube* of the flag.

No Problem! Since all parameters are random, we simply run this three more times. This will give us

$$
\begin{matrix}
flag^3 &=& m_1 \mod N_1  \\\\
flag^3 &=& m_2 \mod N_2 \\\\
flag^3 &=& m_3 \mod N_3
\end{matrix}
$$

And apply CRT to obtain the flag.
