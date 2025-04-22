---
title: "B01lersCTF 2025 : What | Reverse"
date: 2025-04-22
draft: false
categories: ["Reverse"]
ctf: "B01lersCTF 2025"
challenge: "What"
author: "Reyko"
---

## Challenge : what

### Enoncé :

> im so confused

### 1. Analyse du binaire

```bash
file what
# ELF 64-bit PIE, x86-64, not stripped, with debug_info
```

Le programme affiche "What did you say?" puis lit une chaîne caractère. La fonction main contient une boucle analysant une chaîne de contrôle contenant uniquement les caractères : `?`, `W`, `H`, `A`, `T`, `!`.

Chaque bloc commence par `?` (input utilisateur) suivi de plusieurs opérations (`W`, `H`, `A`), se termine par `T` (vérification), puis éventuellement `!` pour afficher le verdict final.

### 2. Fonctionnement

| Code | Action                                                                 |
|------|------------------------------------------------------------------------|
| `?`  | Lire un caractère utilisateur et l'affecter à `in`                   |
| `W`  | `in ^= what[what_idx]`                                                |
| `H`  | `in += what[what_idx]`                                                |
| `A`  | `in *= what[what_idx]`                                                |
| `T`  | Compare `in == solution[idx]`, met `correct` à 0 si échec            |
| `!`  | Affiche le flag si `correct == 1`, sinon affiche un message d'erreur  |

`what_idx` tourne modulo 4 à chaque opération.

Chaque suite `?...T` constitue une équation à une inconnue qu'on peut bruteforce.

### 3. Extraction des données

- Script de contrôle (`S`) : à partir de l'offset `0x2008` (terminé par `\x00`)
- Tableau `what[4]` : offset `0x3040`
- Tableau `solution[61]` : offset `0x3060` (61 QWords)

### 4. Solve

Solve.py :

```python
#!/usr/bin/env python3
import struct

SCRIPT_SIG = b'?WAWWHT?'
OPS = ('W', 'H', 'A')

def parse_segments(script):
    segments = []
    i = 0
    while i < len(script):
        if script[i] == '?':
            i += 1
            ops = []
            while i < len(script) and script[i] != 'T':
                if script[i] in OPS:
                    ops.append(script[i])
                i += 1
            segments.append(ops)
            i += 1
        else:
            i += 1
    return segments

def main():
    with open('what', 'rb') as f:
        data = f.read()

    script_offset = data.find(SCRIPT_SIG)
    end = data.find(b'\x00', script_offset)
    script = data[script_offset:end].decode()

    segments = parse_segments(script)
    what_offset = 0x3040
    sol_offset  = 0x3060

    what_arr = list(data[what_offset:what_offset+4])
    solution = list(struct.unpack_from(f'<{len(segments)}Q', data, sol_offset))

    flag_chars = []
    what_idx = 0
    for idx, ops in enumerate(segments):
        target = solution[idx]
        for x in range(1, 256):
            val = x
            wi = what_idx
            for op in ops:
                if op == 'W': val ^= what_arr[wi]
                elif op == 'H': val = (val + what_arr[wi]) & ((1 << 64) - 1)
                elif op == 'A': val = (val * what_arr[wi]) & ((1 << 64) - 1)
                wi = (wi + 1) % 4
            if val == target:
                flag_chars.append(chr(x))
                break
        what_idx = (what_idx + len(ops)) % 4

    flag = ''.join(flag_chars)
    print("Flag:", flag)

if __name__ == '__main__':
    main()
```

### 5. Flag

```bash
$ python3 solve.py
Segments detected: 61
what[] = ['0x57', '0x48', '0x41', '0x54']
Flag: bctf{1m_p3rplexed_to_s4y_th3_v3ry_l34st_rzr664k1p5v2qe4qdkym}
```

