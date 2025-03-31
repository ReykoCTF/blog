---
title: "NullCon 2025 : Bfail | Web / Crypto"
date: 2025-03-25
draft: false
categories: ["Web"] ["Crypto"]
ctf: "NullCon 2025"
challenge: "bfail"
author: "Reyko"
---

# Bfail : 

URL: http://52.59.124.14:5013/
Flag : `ENO{BCRYPT_FAILS_TO_B_COOL_IF_THE_PW_IS_TOO_LONG}`

## 1. Contexte

On a une page de login un peu bidon. Dans le code source (accessible via /source ), on
trouve :
- Le formulaire de login qui est en **POST**, mais la route n'accepte que les **GET**.
- La vérif se fait avec `bcrypt.checkpw(password, app.ADMIN_PW_HASH)` .
- Le mot de passe de l’admin est généré avec **128 octets** de données aléatoires, mais **bcrypt ne prend en compte que les 72 premiers octets**.
- Le code source balance les premiers octets (environ 70/71 octets), donc il ne reste qu’à deviner les 2 derniers octets (65 536 possibilités).

## 2. Brute-force Off-Line
### 2.1. Brute-force Multicoeurs (bfail_brute_multi.py)
**Disclaimer :**
Le script est optimisé pour une machine avec **24 coeurs**. Si tu n'as pas un ordi de compète,
adapte-le ! (ou achète un ordi de compète :kek:)

Script complet :
```python
import bcrypt
import multiprocessing

partial = b'\xec\x9f\xe0a\x978\xfc\xb6:T\xe2\xa0\xc9<\x9e\x1a\xa5\xfao\xb2\x15\x86\xe5\x24\x86Z\x1a\xd4\xca#\x15\xd2x\xa0\x0e0\xca\xbc\x89T\xc5V6\xf1\xa4\xa8S\x8a%I\xd8gI\x15\xe9\xe7$M\x15\xdc@\xa9\xa1@\x9c\xeee\xe0\xe0\xf76'
pw_hash = b"$2b$12$8bMrI6D9TMYXeMv8pq8RjemsZg.HekhkQUqLymBic/cRhiKRa3YPK"

TOTAL = 65536

def worker(worker_id, start, end, verbose_step=1000):
    for i, val in enumerate(range(start, end)):
        b1 = val >> 8
        b2 = val & 0xff
        candidate = partial + bytes([b1, b2])
        candidate = candidate[:72]
        
        if bcrypt.checkpw(candidate, pw_hash):
            return (b1, b2)
        
        if i % verbose_step == 0:
            print(f"[Worker {worker_id}] {i+start} / {end} essais...")
    
    return None

def main():
    workers = 24  # Oh oui les 24 coeurs
    pool = multiprocessing.Pool(workers)
    
    step = TOTAL // workers
    remainder = TOTAL % workers
    tasks = []
    current_start = 0
    
    for w_id in range(workers):
        chunk_size = step + (1 if w_id < remainder else 0)
        current_end = current_start + chunk_size
        tasks.append(pool.apply_async(worker, args=(w_id, current_start, current_end)))
        current_start = current_end
    
    pool.close()
    pool.join()
    
    for t in tasks:
        result = t.get()
        if result:
            b1, b2 = result
            print("[+] FOUND CORRECT PASSWORD!")
            print("Last two bytes: 0x{:02x} 0x{:02x}".format(b1, b2))
            candidate = partial + bytes([b1, b2])
            candidate = candidate[:72]
            print("Full 72-byte password:", candidate)
            return

if __name__ == "__main__":
    main()
```
Output du script de compétition :
```text
[+] FOUND CORRECT PASSWORD!
Last two bytes: 0xaa 0x00
Full 72-byte password:
b'\xec\x9f\xe0a\x978\xfc\xb6:T\xe2\xa0\xc9<\x9e\x1a\xa5\xfao\xb2\x15\x86\xe5$\x86Z\x1a\xd4\xc
a#\x15\xd2x\xa0\x0e0\xca\xbc\x89T\xc5V6\xf1\xa4\xa8S\x8a%I\xd8gI\x15\xe9\xe7
$M\x15\xdc@\xa9\xa1@\x9c\xeee\xe0\xe0\xf
76\xaa\x00'
```
### 2.2. Vérification du Candidat (testbcrypt.py)
On vérifie que les 72 premiers octets du candidat fonctionnent bien avec bcrypt :

