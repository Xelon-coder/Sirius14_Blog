---
title: "🔍 iForensics"
date: 2025-04-26T15:58:00+01:00
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

Lors d'un passage de douane, le douanier vous demande de lui remettre votre téléphone ainsi que son code de déverrouillage. Le téléphone vous est rendu quelques heures plus tard ...

Suspicieux, vous envoyez votre téléphone pour analyse au CERT-FR de l'ANSSI. Les analystes du CERT-FR effectuent une collecte sur le téléphone, composée d'un sysdiagnose et d'un backup.

# iForensics - iCrash (intro)
Il semblerait qu'un flag se soit caché à l'endroit où sont stockés les crashes sur le téléphone ...

## An easy one

For this series of challenges, we have 2 files:
- backup.tar.xz
- sysdiagnose_and_crashes.tar.xz
Both of these folders will be used for the whole series.
We're looking for our first flag, which seems to be hidden in the crashes folder, so let's take a look.

Indeed, inside `sysdiagnose_and_crashes`, there's a folder named `fcsc_intro` containing the flag:
Flag was __FCSC{7a1ca2d4f17d4e1aa8936f2e906f0be8}__

# iForensics - iDevice ⭐

Pour commencer, trouvez quelques informations d'intérêt sur le téléphone : version d'iOS et identifiant du modèle de téléphone.

Le flag est au format FCSC{<identifiant du modèle>|<numéro de build>}. Par exemple, pour un iPhone 14 Pro Max en iOS 18.4 (22E240) : FCSC{iPhone15,3|22E240}.

## First look

Now we need to find the model ID and build ID.
After searching a bit, we can confirm that information about iPhone devices, like model and build version, is stored in the `manifest.plist` file.
Luckily, the `backup` folder contains this file — but I'm not working on a Mac, so I can't read it properly.

Using the power of online tools, we can find websites that can open plist files for us:

![manifest.plist file](/Sirius14_Blog/img/writeups/iForensics_1.png)

So flag was: __FCSC{iPhone12,3|20A362}__

# iForensics - iWiFi ⭐

Pour continuer, trouvez quelques informations d'intérêt sur le téléphone : SSID et BSSID du réseau WiFi sur lequel le téléphone est connecté ainsi que le compte iCloud associé au téléphone.

Le flag est au format FCSC{<SSID>|<BSSID>|<compte iCloud>}. Par exemple, si le téléphone est connecté sur le réseau WiFi example, qui a pour BSSID 00:11:22:33:44:55 et que le compte iCloud associé est example@example.com : FCSC{example|00:11:22:33:44:55|example@example.com}.

## Network and co

To continue this case, we now need to find information about the Wi-Fi used by the victim.
After digging through some folders, we found some interesting stuff in `coreCapture/WiFi/` Using the grep command, we can find all occurrences of SSID (and BSSID as well):

```sh
grep -a -i -r 'ssid' .
```

![SSID/BSSID grep](/Sirius14_Blog/img/writeups/iForensics_2.png)

Alright, it's looking good, but we're not done yet!
To find the iCloud email, we could grep again, but there are too many iCloud emails. So let's be clever!

Looking back at the `manifest.plist` file from the previous challenge, we can see it's Robert's phone. So, logically, the email should contain "robert" or something similar.

```sh
grep -a -i -r '@icloud.com' . | grep -a 'robert'
```

![iCloud email grep](/Sirius14_Blog/img/writeups/iForensics_3.png)

Nice we have now all the information to flag this challenge:

Flag was: __FCSC{FCSC|66:20:95:6C:9B:37|robertswigert@icloud.com}__

# iForensics - iNvisible ⭐⭐

Il semblerait qu'un message n'ait pas pu s'envoyer ... Retrouvez le destinataire de ce message.

Le flag est au format FCSC{<destinataire>}. Par exemple, si le destinataire est example@example.com : FCSC{example@example.com}.

## Not so Invisible !

In order to find the email, we first need to locate the message database file. Grep isn't working anymore, so we need to dive deeper into the `backup` folder.

