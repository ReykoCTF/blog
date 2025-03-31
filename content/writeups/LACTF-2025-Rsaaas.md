---
title: "LACTF : Rsaaas | Crypto"
date: 2025-03-31
draft: false
categories: ["Crypto"]
ctf: "LACTF 2025"
challenge: "rsaaas"
author: "Reyko"
---

Solve.py

```
from pwn import *
from Crypto.Util.number import isPrime
import random

def generate_special_prime():
    e = 65537
    lower = 2**63
    upper = 2**64
    while True:
        k = random.randint(lower // e, (upper - 1) // e)
        candidate = e * k + 1
        if candidate < lower or candidate >= upper:
            continue
        if isPrime(candidate):
            return candidate

def generate_regular_prime():
    lower = 2**63
    upper = 2**64
    while True:
        candidate = random.randint(lower, upper)
        if isPrime(candidate):
            return candidate

p = generate_special_prime()
q = generate_regular_prime()

conn = remote('chall.lac.tf', 31176)

conn.recvuntil("Input p: ")
conn.sendline(str(p))
conn.recvuntil("Input q: ")
conn.sendline(str(q))

print(conn.recvall().decode())
```
