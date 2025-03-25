---
title: "Kashi CTF 2025 : Selfe Destruct | Forensics"
date: 2025-03-25
draft: false
categories: ["Forensics"]
ctf: "Kashi CTF 2025"
challenge: "Selfe Destruct"
author: "Reyko"
---

Photorec on the vdi

then :

```
[Feb 22, 2025 - 18:31:26 (CET)] exegol-fofo selfe # find recup_dir.* -type f -exec strings {} \; | grep -E '.*fLaG Part.*

grep -rs "fLaG Part " /
'W(# "fLaG Part " /
fLaG Part 7: 'ed_Th0}'
fLaG Part 2: 'm_rf_no_pr'
    char *secret = "fLaG Part 7: \'ed_Th0}\'\n\nfLaG Part 2: \'m_rf_no_pr\'";
echo "fLaG Part 3: 'eserve_roo'"
echo "fLaG Part 5: 'ht??_No_Er'"
grep -rsa "fLaG Part " /
rgrep -sa "fLaG Part " /
# fLaG Part 6: 'r0rs_4ll0w'
# fLaG Part 1: 'KashiCTF{r'
# fLaG Part 4: 't_Am_1_Rig'
echo "fLaG Part 5: 'ht??_No_Er'"
        .string "fLaG Part 7: 'ed_Th0}'\n\nfLaG Part 2: 'm_rf_no_pr'"
fLaG Part 7: 'ed_Th0}'
fLaG Part 2: 'm_rf_no_pr'
# Kernel image management overrides fLaG Part 4: 't_Am_1_Rig'
```
