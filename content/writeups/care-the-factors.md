---
title: "Care The Factors"
date: 2024-01-21T16:00:10+01:00
tags: ["CTF","Writeups","HACKDAY 2024 Qualification","Crypto"]
author: "Sirius14"
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
description: "HACKDAY 2024 Qualification Crypto challenge by D0ppl3gang3r"
disableHLJS: false # to disable highlightjs
disableShare: false
hideSummary: false
searchHidden: true
ShowReadingTime: true
ShowBreadCrumbs: false
ShowPostNavLinks: false
ShowWordCount: false
ShowRssButtonInSectionTermList: false
UseHugoToc: true
pygmentsUseClasses: true
pygmentsCodeFences: true
cover:
    image: "/Sirius14_Blog/img/logo.png" # image path/url
    alt: "<alt text>" # alt text
    caption: "<text>" # display caption under cover
    relative: false # when using page bundles set this to true
    hidden: true # only hide on current single page
---

This challenge is about Ellipic Curve Cryptography

## First look

We have 2 files, a part of source code and output. Our goal is to find n, this issue is commonly known as Elliptic Curve Discrete Logarithm.

## Analysis

In the first one we have

{{<highlight txt>}}
def getA():
    # You have to find it yourself /:
    pass
def getB():
    # You have to find it yourself /:
    pass

# Some code ...

p = 1410235279292998784331797202421753874063265295308568058662741299116310072677
A = getA()
B = getB()
Fp = FiniteField(p)
E = EllipticCurve(Fp, [A, B])
P = E.random_element()
n = randint(2**30, 2**60)
Q = n * P
flag = enc_flag(n)
{{</highlight>}}

In the second one :
{{<highlight txt>}}
P=(406156291172024449433827761031736513098183950832214481256475543523051604042,937502472800241241676075882016117499207790111193756481427079135615174871684)
Q=(92554882076587701525654416824880284407135974444455993706448015434816328085, 1067245947645250194968549384640439378373660468218406176128671131644883921569)
flag={'iv': '4318aa195451964d2078e230494ef079', 'flag': '75ae6944d3434c9e96affd40c6137bfe23934fddcc6693bdfdd7a1d542f3464a12abc09d87dd0dc8fd860d666dd2b337'}
{{</highlight>}}

We have 2 points on the curve P and Q, but we don't have A and B coefficient. As a reminder, the equation of curve E is `y^2 = x^3 + ax + b`. In addition, the name of the challenge refers to factors, indeed we can determine p factors easily. This a great news, this indicating that Pohlig-Hellman could be feasible:

## Find A and B

Thus the first step is to retreive A and B. After some research I found a crypto stack exchange [post](https://crypto.stackexchange.com/questions/97811/find-elliptic-curve-parameters-a-and-b-given-two-points-on-the-curve) explaining how to do.

Here is sage code to get A and B:
{{<highlight txt>}}
P_x,P_y =(406156291172024449433827761031736513098183950832214481256475543523051604042,937502472800241241676075882016117499207790111193756481427079135615174871684)
Q_x,Q_y =(92554882076587701525654416824880284407135974444455993706448015434816328085, 1067245947645250194968549384640439378373660468218406176128671131644883921569)
p = 1410235279292998784331797202421753874063265295308568058662741299116310072677

A = (((P_y**2-Q_y**2)-(P_x**3-Q_x**3))*pow((P_x-Q_x),-1,p))%p
B = (P_y**2 - P_x**3 - A) % p
{{</highlight>}}

## Pohlig-Hellman

As p is factorable we try to use calcul discrete logarithm for enough factors of p so that the product of these is at least greater than 2**30 (because 2\*\*30 < n < 2\*\*60), if the product is greater than 2\*\*60 that would be even better.

Here is the sage code for pohling Hellman:
{{<highlight txt>}}
P=(406156291172024449433827761031736513098183950832214481256475543523051604042,937502472800241241676075882016117499207790111193756481427079135615174871684)
Q=(92554882076587701525654416824880284407135974444455993706448015434816328085, 1067245947645250194968549384640439378373660468218406176128671131644883921569)
F = FiniteField(p)
E = EllipticCurve(F,[A,B])
P = E.point(P)
Q = E.point(Q)

primes = [2, 3, 7, 43, 349, 409, 6199, 344222059, 58853094821, 162989330351, 284121766519, 940658041742936711869] # you can use E.order() to get it

dlogs = []
for fac in primes:
    t = int(G.order()) // int(fac)
    dlog = discrete_log(t*Q, t*P, operation='+')
    dlogs.append(dlog)
{{</highlight>}}

Then last step of this algorithm is to used Chinese Reminder Theorem
`n = crt(dlogs, primes)`

We print value of n and test if Q == n*P
It works !!!

P.S : Note that we don't need all dlogs tab (DL of last factor is too long for sage), personnally I use `n = crt(dlogs[1:-1], primes)`

## Decrypt and Flag

Last step is to decrypt the flag, key is md5(n) and iv is given, we put this in [cyberchef](https://gchq.github.io/CyberChef/#recipe=AES_Decrypt(%7B'option':'Hex','string':'08fac709549765925a1b15738de81fda'%7D,%7B'option':'Hex','string':'4318aa195451964d2078e230494ef079'%7D,'CBC','Hex','Raw',%7B'option':'Hex','string':''%7D,%7B'option':'Hex','string':''%7D)&input=NzVhZTY5NDRkMzQzNGM5ZTk2YWZmZDQwYzYxMzdiZmUyMzkzNGZkZGNjNjY5M2JkZmRkN2ExZDU0MmYzNDY0YTEyYWJjMDlkODdkZDBkYzhmZDg2MGQ2NjZkZDJiMzM3)

And we obtain the flag : __HACKDAY{W34k_EC_W1th_P0hlig_H3llm4n_4TT4cK}__

Thanks to D0ppl3gang3r for this chall !