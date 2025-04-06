---
title: "LACTF : Extremely Convenient Breaker | Crypto"
date: 2025-03-31
draft: false
categories: ["Crypto"]
ctf: "LACTF 2025"
challenge: "extremely-convenient-breaker"
author: "Reyko"
---


Solve.py : 

```
from pwn import *
import ast

conn = remote('chall.lac.tf', 31180)

conn.recvuntil("Here's the encrypted flag in hex: \n")
flag_enc_hex = conn.recvline().strip().decode()
flag_enc = bytes.fromhex(flag_enc_hex)

blocks = [flag_enc[i:i+16] for i in range(0, len(flag_enc), 16)]

def submit_ciphertext(ciphertext):
    conn.recvuntil("Enter as hex: ")
    conn.sendline(ciphertext.hex())
    response = conn.recvline().strip()
    if b"Uh something went wrong" in response:
        return None
    # Convertir la réponse (ex: b'...') en bytes réels
    try:
        decoded = ast.literal_eval(response.decode()) # Extrait les bytes de la chaîne
        return decoded
    except:
        return None

plaintext_blocks = []
for i in range(len(blocks)):
    # Soumettre un texte chiffré avec le bloc i répété
    ciphertext = blocks[i] * 4
    decrypted = submit_ciphertext(ciphertext)
    if decrypted:
        # Extraire le premier bloc de texte en clair
        plaintext_block = decrypted[:16]
        plaintext_blocks.append(plaintext_block)
        print(f"Bloc {i} déchiffré : {plaintext_block}")

flag = b''.join(plaintext_blocks)

def unpad_pkcs7(data):
    padding_length = data[-1]
    return data[:-padding_length]

try:
    flag = unpad_pkcs7(flag).decode()
except:
    flag = flag.decode(errors='ignore') 

print("Flag:", flag)

conn.close()
```

Repo des challenges : https://github.com/uclaacm/lactf-archive/tree/main/2025
