---
title: "Kashi CTF 2025 : Corruption | Forensics"
date: 2025-03-25
draft: false
categories: ["Forensics"]
ctf: "Kashi CTF 2025"
challenge: "Corruption"
author: "Reyko"
---

Photorec on the file

Then :

```
find . -type f -exec strings {} \; | grep -E '.*Kashi.*'
```
