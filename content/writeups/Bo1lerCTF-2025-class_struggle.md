---
title: "B01lersCTF 2025 : Class_struggle | Reverse" 
date: 2025-04-22
draft: false
categories: ["Reverse"]
ctf: "B01lersCTF 2025"
challenge: "Class_struggle"
author: "Reyko"
---

## Challenge : class-struggle

### Enoncé : I miss the good old days before OOP, when we lived in a classless, stateless society...

Fichier fourni : marx.c

### 1. Analyse statique

Le code source est une réécriture obfusquée du Manifeste du Parti Communiste, chaque mot-clé étant redéfini via des macros pour correspondre à du C valide.

Le flag est vérifié caractère par caractère dans une boucle, comparant un tableau codé en dur `gnmupmhiaosg` avec des transformations sur l'entrée utilisateur.

### 2. Fonction de vérification

Pour chaque caractère `c` à l'index `i` du flag :

    c ^= i * 37;
    c = rol(c, (i+3) % 7);
    c += 42;
    z = (c & 0xF0) | (~c & 0x0F);
    e = ror(z, i % 8);
    if (e != gnmupmhiaosg[i]) return false;

Le flag est accepté si tous les caractères passent cette suite d'opérations.

### 3. Inversion des opérations

On reconstruit le flag en inversant les opérations :

- x = rol(gnm[i], i % 8)
- z = (x & 0xF0) | (~x & 0x0F)
- c3 = (z - 42) & 0xFF
- c2 = ror(c3, (i+3) % 7)
- c1 = c2 ^ (i * 37 & 0xFF)

### 4. Script de résolution

```python
def rol(x, r):
    return ((x << r) & 0xFF) | (x >> (8 - r))

def ror(x, r):
    return (x >> r) | ((x << (8 - r)) & 0xFF)

gnm = [0x32,0xc0,0xbf,0x6c,0x61,0x85,0x5c,0xe4,0x40,0xd0,0x8f,0xa2,
       0xef,0x7c,0x4a,0x02,0x04,0x9f,0x37,0x18,0x68,0x97,0x39,0x33,
       0xbe,0xf1,0x20,0xf1,0x40,0x83,0x06,0x7e,0xf1,0x46,0xa6,0x47,
       0xfe,0xc3,0xc8,0x67,0x04,0x4d,0xba,0x10,0x9b,0x33]

flag = ""
for i, T in enumerate(gnm):
    x = rol(T, i % 8)
    z = (x & 0xF0) | ((~x) & 0x0F)
    c3 = (z - 42) & 0xFF
    c2 = ror(c3, (i + 3) % 7)
    c1 = c2 ^ ((i * 37) & 0xFF)
    flag += chr(c1)

print(flag)
```

### 5. Flag

```
bctf{seizing_the_m3m3s_0f_pr0ducti0n_32187ea8}
```


