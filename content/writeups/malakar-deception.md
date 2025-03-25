---
title: "🧠 Malakar's Deception"
date: 2025-03-25T11:42:16+01:00
author: "Sirius14"
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
description: CyberApocalypse HTB 2025 Machine Learning challenge
summary:  CyberApocalypse HTB 2025 Machine Learning challenge
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

You recently recovered a mysterious magical artifact (malicious.h5) from Malakar's abandoned sanctum. Upon activation, the artifact began displaying unusual behaviors, suggesting hidden enchantments. As Eldoria’s expert mage in digital enchantments, it falls to you to carefully examine this artifact and reveal its secrets.

# First look

We have a malicious model to analyze, and this time it's a TensorFlow model. The file `malicious.h5` contains the entire model.

After attempting to load the model unsuccessfully, I decided to dig deeper and explore whether it was possible to analyze the structure of an H5 file without actually loading it.

# Model Analysis

This [blog post](https://stevejeffersonr.medium.com/fixing-model-compatibility-errors-across-tensorflow-versions-6a2114d85ba) provided a Python script to extract structural information from a model file. 

```py
#!/usr/bin/python3
import h5py

def print_structure(weight_file_path):
    """
    Prints out the structure of HDF5 file.
    Args:
      weight_file_path (str) : Path to the file to analyze
    """
    f = h5py.File(weight_file_path)
    try:
        if len(f.attrs.items()):
            print("{} contains: ".format(weight_file_path))
            print("Root attributes:")
        for key, value in f.attrs.items():
            print("  {}: {}".format(key, value))

        if len(f.items())==0:
            return 

        for layer, g in f.items():
            print("  {}".format(layer))
            print("    Attributes:")
            for key, value in g.attrs.items():
                print("      {}: {}".format(key, value))

            print("    Dataset:")
            for p_name in g.keys():
                param = g[p_name]
                subkeys = param.keys()
                for k_name in param.keys():
                    print("      {}/{}: {}".format(p_name, k_name, param.get(k_name)[:]))
    finally:
        f.close()
 
try:
    print_structure('malicious.h5')
except:
    pass
```

Executing this script on our model revealed an interesting section:

![Output](/Sirius14_Blog/img/writeups/malakar-deception_1.png)

Clearly, there is a backdoor !

# Find backdoor code

The suspicious section appears to be Base64-encoded, but decoding it directly results in unreadable data. There must be an additional layer of obfuscation. Searching for `malicious h5 file analysis` led me to this [GitHub](https://github.com/Azure/counterfit/wiki/Abusing-ML-model-file-formats-to-create-malware-on-AI-systems:-A-proof-of-concept) repository, which explains how malicious code can be hidden in an H5 model. It also provided a Python script to extract and decompile the original backdoor bytecode:

```py
import base64
import marshal
import dis

compressed_code = b'4wEAAAAAAAAAAAAAAAQAAAADAAAA8zYAAACXAGcAZAGiAXQBAAAAAAAAAAAAAGQCpgEAAKsBAAAA\nAAAAAAB8AGYDZAMZAAAAAAAAAAAAUwApBE4pGulIAAAA6VQAAADpQgAAAOl7AAAA6WsAAADpMwAA\nAOlyAAAA6TQAAADpUwAAAOlfAAAA6UwAAAByCQAAAOl5AAAAcgcAAAByCAAAAHILAAAA6TEAAADp\nbgAAAOlqAAAAcgcAAADpYwAAAOl0AAAAcg4AAADpMAAAAHIPAAAA6X0AAAD6JnByaW50KCdZb3Vy\nIG1vZGVsIGhhcyBiZWVuIGhpamFja2VkIScp6f////8pAdoEZXZhbCkB2gF4cwEAAAAg+h88aXB5\ndGhvbi1pbnB1dC02OS0zMjhhYjc5ODJiNGY++gg8bGFtYmRhPnIaAAAADgAAAHM0AAAAgADwAgEJ\nSAHwAAEJSAHwAAEJSAHlCAzQDTXRCDbUCDbYCAnwCQUPBvAKAAcJ9AsFDwqAAPMAAAAA\n'

dis.dis(marshal.loads(base64.b64decode(compressed_code)))
```

Since `marshal` has compatibility issues with some Python environments, I had to use an online Python interpreter to execute the script. The output revealed the following decompiled bytecode:

```
BUILD_LIST               0
LOAD_CONST               1 ((72, 84, 66, 123, 107, 51, 114, 52, 83, 95, 76, 52, 121, 51, 114, 95, 49, 110, 106, 51, 99, 116, 49, 48, 110, 125))
LIST_EXTEND              1
LOAD_GLOBAL              1 (NULL + eval)
CACHE
LOAD_CONST               2 ("print('Your model has been hijacked!')")
```

The flag appears to be stored as ASCII values within a tuple. To retrieve it, we simply convert these values back into characters:

```py
print(''.join([chr(v) for v in [72, 84, 66, 123, 107, 51, 114, 52, 83, 95, 76, 52, 121, 51, 114, 95, 49, 110, 106, 51, 99, 116, 49, 48, 110, 125]]))
```

Flag was __HTB{k3r4S_L4y3r_1nj3ct10n}__