The first thing we notice is that many of the folder names are in hexadecimal, similar to how a .git file might store hashes. By searching for the location of this database, I found this [medium](https://medium.com/@thehackpatrol/modifying-ios-sms-database-93a3f64f8c94), which explains that this file is stored in `Library/SMS/sms.db`.However, we don't have that file. It also mentions that the hash of this file is `3d0d7e5fb2ce288813306e4d4636395e047a3d28`.

Let's dig into `3d` folder !

## Database 

Indeed, there is a file named `3d0d7e5fb2ce288813306e4d4636395e047a3d28` file. Using the file command, we can see that it's an SQLite3 database. After opening it and navigating to the chat table, we find two messages. The first one is pretty odd because the `last_read_message_timestamp` column is 0, indicating there might be an issue with this one. From this, we can identify the receive

![message db](/Sirius14_Blog/img/writeups/iForensics_4.png)

Flag was: __FCSC{kristy.friedman@outlook.com}__

# iForensics - iTreasure ⭐

Avant la remise du téléphone à la douane, le propriétaire du téléphone a eu le temps d'envoyer un trésor. Retrouvez ce trésor.

## Hidden treasure 

Robert sent a message, so let's keep digging into the `3d0d7e5fb2ce288813306e4d4636395e047a3d28` file.
In the `attachment` table, we find some interesting information.

![message db](/Sirius14_Blog/img/writeups/iForensics_5.png)

By downloading `user_info`, we can obtain more details about this file, including some iCloud content links, but they don't work.
The only information we have is the file type — it's an HEIC image.

## Holy grep !

A quick search on Google reveals the magic bytes for the HEIC file type, and now we're ready to retrieve all HEIC images from the `backup` folder!

```sh
grep -a -i -r 'ftypheic' 
```

We have 3 results, let's take the first one `6f4e34098e00a80fde876c8638fb1d685be2318b`, adding .heic extension and display it.

![flag image](/Sirius14_Blog/img/writeups/iForensics_6.png)

Flag was: __FCSC{511773550dca}__

# iForensics - iBackdoor 1/2 ⭐⭐

Vous continuez vos analyses afin de trouver la backdoor sur le téléphone. Vous finissez par vous rendre compte qu'une application est compromise et que le téléphone était infecté au moment de la collecte ... Trouvez l'identifiant de l'application compromise ainsi que l'identifiant de processus (PID) du malware.

Le flag est au format FCSC{<identifiant application>|<PID>}. Par exemple, si l'application compromise est Example (com.example) et que le PID est 1337 : FCSC{com.example|1337}.

## Signal signal !

For now, we haven’t focused on the .ips files, which are crash reports. However, we notice some weirdly named ones, like `Signal.hooked-*.ips`.
It looks like there's a malware using Signal to launch itself discreetly, so the compromised application seems to be `org.whispersystems.signal` (the coalition name for the Signal app).

But how do we find the PID? There are indeed several `Signal.hooked-*.ips` files with different PIDs. Let’s dig deeper!

## Back to the beginning

After searching through multiple folders, we find the `spindump-nosymbols.txt` file, which is a snapshot of system processes and thread activity without debug symbols — useful for analyzing performance issues or crashes.
By using grep again (on Signal.Hooked), it becomes easier to understand how the malware is affecting Signal.

![spindump-nosymbols.txt](/Sirius14_Blog/img/writeups/iForensics_7.png)

This process has for parent Signal.hooked, and have for PID 345.

Flag was: __FCSC{org.whispersystems.signal|345}__

# iForensics - iC2 ⭐⭐⭐

Retrouvez le nom de l'outil malveillant déployé sur le téléphone, ainsi que le protocole, l'adresse IP et le port de communication vers le serveur C2.

## Mussel

In the previous challenge, we discovered that the malware process hides itself within the Signal app, and the name of this process is `mussel`.

To gather more information about this process, let's check the `ps.txt` file:

```
root                 0    99   345   344  4004004   0.0  0.0   0  0        0      0 -        ??  ?     7:56AM   0:00.00 /var/containers/Bundle/Application/4B6E715E-641B-4F43-B39B-CA9AE3E8B73B/Signal.app/mussel dGNwOi8vOTguNjYuMTU0LjIzNToyOTU1Mg==
```

There is like a base64 on the launch command, let's decode it: `tcp://98.66.154.235:29552`.
Nice we found the C2 !!!

Unfortunately, I didn't have time to identify the malware tool used.

## Acknowledgment 

Big thanks to the FCSC team for this series of challenge on ios. It was very a very cool forensic series !