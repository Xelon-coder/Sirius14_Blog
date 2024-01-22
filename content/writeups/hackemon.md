---
title: "🖼️ Hackemon"
date: 2024-01-22T15:08:53+01:00
tags: ["CTF","Writeups","HACKDAY 2024 Qualification","🖼️ Stego"]
author: "Sirius14"
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
description: "HACKDAY 2024 Qualification Steganography challenge by Yam"
summary: "HACKDAY 2024 Qualification Steganography challenge by Yam"
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

An agent who has infiltrated the syndicate has sent us a message giving us information about Syndicate. We need to decrypt the message to retrieve the information. The undercover agent said in his message that he wanted to show you the last pokemon he'd captured in his game. Connect to the instance using netcat to begin the challenge.

`nc challenges.hackday.fr 50392`

## Netcat

When we connect to the netcat instance we need to specify an email address, then this message appeared :

{{<highlight txt>}}
Now I have to confirm your pokemon skills
Can you please specify, in the order you receive them, the Japanese names
of each pokemon that I sent you?
Ex: rattata --> koratta
{{</highlight>}}

Then I checked my email address and found these 5 pokemon cards
![pokemon cards](/Sirius14_Blog/img/writeups/hackemon_1.png)
With this [site](https://bulbapedia.bulbagarden.net/wiki/List_of_Japanese_Pok%C3%A9mon_names) I retreive the japanese name, and valid the first step:
- pidgey -> `poppo`
- squirtle -> `zenigame`
- bulbasaur -> `fushigidane`
- charmander -> `hitokage`
- vulpix -> `rokon`

After few seconds I receive a QR code.
![pokemon cards](/Sirius14_Blog/img/writeups/hackemon_2.png)

## Instagram

This QR code lead us to an instagram account with a unique post. In this post we have just one comment with a link :

`This one is not bad either! https://ibb.co/q0P0ThX`

Once clicked on the link, we obtain another image:
![hackday card](/Sirius14_Blog/img/writeups/hackemon_3.jpg)

I pass this image in [aperisolve](https://www.aperisolve.com/) but nothing to report.

## Pika pika

I try to test with another famous tool in steganography called Stegseek, we can bruteforce password and find a hidden file named :

`i_want_to_tell_u_smth.txt`

![pika pika file](/Sirius14_Blog/img/writeups/hackemon_4.png)

It's pika lang, an esoteric language used in steganography, with [dcode](https://www.dcode.fr/langage-pikalang) we can interpret this code and obtain:

`The flag is Hacday{reverse the name of the Pokemon below}`

So flag is : __HACKDAY{uhcakip}__