---
title: "ThCon 2k25 : Foreign Interference ? Part 1,2 et 3 | Stegano "
date: 2025-04-13
draft: false
categories: ["Stegano"]
ctf: "ThCon 2k25"
challenge: "Foreign Interference ? Part 1,2 et 3"
author: "Reyko"
---

## WU - Foreign Interference ? Part 1,2 et 3 - Stegano - ThCon 2k25

### Part 1 and 2

### Contexte :

#### Nom :
Part 1 :
Foreign Interference ? : Are they hidding something ?

Part 2 :
Foreign Interference ? : The quality's so bad I can't hear anything

Part 3 :
Foreign Interference ? : XSS's real origin

#### Enoncé :

Part 1 :
```
S.N.A.F.U. is concerned that foreign adversaries may be behind the sudden gang attacks. Your colleagues at SNAFU have found some files on the computer of a gang member that goes by the name of Viktor (aka "Crypt") including a copy of the NUSA (New United States of America) anthem. Try to find out what this file contains: foreign agent codes, passwords, or maybe it's just a (very) poor-quality audio file? Flag format THC{...}
```

part 2 :
```
Note this challenge uses the same file as the other Foreign Interference ? challenge

S.N.A.F.U. is concerned that foreign adversaries may be behind the sudden gang attack. Your colleagues at SNAFU have found some files on the computer of a gang member that goes by the name of Viktor including a copy of the NUSA (New United States of America) anthem. Try to find out what this file contains: foreign agent codes, passwords, or maybe it's just a (very) poor-quality audio file?

flag format : A famous sentence in lower case with only spaces between words. Add the THC{} around Examples :

Cogito, ergo sum : THC{cogito ergo sum}
Fiat lux : THC{fiat lux}
```

Part 3 :
```
It seems like you have found a passphrase ("god save the king"), and a locker ! Perhaps this will confirm our suspicions on the country that is backing our opponents, although we at the SNAFU already have some suspicions.
```

#### Ressource :

- national_anthem.wav

### Solve :

Part 1 :

$ steghide extract -sf national_anthem.wav

Aucun mot de passe n'est requis, cette commande recover le fichier "youlost" :

$ ls -hail youlost
total 16K
6291928 drwxr-xr-x 2 1000 rvm 4.0K Jan  3 12:18 .
6291908 drwxrws--- 3 root 993 4.0K Apr 12 17:36 ..
6291929 -rw-r--r-- 1 1000 rvm  159 Jan  3 11:55 flag.md
6291930 -rwxrwxrwx 1 1000 rvm 2.4K Jan  3 11:49 kings_locker.kdbx

$ cat flag.md
You need the passphrase to open the locker (only lowercase letter, with spaces)
But here's a first flag for you : THC{1t_c4n'7_b3_th3_NUSA_th3y_h4v3_n0_k1ng5}

THC{1t_c4n'7_b3_th3_NUSA_th3y_h4v3_n0_k1ng5} est donc le flag de la partie 1.

Pour la partie 2 nous avons un fichier de type BDD Keypass, il faut donc trouver son mot de passe qui constituera le flag de la partie 2.

**GUESS TIME ! :** (part 2)

3 indices :

- flag format : A famous sentence in lower case with only spaces between words. Add the THC{} around Examples : Cogito, ergo sum : THC{cogito ergo sum} / Fiat lux : THC{fiat lux}

- Le flag de la partie 1 : 1t_c4n'7_b3_th3_NUSA_th3y_h4v3_n0_k1ng5

- Le nom de la BDD : kings_locker.kdbx

On parle donc a plusieurs reprise de roi, un mot de passe est fait pour protégé une BDD donc instinctivement on guess "god save the king" (oui bon OK c'est de la change)

Part 3 :

Il suffit de déverouiller la BDD Keypass avec le password "god save the king" et on obtient le flag de la partie 3 !

Pour l'intender solve, je pense qu'il fallait faire un traitement dans le spectre audio du wav pour pouvoir écouter "god save the king"
