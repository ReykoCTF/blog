---
title: "LACTF : Crypto Civilization | Crypto"
date: 2025-03-31
draft: false
categories: ["Crypto"]
ctf: "LACTF 2025"
challenge: "Crypto-Civilization"
author: "Reyko"
---

Solve.py :

```
import hashlib
from pwn import *
import binascii
import random

def PRG(s):
    h = hashlib.new('sha3_256')
    h.update(s)
    return h.digest()[:4]

prg_map = {}
prg_outputs = []

for s_int in range(0x10000):
    s = s_int.to_bytes(2, 'big')
    output = PRG(s)
    if output not in prg_map:
        prg_map[output] = s
        prg_outputs.append(output)

def xor_bytes(a, b):
    return bytes([x ^ y for x, y in zip(a, b)])

conn = remote('chall.lac.tf', 31173)

conn.recvuntil(b'Can you level up to a Crypto Pro?')

for _ in range(200):
    conn.recvuntil(b'Crypto test #')
    
    conn.recvuntil(b"Here's y: ")
    y_hex = conn.recvline().strip().decode()
    y = binascii.unhexlify(y_hex)
    
    found = False
    com = None
    decom0 = None
    decom1 = None
    
    for a in prg_outputs:
        b = xor_bytes(a, y)
        if b in prg_map:
            com = a
            decom0 = prg_map[a]
            decom1 = prg_map[b]
            found = True
            break
    
    if not found:
        com = random.choice(prg_outputs)
        decom0 = prg_map[com]
        decom1 = None
    
    conn.sendlineafter(b'> ', com.hex().encode())
    
    line = conn.recvline().decode().strip()
    if 'chicken' in line:
        choice = 0
    elif 'beef' in line:
        choice = 1
    else:
        print("Unexpected line:", line)
        conn.close()
        exit(1)
    
    if choice == 0:
        decom = decom0
    else:
        if found:
            decom = decom1
        else:
            decom = os.urandom(2)
    
    conn.sendlineafter(b'> ', decom.hex().encode())

conn.interactive()
```
