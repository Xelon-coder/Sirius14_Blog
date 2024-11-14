---
title: "🔒 Paranoia"
date: 2024-10-27T14:50:18+01:00
author: "Sirius14"
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
description: "HeroCTF 2024 Crypto challenge by Alol"
summary:  "HeroCTF 2024 Crypto challenge by Alol"
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

I always feel that somebody's watching me
And I have found a way to keep my privacy (oh, oh)

# First look

This challenge was a little bit tricky, plaintext is twice encrypted with two different symmetric algorithms.

{{<highlight txt>}}
class Paranoia:
 def __init__(self, keys):
 self.keys = keys

 def __pad(self, data: bytes, bs: int) -> bytes:
 return data + (chr(bs - len(data) % bs) * (bs - len(data) % bs)).encode()

 def __encrypt(self, algorithm, data: bytes, key: bytes):
 cipher = Cipher(algorithm(key), modes.ECB())
 encryptor = cipher.encryptor()
 return encryptor.update(data) + encryptor.finalize()

 def encrypt(self, data: bytes):
 """
 🇨🇳 encryption to protect against the 🇺🇸 backdoor and
 🇺🇸 encryption to protect against the 🇨🇳 backdoor

 I'm a genius !
 """

 data = self.__pad(data, 16)
 data = self.__encrypt(AES, data, self.keys[0])
 data = self.__encrypt(SM4, data, self.keys[1])
 return data

with open("flag.txt", "rb") as f:
 flag = f.read()

keys = [os.urandom(16) for _ in range(2)]
paranoia = Paranoia(keys)

banner = b"I don't trust governments, thankfully I've found smart a way to keep my data secure."

print("pt_banner =", banner)
print("ct_banner =", paranoia.encrypt(banner))
print("enc_flag  =", paranoia.encrypt(flag))

# To comply with cryptography export regulations,
# 6 bytes = 2**48 bits, should be bruteforce-proof anyway
for n, k in enumerate(keys):
 print(f"k{n} = {k[3:]}")
{{</highlight>}}

However, we have hints !!!
Indeed, the script gave us the last part of each key, but considering the time complexity of brute-forcing these keys, it's not reasonable unless ...

# Meet in the middle

... unless we use a specific attack name meet in the middle. The principle is to join into an intermediate value, a part encrypts the known plaintext with AES and stores the result in a correspondence table. Then a second part decrypts the ciphertext, once a correspondence is found in the table we have cracked both keys but instead of O(2*\*48) complexity we have this time O(2*\*25) that's better. Here is the Python script I use to retrieve both keys.

{{<highlight txt>}}
import itertools
from cryptography.hazmat.primitives.ciphers import Cipher, algorithms, modes

# Données et configuration
pt_banner = b"I don't trust governments, thankfully I've found smart a way to keep my data secure."
ct_banner = b"\xd5\xae\x14\x9de\x86\x15\x88\xe0\xdc\xc7\x88{\xcfy\x81\x91\xbaH\xb6\x06\x02\xbey_0\xa5\x8a\xf6\x8b?\x9c\xc9\x92\xac\xdeb=@\x9bI\xeeY\xa0\x8d/o\xfa%)\xfb\xa2j\xd9N\xf7\xfd\xf6\xc2\x0b\xc3\xd2\xfc\te\x99\x9aIG\x01_\xb3\xf4\x0fG\xfb\x9f\xab\\\xe0\xcc\x92\xf5\xaf\xa2\xe6\xb0h\x7f}\x92O\xa6\x04\x92\x88"
known_k0 = b'C\xb0\xc0f\xf3\xa8\n\xff\x8e\x96g\x03"'  # Octets 3 à 15 de k0
known_k1 = b'Q\x95\x8b@\xfbf\xba_\x9e\x84\xba\x1a7'  # Octets 3 à 15 de k1

def pad(data: bytes, block_size: int) -> bytes:
 padding_len = block_size - (len(data) % block_size)
 return data + bytes([padding_len] * padding_len)

def aes_encrypt(data, key):
 cipher_aes = Cipher(algorithms.AES(key), modes.ECB())
 encryptor_aes = cipher_aes.encryptor()
 return encryptor_aes.update(data) + encryptor_aes.finalize()

def sm4_decrypt(data, key):
 cipher_sm4 = Cipher(algorithms.SM4(key), modes.ECB())
 decryptor_sm4 = cipher_sm4.decryptor()
 return decryptor_sm4.update(data) + decryptor_sm4.finalize()

def mitm_attack():
 intermediate_dict = {}

 possible_bytes = range(256)
 padded_pt_banner = pad(pt_banner, 16)
    
 for k0_prefix in itertools.product(possible_bytes, repeat=3):
 k0_full = bytes(k0_prefix) + known_k0
 aes_output = aes_encrypt(padded_pt_banner, k0_full)
 intermediate_dict[aes_output] = k0_full

 for k1_prefix in itertools.product(possible_bytes, repeat=3):
 k1_full = bytes(k1_prefix) + known_k1
 sm4_decrypted = sm4_decrypt(ct_banner, k1_full)

 if sm4_decrypted in intermediate_dict:
 print("Keys were found !!!")
 print("k0 =", intermediate_dict[sm4_decrypted])
 print("k1 =", k1_full)
 return intermediate_dict[sm4_decrypted], k1_full

 print("No keys were found")

if __name__=="__main__":
 mitm_attack()
{{</highlight>}}

In a few minutes we obtain k0 and k1
{{<highlight txt>}}
 k0 = b'If-C\xb0\xc0f\xf3\xa8\n\xff\x8e\x96g\x03"'
 k1 = b'\x94\xcb\x92Q\x95\x8b@\xfbf\xba_\x9e\x84\xba\x1a7'
{{</highlight>}}

Let's decrypt the flag and finish this challenge.

Flag was Hero{p4r4n014_p4r4n014_3v3ryb0dy_5_c0m1n6_70_637_m3!}