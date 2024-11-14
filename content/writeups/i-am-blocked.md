---
title: "🔒 I Am Blocked"
date: 2024-01-22T16:45:17+01:00
tags: ["CTF","Writeups","HACKDAY 2024 Qualification","🔒 Crypto"]
author: "Sirius14"
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
description: "HACKDAY 2024 Qualification Crypto challenge by Chelinka"
summary:  "HACKDAY 2024 Qualification Crypto challenge by Chelinka"
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
---

Our undercover agent is finding it increasingly difficult to communicate important information and is forced to hide it more and more to avoid detection. As he uses RSA, and the message he's sending is too big the be computed efficiently, he truncates the message into blocks before encrypting it. Connect to the service in order to solve the challenge. Don't get blocked by the difficulty ^^

`nc challenges.hackday.fr 50397`

Hint: charset used is [a-zA-Z0-9 :/.,!]

## First look

Once connected to the netcat session we have a lot of encrypted message (~60), then we can encrypt a message as many times as we want. First thing I do is to deconnect et reconnect to see wether key change or not. Encrypted message were the same so key didn't change, this is a good news. Now we will understand how does this encryption system works !

According to the title of the challenge and his description, message are separated in block then encrypted with RSA, we need to find the length of one block :

![Rsa block](/Sirius14_Blog/img/writeups/i-am-blocked_1.png)

We can observe that block length is 4 and we don't have a Cipher Block Chaining otherwise value of the first and second block would not have been the same.
So we know ciphertext and we can encrypt as many times as we want, so let's bruteforce all block of 4 with the given charset.

Yeah ... but no. After one night of bruteforce, my computer was still to `a` because of the extremely long exchanges with the network. The bette idea is to bruteforce in local, but for that we need N (we presume that e=65537).

## Find N

To find N I found a really cool [post](https://crypto.stackexchange.com/questions/65965/determine-rsa-modulus-from-encryption-oracle) on crypto stack exchange, given 2 plaintexts and ciphertext we can determine N with this formula : 

`N = gcd(p1**e - c1, p2**e - c2)`

So I found N value, let's pass to bruteforce.

## Bruteforce

Here is the algorithm I use for bruteforcing with the given charset (stock.txt contains all the 60 blocks given when you starting a netcat session).
```py
from Crypto.Util.number import *

print("[+] Load data")
with open("stock.txt","r") as f:
    tab = [int(l.strip()) for l in f.readlines()]

print("[+] Load complete")

import itertools

dico = "0123456789 :/.,!AZERTYUIOPQSDFGHJKLMWXCVBNazertyuiopqsdfghjklmwxcvbn"

combinaisons = itertools.product(dico, repeat=4)

N=536950613076426531617234057820719016864884812372461607779203931894774391156149464286518790303851427993590665950537510482359554079703615556704958612013459885538309303475995282786688272835567211178672382508980142091719860622019873713116133455870656955498517944232607088709787695145372901198054604942724200875198473952572826892780330829380014417396290793849424864234151993261301067796580721893476858205689179392073731357217028542466210785479815086069159472892378763500725382987133404928657446654271731813026751333904279681681315488821515546597502488936792249208521230735340233324538461272147935990212573137222943335697678815574261088573766902435862454964295200812262687618887479923950553731411554553010066344323609225383611400929049571244993644032773461627350945848051865504206645800813304778897302190915235874453005465195837188906743441081298236577420338655696755930642279805002965300188447416588969618789324022330495074259881199243781018335520814995622192311268640634695790301798109094211634219398139914064457083097391436420126978420163072586702156092101099079798904376634638028677003312507004708955914605491814434263964222320148911221672131485813944854331939349514732354118647936495808414183919146399906625403987787906484724313810541

for combinaison in combinaisons:
    p = (''.join(combinaison))
    try:
        temp = pow(bytes_to_long(p.encode()+b''),65537,N)
    except:
        print("error")
    if temp in tab:
        print("[+] FOUND : n°",tab.index(temp)," , value : ",p)
```

I reorganise all the block with their corresponding number and we have the plaintext :

`My secret passwords are https://www.mediafire.com/file/kvdn1o5fnl6f21o/Uruguay.kdbx/file here I used the words Super Duper Password Cracker Head Hitter Uber in order to forge a strong password, but nobody can bruteforce the combination !!!`

After download keepass file, I need to bruteforce again to retreive the password with these 7 words: `Super Duper Password Cracker Head Hitter Uber`.

Then we can open keepass and Flag with : __HACKDAY{BLOCK_BY_BLOCK_THIS_IS_MINECRAFT}__