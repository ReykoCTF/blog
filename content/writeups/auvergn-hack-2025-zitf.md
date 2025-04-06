---
title: "Auvergn'Hack 2025 : Zitf | Hardware"
date: 2025-04-06
draft: false
categories: ["Hardware"]
ctf: "Auvergn'Hack 2025"
challenge: "Zitf"
author: "Reyko"
---

# WU zitf - Hardware - Auvergn'Hack

Challenge :
zitf

Enoncé :
```
As a huge fan of ZITF, you want to display "ZITF" on a 14-segment display. A diagram of the 14 segments layout show how segments should be powered on.
The output should be in the format ZiTF{<bits of letter 1><bits of letter 2>...}.
For example, "ABC" becomes ZiTF{000000111101110000111000111100000000111001}.
Segments A is at position 2^13 and segment N at 2^0.
```

Ressource : 

![14segments](/images/14segments.png)

### Mapping des segments → bits :

A → bit 0
B → bit 1
C → bit 2
D → bit 3
E → bit 4
F → bit 5
G → bit 6
H → bit 7
I → bit 8
J → bit 9
K → bit 10
L → bit 11
M → bit 12
N → bit 13

### Affichage des lettres par segment :

```
    Z utilise les segments :

        A (bit 0)

        L (bit 11)

        M (bit 12)

        D (bit 3)

    I utilise les segments :

        I (bit 8)

        J (bit 9)
        
        A (bit 0)

        D (bit 3)        

    T utilise les segments :

        A (bit 0)

        I (bit 8)

        J (bit 9)

    F utilise les segments :

        A (bit 0)

        F (bit 5)

        E (bit 4)

        G (bit 6)
```

### Bit des lettres ZITF :

```
Z :
Bit	13	12	11	10	9	8	7	6	5	4	3	2	1	0
Val	0	1	1	0	0	0	0	0	0	0	1	0	0	1

I :

Bit	13	12	11	10	9	8	7	6	5	4	3	2	1	0
Val	0	0	0	0	1	1	0	0	0	0	1	0	0	1

T : 

Bit	13	12	11	10	9	8	7	6	5	4	3	2	1	0
Val	0	0	0	0	1	1	0	0	0	0	0	0	0	1

F : 

Bit	13	12	11	10	9	8	7	6	5	4	3	2	1	0
Val	0	0	0	0	0	0	0	1	1	1	0	0	0	1
```

### Flag :

ZiTF{01100000001001000011000010010000110000000100000001110001}
