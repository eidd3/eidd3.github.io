---
title: TryHackMe - UltraTech
published: true
---

---
## Reconocimiento Inicial

Se realiza un escaneo de puertos con **nmap** para identificar los servicios activos en la máquina objetivo (10.10.15.224):

```bash
nmap -sS -p- --open --min-rate 2000 -n -Pn 10.10.15.224
```

![](https://eidd3.github.io/assets/img/UltraTech/1.png)


Posteriormente se ejecuta un escaneo más detallado en los puertos identificados:

```bash
nmap -sCV -Pn -p21,22,8081,31331 10.10.15.224 -oN Ports.txt
```

![](https://eidd3.github.io/assets/img/UltraTech/2.png)

### Puertos Abiertos

- Puerto 21: FTP - vsftpd 3.0.5 
- Puerto 22: SSH - OpenSSH 8.2p1 
- Puerto 8081: HTTP - Node.js Express framework
- Puerto 31331: HTTP - Apache/2.4.41

Puerto 21? sin credenciales a probar, puerto 22 lo mismo, enumero el puerto 8081.

En el puerto **8081** la web se ve asi:

![](https://eidd3.github.io/assets/img/UltraTech/3.png)


Comienzo haciendo un poco de fuzz en la web.

> 💡 Al acceder a cualquier ruta vemos el mensaje de error del framework de Node.js Express. También podría ser Fiber, pero la diferencia es que Express genera un HTML mínimo con la respuesta, mientras que Fiber solo muestra texto plano. Con esto también confirmamos que se trata de Express.

![](https://eidd3.github.io/assets/img/UltraTech/3-5.png)

![](https://eidd3.github.io/assets/img/UltraTech/4.png)

![](https://eidd3.github.io/assets/img/UltraTech/5.png)

Siguiendo con la enumeración, no encuentro gran cosa: una ruta `/auth` que solicita los parámetros _login_ y _password_ para autenticarse, pero no dispongo de credenciales; y una ruta `/ping` que muestra rutas del sistema.

![](https://eidd3.github.io/assets/img/UltraTech/7.png)

![](https://eidd3.github.io/assets/img/UltraTech/8.png)

Por el momento lo dejo ahí y empiezo a enumerar el puerto **31331**.

Viendo un poco la web, lo primero que llama la atención son los nombres del equipo y sus supuestos nombres de usuario.

![](https://eidd3.github.io/assets/img/UltraTech/9.png)

Por lo tanto, guardé esa información en un archivo.

El resultado de dirsearch muestra las siguientes rutas:

![](https://eidd3.github.io/assets/img/UltraTech/10.png)

### Descubrimiento de Command Injection

Lo primero que veo es la ruta `/js` que tiene directory listing y los siguientes archivos:

![](https://eidd3.github.io/assets/img/UltraTech/11.png)

**api.js:**

```javascript
(function() {
    console.warn('Debugging ::');

    function getAPIURL() {
	return `${window.location.hostname}:8081`
    }
    
    function checkAPIStatus() {
	const req = new XMLHttpRequest();
	try {
	    const url = `http://${getAPIURL()}/ping?ip=${window.location.hostname}`
	    req.open('GET', url, true);
	    req.onload = function (e) {
		if (req.readyState === 4) {
		    if (req.status === 200) {
			console.log('The api seems to be running')
		    } else {
			console.error(req.statusText);
		    }
		}
	    };
	    req.onerror = function (e) {
		console.error(xhr.statusText);
	    };
	    req.send(null);
	}
	catch (e) {
	    console.error(e)
	    console.log('API Error');
	}
    }
    checkAPIStatus()
    const interval = setInterval(checkAPIStatus, 10000);
    const form = document.querySelector('form')
    form.action = `http://${getAPIURL()}/auth`;
    
})();
```

El código realiza peticiones **GET** a sí mismo cada 10 segundos para comprobar el estado de la API. Esto ocurre en el puerto **8081**, no en el **31331**.

Volviendo a ese _endpoint_ en el puerto **8081** vemos el output del comando.

![](https://eidd3.github.io/assets/img/UltraTech/12.png)

Y si intento hacer un ping a mi **IP**, también lo ejecuta.

![](https://eidd3.github.io/assets/img/UltraTech/13.png)

Probando separadores comunes como `&`, `;` y `|` para intentar una inyección de comandos después de la **IP**, ninguno funcionó hasta que probé con la comilla invertida (`) y realicé una petición con curl para comprobarlo.

![](https://eidd3.github.io/assets/img/UltraTech/14.png)

Acá intenté generar la reverse shell de varias maneras, pero no funcionó. Entonces intenté usar comandos típicos para ver si aparecían en el output _(es lo que tendría que haber probado antes de la reverse shell, xd)_.

![](https://eidd3.github.io/assets/img/UltraTech/15.png)

### Explotación - Obtención de Credenciales

Al listar el contenido se ve un archivo **utech.db.sqlite** y, al leerlo, muestra usuarios con un hash a su lado.

![](https://eidd3.github.io/assets/img/UltraTech/16.png)

Con CrackStation comprobamos si coinciden con alguna contraseña y se muestran ambas.

![](https://eidd3.github.io/assets/img/UltraTech/17.png)

Me logueo con usuario _admin_ pero solo se ve este mensaje para r00t.

![](https://eidd3.github.io/assets/img/UltraTech/18.png)

### Acceso SSH

Con los nombres de usuario guardados previamente y estas dos contraseñas, realicé un password‑spraying contra el servicio **SSH**.

![](https://eidd3.github.io/assets/img/UltraTech/19.png)

### Escalada de Privilegios

Una vez dentro, se observa que el usuario **r00t** pertenece al grupo _docker_, lo cual es riesgoso, ya que podría ejecutar Docker sin necesidad de privilegios de root, lo que permitiría:
- Crear contenedores con acceso total al sistema de archivos del host.
- Montar directorios del host dentro del contenedor.
- Ejecutar comandos como root dentro del contenedor.

![](https://eidd3.github.io/assets/img/UltraTech/20.png)


#### Conclusión

La máquina fue comprometida explotando una vulnerabilidad de command injection en el endpoint /ping del servicio Node.js Express. Esto permitió acceder a credenciales almacenadas en la base de datos SQLite, las cuales fueron crackeadas y reutilizadas para obtener acceso SSH a la máquina.
