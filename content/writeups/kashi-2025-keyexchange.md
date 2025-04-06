---
title: "Kashi CTF 2025 : Key Exchange | Crypto"
date: 2025-03-25
draft: false
categories: ["Crypto"]
ctf: "Kashi CTF 2025"
challenge: "Key Exchange"
author: "Reyko"
---

Solve.py
```
from pwn import *
import hashlib
from Crypto.Cipher import AES
import json
import re

Gx = 26247035095799689268623156744566981891852923491109213387815615900925518854738050089022388053975719786650872476732087
Gy = 8325710961489029985546751289520108179287853048861315594709205902480503199884419224438643760392947333078086511627871

def solve():
    conn = remote('kashictf.iitbhucybersec.in', 54203)

    pk_pattern = re.compile(r"Public Key:\s*\(?\s*(\d+)\s*,\s*(\d+)\s*\)?")

    while True:
        line = conn.recvline(timeout=2).decode()
        if "And my Public Key" in line:
            match = pk_pattern.search(line)
            if match:
                PAx, PAy = map(int, match.groups())
                break
        elif not line:
            raise Exception("Serveur injoignable ou format inattendu")

    conn.sendlineafter(b'x-coord: ', str(Gx).encode())
    conn.sendlineafter(b'y-coord: ', str(Gy).encode())

    encrypted_msg = conn.recvuntil(b'}').decode()  # Capture tout jusqu'au }
    data = json.loads(encrypted_msg.split('Message: ')[1])

    # Déchiffrement avec SHA1
    sha = hashlib.sha1(str(PAx).encode()).digest()[:16]
    cipher = AES.new(sha, AES.MODE_CBC, bytes.fromhex(data['iv']))
    plain = cipher.decrypt(bytes.fromhex(data['ciphertext']))

    print(plain[:-plain[-1]].decode())  # PKCS#7 unpad

if __name__ == "__main__":
    solve()
```

Output :

```
[Feb 23, 2025 - 11:42:34 (CET)] exegol-fofo /workspace # python3 ecc.py
[+] Opening connection to kashictf.iitbhucybersec.in on port 54203: Done
NaeusGRX{L_r3H3Nv3h_kq_Sun1Vm_O3w_4fg_4lx_1_t0d_a4q_lk1s_X0hcc_Dd4J_R4GlxBmP}

Hint: DamnKeys
[*] Closed connection to kashictf.iitbhucybersec.in port 54203
```

Then :

https://www.dcode.fr/vigenere-cipher of NaeusGRX{L_r3H3Nv3h_kq_Sun1Vm_O3w_4fg_4lx_1_t0d_a4q_lk1s_X0hcc_Dd4J_R4GlxBmP} with DamnKeys key


Repo des challenges : https://github.com/IIT-BHU-CyberSec/Kashi-CTF-2025
