---
title: "B01lersCTF 2025 : Trouble_at_the_spa | Web"
date: 2025-04-22
draft: false
categories: ["Web"]
ctf: "B01lersCTF 2025"
challenge: "Trouble_at_the_spa"
author: "Reyko"
---

# Challenge : **trouble at the spa**

## Énoncé : 

> _I had this million‑dollar app idea the other day, but I can't get my routing to work!_ _Can you navigate me to the endpoint of my dreams?_

Une application React obfusquée est hébergée sur GitHub Pages :  
[https://ky28060.github.io/](https://ky28060.github.io/)

Le dépôt source complet est fourni.

## 1. Analyse : 

Sources : 

| Fichier          | Extrait contenu                                         |
| ---------------- | ------------------------------------------------------- |
| `src/main.tsx`   | `<Route path="/flag" element={<Flag />} />`             |
| `src/Flag.tsx`   | `{'bctf{test_flag}'}` _(ou la chaîne chiffrée en prod)_ |
| `src/Header.tsx` | `<a href="/flag">Flag</a>`                              |

- Le router React comporte bien une route interne /flag.
- Le composant `Flag` affiche simplement le flag.

- Le lien du _Header_ est un `<a>` traditionnel : sur GitHub Pages cela déclenche une requête vers _/flag_ → 404.

> **Déduction :** pour voir le flag il suffit d’appliquer la route _à chaud_, sans recharger la page.

Pourquoi le clic normal échoue ?

- GitHub Pages sert uniquement des fichiers statiques.  
Toute URL différente de `/` cherche un fichier réel et renvoie 404.

-  Tant que l’application est déjà chargée, _React Router_ gère la navigation uniquement via l'History API.

## 3. Exploitation : 

Dans F12, exécuter les deux commandes History API :
    
    ```js
    // change l’URL SANS rechargement
    history.pushState({}, '', '/flag');
    // avertit React Router
    dispatchEvent(new PopStateEvent('popstate'));
    ```

|Instruction|Effet|
|---|---|
|`pushState`|Ajoute une entrée `/flag` dans l’historique sans recharger la page.|
|`PopStateEvent`|Émet manuellement l’évènement capté par React Router ; celui‑ci rend la nouvelle route.|

Le flag s'affiche en haut de la page !


## 4. Extraction « offline » : 

Il est aussi possible de télécharger le bundle `index-*.js`, de repérer la table de chaînes et le décodeur RC4 généré par _javascript‑obfuscator_, puis de déchiffrer l’entrée contenant le flag – mais la méthode History API est plus rapide pendant le CTF.

## 5. Flag : 

```
bctf{r3wr1t1ng_h1st0ry_1b07a3768fc}
```


