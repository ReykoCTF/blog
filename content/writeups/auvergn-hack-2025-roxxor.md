---
title: "Auvergn'Hack 2025 : Roxxor Monkey 1/3 | OSINT"
date: 2025-04-06
draft: false
categories: ["OSINT"]
ctf: "Auvergn'Hack 2025"
challenge: "Roxxor Monkey"
author: "Reyko"
---

# WU Roxxor Monkey 1/3 - Hardware - Auvergn'Hack

Challenge :
Roxxor Monkey 1/3

Enoncé :
```
Je l'ai plus mais on nous donnait le nom Roxxor Monkey en nous disant qu'il communique sur les réseaux publiques.
```

### recherche de compte :

Recherche automatisée avec l'outil Sherlock :

```
[Apr 06, 2025 - 14:48:42 (CEST)] exegol-enable /workspace # sherlock RoxxorMonkey                                   
[*] Checking username RoxxorMonkey on:

[+] Freelance.habr: https://freelance.habr.com/freelancers/RoxxorMonkey
[+] GNOME VCS: https://gitlab.gnome.org/RoxxorMonkey
[+] GitHub: https://www.github.com/RoxxorMonkey
[+] LibraryThing: https://www.librarything.com/profile/RoxxorMonkey
[+] Mydramalist: https://www.mydramalist.com/profile/RoxxorMonkey
[+] NationStates Nation: https://nationstates.net/nation=RoxxorMonkey
[+] NationStates Region: https://nationstates.net/region=RoxxorMonkey
[+] Twitter: https://x.com/RoxxorMonkey
[+] YandexMusic: https://music.yandex/users/RoxxorMonkey/playlists
[+] YouTube: https://www.youtube.com/@RoxxorMonkey

```
Après avoir passé les liens, on trouve rapidement que le plus intéressant est le GitHub :
https://github.com/RoxxorMonkey
Repo : https://github.com/RoxxorMonkey/monkey-demo


### Recherches sur le repo :

En OSINT sur GitHub, le plus souvent les flags sont cachés dans les diff des commit :

On trouve directement le flag ici :
https://github.com/RoxxorMonkey/monkey-demo/commit/5ed6de9e77fbd0ae4a8c9594ff7a775304670aa1
```
API_KEY=ZiTF{9c179a69404bd29b8899648e02ce9076}
URL="https://zitf.fr/api/bananas"
```
