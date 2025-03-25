---
title: "Kashi CTF 2025 : Do Not Redeem #1 | Forensics"
date: 2025-03-25
draft: false
categories: ["Forensics"]
ctf: "Kashi CTF 2025"
challenge: "Do Not Redeem #1"
author: "Reyko"
---

Find the message DB :

```
[Feb 23, 2025 - 10:22:53 (CET)] exegol-fofo kitler # find . -type f -exec sh -c 'echo "File: $1"; strings "$1"' _ {} \; | grep -E '.*mmssms.db.*'
File: ./data/data/com.android.providers.telephony/databases/mmssms.db-journal
File: ./data/data/com.android.providers.telephony/databases/mmssms.db
ce.cm.notify_mmssms_db_lost
ce.cm.notify_mmssms_db_losttrue*
File: ./data/user/0/com.android.providers.telephony/databases/mmssms.db-journal
File: ./data/user/0/com.android.providers.telephony/databases/mmssms.db
ce.cm.notify_mmssms_db_lost
ce.cm.notify_mmssms_db_losttrue*
File: ./data/user_de/0/com.android.providers.telephony/databases/mmssms.db-journal
File: ./data/user_de/0/com.android.providers.telephony/databases/mmssms.db
        mmssms.db
mmssms.db.txt
File: ./data_mirror/data_ce/null/0/com.android.providers.telephony/databases/mmssms.db-journal
File: ./data_mirror/data_ce/null/0/com.android.providers.telephony/databases/mmssms.db
ce.cm.notify_mmssms_db_lost
ce.cm.notify_mmssms_db_losttrue*
File: ./data_mirror/data_de/null/0/com.android.providers.telephony/databases/mmssms.db-journal
File: ./data_mirror/data_de/null/0/com.android.providers.telephony/databases/mmssms.db

```

mmssms.db is the classic db where we can find messages.

Then search on the DB :

```
[Feb 23, 2025 - 10:24:17 (CET)] exegol-fofo kitler # sqlite3 "./data/data/com.android.providers.telephony/databases/mmssms.db" "SELECT date, body FROM sms WHERE body LIKE '%839216%';"
1740251608654|839216 is your Amazon OTP. Don't share it with anyone.

```
