---
title: Boolean-BasedBlindSQLinjection 2
published: true
---

---

[SQLi-Blind](https://github.com/blabla1337/skf-labs/tree/master/python/SQLI-blind)

> Detección y explotación Blind SQL Injection. 

En la inyección SQL ciega basada en booleanos, el atacante envía consultas SQL al servidor que dan como resultado una respuesta booleana, como verdadero o falso. El atacante puede utilizar esta técnica para hacer preguntas de sí o no sobre la base de datos y usar las respuestas para inferir información sobre los datos.

![](https://eidd3.github.io/assets/img/Boolean-basedblindSQLinjection2/Untitled.png)

La página tiene varios apartados, como “Welcome”, “About us” o “Contact”, cada uno con su id que se refleja en la URL, al pasarse del id 3 ya muestra un error “page not found!”.

![](https://eidd3.github.io/assets/img/Boolean-basedblindSQLinjection2/Untitled1.png)

Al empezar a probar distintas expresiones vamos a poder identificar si la pagina es vulnerable a SQLi, empezamos con lo clásico `1’ and 1=1-- -`, pero a la web no le gusta y muestra “page not found!” si probamos con doble comilla `1” and 1=1-- -` tampoco le gusta, pero si no colocamos nada y seguido del 1 agregamos `and 1=1-- -` la página no muestra el mensaje del abajo en gris, pero tampoco da error, así que eso es buena señal, si cambiamos la igualdad `and 1=2-- -` vuelve a dar error, por lo que podría ser una indicación de que la página es vulnerable.

Si queremos determinar las columnas dadas para la consulta también se podría enumerar con una consulta “_order by_”, probando distintos números podemos verificar que existen 3 columnas ya que al pasarse de eso, la web da error.

![](https://eidd3.github.io/assets/img/Boolean-basedblindSQLinjection2/Untitled2.png)

Ahora intentaría enumerar el nombre de la tabla en uso, para hacer eso podría usar la siguiente consulta `1 and (select '1' from test)='1'-- -` pero pasando por wfuzz para enumerar la tabla correcta.

![](https://eidd3.github.io/assets/img/Boolean-basedblindSQLinjection2/Untitled3.png)

Y el resultado muestra las tablas “pages” y “users”, con estas tablas podría intuir que en la tabla “users” están las columnas “username” y “password” y tratar de extraer esa información. 

Pruebo tratar de enumerar usuarios con la siguiente consulta `1 and (select substring(username,1,1) from users)='a'-- -` y la respuesta es errónea, pero probando cambiar la a por una mayúscula la respuesta es valida. 

![](https://eidd3.github.io/assets/img/Boolean-basedblindSQLinjection2/Untitled4.png)

![](https://eidd3.github.io/assets/img/Boolean-basedblindSQLinjection2/Untitled5.png)

Cambiado la posición del carácter que estamos comparando por la segunda posición e igualándolo a una d, la respuesta es valida, por lo que podría pensar que el usuario existe y es “Admin” y con este método tratar de enumerar el resto de usuarios y sus contraseñas con un script en Python.  

> Cabe aclarar que para todo esto nos aprovechamos del mensaje “SQL injection - Blind!” en la respuesta.

![](https://eidd3.github.io/assets/img/Boolean-basedblindSQLinjection2/Untitled6.png)

![](https://eidd3.github.io/assets/img/Boolean-basedblindSQLinjection2/Untitled7.png)

Una vez con los hashes, los pasamos por [crackstation](https://crackstation.net/) y nos da las contraseñas en texto claro :)

![](https://eidd3.github.io/assets/img/Boolean-basedblindSQLinjection2/Untitled8.png)


