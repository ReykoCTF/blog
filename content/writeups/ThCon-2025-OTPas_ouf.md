---
title: "ThCon 2k25 : OTPas_ouf | Crypto"
date: 2025-04-13
draft: false
categories: ["Crypto"]
ctf: "ThCon 2k25"
challenge: "OTPas_ouf"
author: "Reyko"
---

## WU - OTPas_ouf - Crypto - ThCon 2k25

### Contexte :

#### Nom :
OTPas_ouf

#### Enoncé :
It seems like Zypherion and Mammoth, two XSS gang members neither known for their cleverness nor prudence just exchanged a message for a "vewy top secret much important" occasion.

Knowing the IQ level this may be of much interest or not, but we cannot afford to take any chances so we need you to help us decrypt this message.


#### Ressources :

- OTP_encryption.py :

```
from random import randint

def generate_OTP():
    OTP = b''
    for _ in range(10):
        OTP += int.to_bytes(randint(0,255))
    return OTP

def encrypt_file(input_file: str, output_file: str, passwd: bytes):
    with open(input_file, "rb") as ifile:
        input_data = ifile.read()

    with open(output_file, 'wb') as ofile:
        buffer = bytes([(input_data[k] ^ passwd[k % len(passwd)]) for k in range(len(input_data))])
        ofile.write(buffer)

if __name__ == '__main__':
    otp = generate_OTP()
    input_file = input("Entrer le nom du fichier à chiffrer: ").lower()
    output_file = input_file + '.encrypted'
    encrypt_file(input_file, output_file, otp)
    print(f"Le fichier {input_file} a été chiffré avec succès.")
```

- picture.jpg.encrypted :

L'image qu'il faut donc récupérer.


### Solve :

solve.py :

```
#!/usr/bin/env python3
import sys

def decrypt_file(input_file: str, output_file: str, key: bytes):
    with open(input_file, 'rb') as f:
        encrypted_data = f.read()
    # Déchiffrement par XOR en répétant la clé
    decrypted_data = bytes([encrypted_data[i] ^ key[i % len(key)] for i in range(len(encrypted_data))])
    with open(output_file, 'wb') as f:
        f.write(decrypted_data)

def main():
    input_file = "picture.jpg.encrypted"
    output_file = "picture.jpg"

    # Header JFIF standard (10 octets)
    known_header = bytes([0xFF, 0xD8, 0xFF, 0xE0, 0x00, 0x10, 0x4A, 0x46, 0x49, 0x46])

    # Lecture des 10 premiers octets du fichier chiffré pour récupérer la clé
    with open(input_file, 'rb') as f:
        encrypted_header = f.read(10)

    # Récupération de la clé par XOR
    otp_key = bytes([encrypted_header[i] ^ known_header[i] for i in range(10)])
    print("Clé OTP récupérée :", otp_key.hex())

    # Déchiffrer le fichier complet
    decrypt_file(input_file, output_file, otp_key)
    print("Fichier déchiffré enregistré sous :", output_file)

if __name__ == '__main__':
    main()

```
