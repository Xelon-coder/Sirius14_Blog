---
title: "🛠️ Speak to me"
date: 2025-04-27T15:59:00+01:00
tags: ["CTF","Writeups","FCSC 2025","🛠️ Hardware"]
author: "Sirius14"
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
description: "FCSC 2025 Hardware challenge"
summary:  "FCSC 2025 Hardware challenge"
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

Nous avons trouvé un microcontrôleur qui parle un protocole binaire inconnu sur une liaison série, pouvez-vous en trouver les secrets ? `nc chall.fcsc.fr 2303`

Difficulty: ⭐⭐

# First look

We need to communicate with an unknown protocol.
By using a simple Python socket script, it's possible to capture and analyze the first bytes of the communication.

```py
import socket

HOST = 'chall.fcsc.fr'
PORT = 2303

sock = socket.socket()
sock.connect((HOST,PORT))

print(sock.recv(2048))
print(sock.recv(2048).hex())
```

First, we receive a sentence telling us to wait:`Initializing chip, please wait...`

Then, a sequence of 8 bytes is sent:
`0701fc5cab91c4ee`

# Repeat

As we don't know exactly what to send, the easiest thing to do is to resend the last packet we received:
`0701fc5cab91c4ee`

We got the response:
`\x1e\x03invalid command, got 0x37.\xa2\xf7\xda`

Next, we test some variations, as something else could be a valid command. Let’s try replacing the `07` with `01` in the previous payload. By sending `0101fc5cab91c4ee`, we get:
`+\x03invalid length, got 0x01. rebooting...\x87u\xc7\x1c`.

From this, we can deduce that the first byte determines the length of the payload that follows it. This makes sense — in `0701fc5cab91c4ee`, we have 8 bytes in total, so the `07` means there are 7 bytes after it.
Now, let’s try modifying the second byte to see what happens.

By sending `0702fc5cab91c4ee`, we receive:
`)\x03invalid CRC32 for message 0x0702fc5c\x80/\x18\xb6`.

This suggests that there’s a CRC32 at the end of the message, likely used to check for transmission errors.
To figure out the CRC32 type, let's use a valid CRC32 — the one from the first message we received: `ab91c4ee`, which is the CRC32 of `0701fc5c`.

Using a [website](https://crccalc.com/?crc=0701fc5c&method=CRC-32&datatype=hex&outtype=hex), we find that it corresponds to CRC32/MPEG-2.

![CRC32](/Sirius14_Blog/img/writeups/speak_to_me_1.png)

# Script !

Now that we know the protocol structure, we can summarize the key points:
- The first byte declares the byte-size of the message.
- The last 4 bytes are the CRC32 checksum of the previous 4 bytes.
- The remaining bytes are related to the command being sent.

With this information in hand, let's create a Python script to test different values for the second byte, as we now have a correct CRC32 formula.

```py
import socket
import zlib

sock = socket.socket()
sock.connect(('chall.fcsc.fr', 2303))

print(sock.recv(1024))
res = (sock.recv(1024)) 
print(res.hex())

def crc32mpeg2(buf, crc=0xffffffff):
    for val in buf:
        crc ^= val << 24
        for _ in range(8):
            crc = crc << 1 if (crc & 0x80000000) == 0 else (crc << 1) ^ 0x104c11db7
    return crc

for i in range(0xff):

    msg = b'\x07' + bytes.fromhex(str(hex(i)[2:].zfill(2))) + b'\xfc\x5c' 

    full_msg = msg + crc32mpeg2(msg).to_bytes(4, 'big')

    sock.send(bytes.fromhex(full_msg.hex()))

    print(sock.recv(1024))
```

After running the Python script and testing various second-byte values, we finally receive a response after a few seconds. And there it is — the flag!

![CRC32](/Sirius14_Blog/img/writeups/speak_to_me_2.png)

Flag was: __FCSC{e1d6c52dfe96ab07bccd8d8dcb240a91cec586b746967c084febe75f8579907c}__