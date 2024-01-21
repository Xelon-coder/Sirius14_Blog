---
title: "Sound Zero"
date: 2024-01-21T23:09:19+01:00
tags: ["CTF","Writeups","HACKDAY 2024 Qualification","Stego"]
author: "Sirius14"
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
description: "HACKDAY 2024 Qualification Crypto challenge by Didouad"
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

We have just received a surprising report from our field teams. While searching for a recently apprehended criminal, they found an interesting hard drive on the scene. However, after interrogation, this criminal revealed that he is part of an organization that could very well be SYNDICAT! Judging by the rather peculiar content on the hard drive, it certainly contains a hidden message. And, as you have guessed, your job is to uncover it! 

Download the music, and recover the flag hidden inside !

## First look

This a steganography challenge on audio file, when you listening the file you heard a rickroll. No god, I don't want to be rickrolled again !!!

The first thing I did was plotting the spectrogram with Sonic Visualizer, here is the result :
![spectrogram](/Sirius14_Blog/img/writeups/sound_zero_1.png)

Nothing special.

## LSB 

My second hypothesis was on Least Significant Byte technique. Usually it's current in CTF to use this method in image but in audio file it's rare.
I searched `LSB audio steganography` and found [this](https://sumit-arora.medium.com/audio-steganography-the-art-of-hiding-secrets-within-earshot-part-2-of-2-c76b1be719b3) medium article. I test the code to decrypt :
{{<highlight txt>}}
import wave
song = wave.open("challenge.wav", mode='rb')
# Convert audio to byte array
frame_bytes = bytearray(list(song.readframes(song.getnframes())))

# Extract the LSB of each byte
extracted = [frame_bytes[i] & 1 for i in range(len(frame_bytes))]
# Convert byte array back to string
string = "".join(chr(int("".join(map(str,extracted[i:i+8])),2)) for i in range(0,len(extracted),8))
# Cut off at the filler characters
decoded = string.split("###")[0]

# Print the extracted text
print("Sucessfully decoded: "+decoded)
song.close()
{{</highlight>}}

And in my terminal I obtain the flag : __HACKDAY{S0und_l1k3_a_g0Od_pl4n}__