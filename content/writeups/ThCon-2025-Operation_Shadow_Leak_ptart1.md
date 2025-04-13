---
title: "ThCon 2k25 : Operation Shadow Leak pt. 1 | Forensic"
date: 2025-04-13
draft: false
categories: ["Forensic"]
ctf: "ThCon 2k25"
challenge: "Operation Shadow Leak pt. 1"
author: "Reyko"
---

## WU - Operation Shadow Leak pt. 1  - Forensic - ThCon 2k25

### Contexte :

#### Nom :
Operation Shadow Leak pt. 1

#### Enoncé :

During their digital assault, a S.N.A.F.U agent managed to infiltrate an XSS member’s computer long enough to extract valuable RAM data and steal a critical disk image.

This intelligence may hold the key to decrypting vital documents that could expose the XSS's strategies and counter their plans for a devastating raid on the city's infrastructure.

You need to find both a flag and the bitlocker password, this one will be useful later.
https://filesender.renater.fr/?s=download&token=ce51db62-6fe7-4e94-b608-60c7671e9a4e


### Solve :

Just grep :) :

```bash
find . -type f -exec strings {} \; | grep -E '.*THC{.*
```
