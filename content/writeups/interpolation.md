---
title: "🔒 Interpolation"
date: 2024-10-27T13:58:16+01:00
author: "Sirius14"
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
description: "HeroCTF 2024 Crypto challenge by Alol"
summary:  "HeroCTF 2024 Crypto challenge by Alol"
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

Has missing data really ever stopped anyone?

# First look

The script is concise, it uses secret shamir sharing but each coefficient is a part of the secret.
{{<highlight txt>}}
#!/usr/bin/sage
import hashlib
import re

with open("flag.txt", "rb") as f:
 FLAG = f.read()
 assert re.match(rb"Hero{[0-9a-zA-Z_]{90}}", FLAG)

F = FiniteField(2**256 - 189)
R = PolynomialRing(F, "x")
H = lambda n: int(hashlib.sha256(n).hexdigest(), 16)
C = lambda x: [H(x[i : i + 4]) for i in range(0, len(FLAG), 4)]

f = R(C(FLAG))

points = []
for _ in range(f.degree()):
 r = F.random_element()
 points.append([r, f(r)])
print(points)

flag = input(">").encode().ljust(len(FLAG))

g = R(C(flag))

for p in points:
 if g(p[0]) != p[1]:
 print("Wrong flag!")
 break
else:
 print("Congrats!")
{{</highlight>}}

The goal is to retrieve the correct coefficients, then we will need to brute force a pair of 4 bytes in all the charsets to compare their hash value with coefficients.

# Retreive coefficients

With only one try I didn't have enough shares, so as the flag didn't change I merged the two shares, now using sage it's quite easy to retrieve coefficients values. To do that we can use [lagrange interpolation](https://en.wikipedia.org/wiki/Lagrange_polynomial)

{{<highlight txt>}}
F = FiniteField(2**256 - 189)
R.<x> = PolynomialRing(F, "x")

# Initialize the Lagrange polynomial
P = R(0)

# Lagrange interpolation
for i, (xi, yi) in enumerate(points):
 Li = R(1)
 for j, (xj, yj) in enumerate(points):
 if i != j:
 Li *= (x - xj) / (xi - xj)
 P += yi * Li

print(P.coefficients(sparse=False))
{{</highlight>}}

Here is the result : 

{{<highlight txt>}}
[51862623363251592162508517414206794722184767070638202339849823866691337237984,
 37382279584575671665412736907293996338695993273870192478675632069138612724862,
 54922548012150305957596790093591596584466927559339793497872781061995644787934,
 78252810134582863205690878209501272813895928209727562041762503202357420752872,
 42828444749577646348433379946210116268681295505955485156998041972023283883825,
 16605552275238206773988750913306730384585706182539455749829662274657349564685,
 10009681240064642703458239750230614173777134131788316383198404412696086812123,
 78645989056858155953548111309497253790838184388240819797824701948971210482613,
 4244268215373067710299345981438357655695365045434952475766578691548900068884,
 4587316730151077745530345853110346550953429707066041958662730783235705675823,
 98676420105970876355731743378079563095438931888109560800924537433679751968410,
 15596341609452054024790211046165535925702287406391095849367220616094959319247,
 32403908412257070302225532346590438994349383666861558172214850130936584778364,
 115533839068795212658451397535765278473898133068309149603041276877934373391258,
 7092396080272228853132842491037895182885372693653833621714864119915575351959,
 66681440692524165569992671994842901187406728987456386756946647843877275534778,
 43594818259201189283635356607462328520192502107771693650896092861477784342431,
 91842050171741174464568525719602040646922469791657773826919079592778110767648,
 105484582062398143020926667398250530293520625898492636870365251172877956081489,
 48478433129988933656911497337570454952912987663301800112434018755270886790086,
 9286536496641678624961072298289256247776902880262474453231051084428770229931,
 71177914266346294875020009514904614231152252028035180341047573071890295627281,
 58688474918974956495962699109478986243962548972465028067725936901754910032197,
 91356407137791927144958613770622174607926961061379368852376771002781151613901]
{{</highlight>}}

To be sure that it's the correct coefficient we can try with known part of the flag `Hero`, his sha256 value is `72a9345fb29494a4e9667d7bd68f37e8fe1df384270553f17b4c4a06b1f2b5e0` by converting into decimal we obtain `51862623363251592162508517414206794722184767070638202339849823866691337237984` that is the first value in our coefficient array.

# Let's break these hashes

With a simple Python script, we can brute-force all possibilities and find the flag. Using a dictionary to store the index makes it easier to put the flags back in the right order. After a few minutes, we finally found the flag!!!

```py
import hashlib

dico = {}

for i in alphabet:
 for j in alphabet:
 for k in alphabet:
 for l in alphabet:
 if int(hashlib.sha256((i+j+k+l).encode('utf-8')).hexdigest(), 16) in solutionTab:
 dico[solutionTab.index(int(hashlib.sha256((i+j+k+l).encode('utf-8')).hexdigest(), 16))] = i+j+k+l

dico = dict(sorted(dico.items(), key=lambda item: item[0]))

result = ""
for key, value in dico.items():
 result += f"{value}"

print(result)
```

Flag was __Hero{th3r3_4r3_tw0_typ35_0f_p30pl3_1n_th15_w0rld_th053_wh0_c4n_3xtr4p0l4t3_fr0m_1nc0mpl3t3_d474}__