---
title: "Again and Again Ect"
date: 2024-01-21T21:35:34+01:00
tags: ["CTF","Writeups","HACKDAY 2024 Qualification","Crypto"]
author: "Sirius14"
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
description: "HACKDAY 2024 Qualification Crypto challenge by Chelinka"
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

We've intercepted an outgoing communication from the Syndicate's backup server. It looks like a member has made himself an emergency door so he can chat with someone inside... But what did he want to say? Decrypt the conversation using the fact that they are talking about `Hack107` and one of their team member...

## First look

To begin this challenge we have 4 files in our disposition, but the all are encrypted. No hint, no other information to guide us. Here is the content :

{{<highlight txt>}}
File 1: YiEjKycKVGgHX3UFFxVCKAsPVBYIPBEdF3xtKDgEXWMGJAEXLC5ZQFg4bl82RjNnLwlDUQwoHSdabzxCXy00ZE0oFlg5ODpZV0cUARcpNC9UNVs9XDRPAA4cFBAANllSMiRCCFxEDxZnWDIrN2wVPWwpCCIhETknTkYONjhGTgU+HwcXZgICEl1VFE0VKCFeSTk5EVJMTBsPVxBfA0slPCBQEwAHPxZ5PD9HOj8BMWEKJSFHKzhZH2sJOBAncUdtZFYgMiUBRX5/YhscABUEJyoZMVwjbTQCLlNfM0UtLHE4LicUJgBEIi10bg==

File 2: FWlvACkEWnlDB3UaFxMWKBcLVBEFK0cZBjMoByJQW19PMwIgYzFQGB0fYD0=

File 3: ZSgqKTsCETsWXjFKDwJCOBYZBkUMKgMKACM+RXYEXFQBZxMqNjAVGhcaK1l1FjcpMkZ/VgglSyZbaDwKUmwiIUozV1h5bRFQFFhXXlVrKD8AIF48VXMbHANZVQQCKhkGRS4LREQMCFM0XCYiPGJr

File 4: ZSE9Li0ERWlTfSxKDAIVYRgIEBcIPRRYDCNtHzccUR8cJAsjIjZcXkoxKVo4XzppNQlbUQUnD2JZMWgWWCc1KhkuClgcDBp6M3I/FSYEBwQ7AGgHYRYwJyc0cTgqCi4tNQ02O3kwPiwJdgQYAQ0nFzFqSmwPGSghXwJRdzwTQx4iVhJSMhgCU1BaDggCM3QfB1EsD1JMRhEMEhkefQ==
{{</highlight>}}

All these file are encrypted then encoded in base64. We have a plaintext : `Hack107` which could be useful 

## Xor Xor Xor Xor

Let's start with this hypothesis (you understand why later) : All these file are encrypted with the same key.
Majority of stream cipher or block cipher use xor operation between a stream (depend on key) and plaintext a give the ciphertext. Now what impact does the key have ? Why it's so important to change key between 2 files ?

Consider p1,p2,c1,c2 and k respectively : plaintext_1,plaintext_2,ciphertext_1,ciphertext_2 and key
If we use the same key to encrypt p1 and p2 we have :
- `c1 = p1^k`
- `c2 = p2^k`

But if we rearrange the terms :

- `c1^c2 = p1^k^p2^k = p1^p2`
- `p2 = c1^c2^p1`

Thus it is possible to retreive plaintext_2 with a known plaintext_1 !!!
This is our case we know that `Hack107` is at least in one of these 4 files.

Let's code a python script to test if our hypothesis is the good one:
{{<highlight txt>}}
import base64

def byte_xor(ba1, ba2):
    return bytes([_a ^ _b for _a, _b in zip(ba1, ba2)])

conv1 = base64.b64decode("YiEjKycKVGgHX3UFFxVCKAsPVBYIPBEdF3xtKDgEXWMGJAEXLC5ZQFg4bl82RjNnLwlDUQwoHSdabzxCXy00ZE0oFlg5ODpZV0cUARcpNC9UNVs9XDRPAA4cFBAANllSMiRCCFxEDxZnWDIrN2wVPWwpCCIhETknTkYONjhGTgU+HwcXZgICEl1VFE0VKCFeSTk5EVJMTBsPVxBfA0slPCBQEwAHPxZ5PD9HOj8BMWEKJSFHKzhZH2sJOBAncUdtZFYgMiUBRX5/YhscABUEJyoZMVwjbTQCLlNfM0UtLHE4LicUJgBEIi10bg==")
conv2 = base64.b64decode("FWlvACkEWnlDB3UaFxMWKBcLVBEFK0cZBjMoByJQW19PMwIgYzFQGB0fYD0=")
conv3 = base64.b64decode("ZSgqKTsCETsWXjFKDwJCOBYZBkUMKgMKACM+RXYEXFQBZxMqNjAVGhcaK1l1FjcpMkZ/VgglSyZbaDwKUmwiIUozV1h5bRFQFFhXXlVrKD8AIF48VXMbHANZVQQCKhkGRS4LREQMCFM0XCYiPGJr")
conv4 = base64.b64decode("ZSE9Li0ERWlTfSxKDAIVYRgIEBcIPRRYDCNtHzccUR8cJAsjIjZcXkoxKVo4XzppNQlbUQUnD2JZMWgWWCc1KhkuClgcDBp6M3I/FSYEBwQ7AGgHYRYwJyc0cTgqCi4tNQ02O3kwPiwJdgQYAQ0nFzFqSmwPGSghXwJRdzwTQx4iVhJSMhgCU1BaDggCM3QfB1EsD1JMRhEMEhkefQ==")

plaintext = b"Hack107"

for i in range(len(conv2)):
    tmp = byte_xor(conv1,conv2)
    res = byte_xor(tmp,i*b"\x00"+plaintext) # test every position
    print(res)
{{</highlight>}}

In this script I only test conv1 and conv2, when i=3 we have: `wHLcome to`
So plaintext_1 start with Welcome to and we can guess like that the beginning of the other plaintext.

## Guessing part

After few plaintext I was stuck here:
- p1 : `Welcome to our irc server, AntiRickRoll`
- p2 : ` - Hack107 putting the accent on the se`
- p3 : `Please send me your address, then your `
- p4 : `Perfect! My new address is vale.scafati`

Therefore I need to guess the most suitable word to use. I have just brutforced p2 with all possibilities of 5,6 or 7 letters (7 was the max otherwise we overflow size of p2), and I found the correct word : `seven`. Yeah it's quite logic !
Then I continue and finish to decrypt enough part to obtain the flag, it was easier to do because p2 is reuse in p3: 
- p1 : `Welcome to our irc server, AntiRickRoll. I hope you haven't had too much trouble along the way. We'll be able to ...`
- p2 : ` - Hack107 putting the accent on the seven.\n`
- p3 : `Please send me your address, then your token, and I'll do the rest. - Hack107 putting the accent on the seven.\n`
- p4 : `Perfect! My new address is vale.scafati02@gmail.com and my token is HACKDAY{DO_NOT_USE_SAME_KEY_PLS_ITS_NOT_SAFE}`

Flag : __HACKDAY{DO_NOT_USE_SAME_KEY_PLS_ITS_NOT_SAFE}__