```python
import bcrypt
pw_hash = b"$2b$12$8bMrI6D9TMYXeMv8pq8RjemsZg.HekhkQUqLymBic/cRhiKRa3YPK"
candidate_73 =
b'\xec\x9f\xe0a\x978\xfc\xb6:T\xe2\xa0\xc9<\x9e\x1a\xa5\xfao\xb2\x15\x86\xe5
$\x86Z\x1a\xd4\xca#\x15\xd2x\xa0\x0e0\xca\xbc\x89T\xc5V6\xf1\xa4\xa8S\x8a%I\
xd8gI\x15\xe9\xe7$M\x15\xdc@\xa9\xa1@\x9c\xeee\xe0\xe0\xf76\xaa\x00'
candidate_72 = candidate_73[:72]
print('candidate_73 length:', len(candidate_73))
print('candidate_72 length:', len(candidate_72))
print('bcrypt.checkpw(candidate_72, pw_hash) ->',
bcrypt.checkpw(candidate_72, pw_hash))
```
Output : 
```text
candidate_73 length: 73 candidate_72 length: 72 bcrypt.checkpw(candidate_72,pw_hash) -> True
```
### 2.3. Génération de la Chaîne URLEncodée
Pour envoyer le mot de passe au serveur, il faut l'URLencoder exact.
**Important** : Dans la chaîne binaire, le symbole dollar ( `$` ) doit être échappé dans le shell en
écrivant `\$` .

Commande :

```
python3 -c "import urllib.parse; candidate_72 =
b'\xec\x9f\xe0a\x978\xfc\xb6:T\xe2\xa0\xc9<\x9e\x1a\xa5\xfao\xb2\x15\x86\xe5
\$\x86Z\x1a\xd4\xca#\x15\xd2x\xa0\x0e0\xca\xbc\x89T\xc5V6\xf1\xa4\xa8S\x8a%I
\xd8gI\x15\xe9\xe7\$M\x15\xdc@\xa9\xa1@\x9c\xeee\xe0\xe0\xf76\xaa\x00'[:72];
print(urllib.parse.quote_from_bytes(candidate_72))"
```

Output :
```text
%EC%9F%E0a%978%FC%B6%3AT%E2%A0%C9%3C%9E%1A%A5%FAo%B2%15%86%E5%24%86Z%1A%D4%C
A%23%15%D2x%A0%0E0%CA%BC%89T%C5V6%F1%A4%
A8S%8A%25I%D8gI%15%E9%E7%24M%15%DC%40%A9%A1%40%9C%EEe%E0%E0%F76%AA
```

## 3. Envoi de la Requête GET pour Obtenir le Flag

```
curl -v -X GET \
-H "Content-Type: application/x-www-form-urlencoded" \
--data-binary
"username=admin&password=%EC%9F%E0a%978%FC%B6%3AT%E2%A0%C9%3C%9E%1A%A5%FAo%B2%15%86%E5%24%86Z%1A%D4%CA%23%15%D2x%A0%0E0%CA%BC%89T%C5V6%F1%A4%A8S%8A%25I%D8gI%15%E9%E7%24M%15%DC%40%A9%A1%40%9C%EEe%E0%E0%F76%AA" \
http://52.59.124.14:5013/
```

Réponse du serveur :
```bash
* Trying 52.59.124.14:5013...
* Connected to 52.59.124.14 (52.59.124.14) port 5013 (#0)
> GET / HTTP/1.1
> Host: 52.59.124.14:5013
> User-Agent: curl/7.88.1
> Accept: */*
> Content-Type: application/x-www-form-urlencoded
> Content-Length: 206
>
< HTTP/1.1 200 OK
< Server: Werkzeug/3.1.3 Python/3.13.1
< Date: Sat, 01 Feb 2025 18:30:29 GMT
< Content-Type: text/html; charset=utf-8
< Content-Length: 125
< Connection: close
<
* Closing connection 0
Congrats! It appears you have successfully bf'ed the password. Here is your
ENO{BCRYPT_FAILS_TO_B_COOL_IF_THE_PW_IS_TOO_LONG}
```

## 4. Conclusion
### Pour résumer:
### 1 - Analyse du Code Source :
On a vu que le mot de passe de l'admin est généré sur 128 octets, mais bcrypt ne considère
que les 72 premiers. Le code source balance déjà la majeure partie du mot de passe, donc il ne
restait qu'à brute-forcer les 2 derniers octets (65 536 possibilités).
### 2 - Brute-force :
Le Script `bfail_brute_multi.py` de la mort qui tue a trouvé que les 2 derniers octets étaient
`0xaa` et `0x00`.
### 3 - Vérification et Préparation :
Avec `testbcrypt.py`, on confirme que les 72 premiers octets (candidat correct) passent la
vérif de bcrypt. Ensuite, il reste plus qu'a généré la chaîne URLencodée en prenant soin
d'échapper les dollars.
### 4 - Envoi via curl :
On envoie pour finir la requête GET avec `--data-binary` pour que la donnée ne soit pas
modifiée, et le serveur renvoie le flag.

Made with <3 with my best bro openai
