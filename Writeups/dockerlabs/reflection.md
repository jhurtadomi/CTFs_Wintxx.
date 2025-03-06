# MACHINE REFLECTION

## INFORMACIÓN GENERAL


- **Plataforma**: DockerLabs
- **Sistema Operativo**: Linux
- **Dificultad**: Fácil

![machine_dockerlabs](https://github.com/Jean25-sys/CTFs_Wintx/blob/main/Writeups/dockerlabs/images/reflection/machine_dockerlabs.png)

## DESPLIEGUE

```python
bash autodeploy.sh reflection.tar
```
![despliegue](https://github.com/Jean25-sys/CTFs_Wintx/blob/main/Writeups/dockerlabs/images/reflection/desplieguie.png)

## RECONOCIMIENTO

```python
nmap -p- -sS --min-rate 5000 -vvvv -n -Pn -sCV 172.17.0.2
```
```ruby
PORT   STATE SERVICE REASON         VERSION
22/tcp open  ssh     syn-ack ttl 64 OpenSSH 9.2p1 Debian 2+deb12u3 (protocol 2.0)
| ssh-hostkey: 
|   256 89:6c:a5:af:d5:e2:83:6c:f9:87:33:44:0f:78:48:3a (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBMps8s+30oFAKg2941FBb6ll6Wz5WNZOVIZRGUJGalfdnfePoKGgyGnxQBLJCH4ewP7EvGTD+ge0gGr0FIzeMPk=
|   256 65:32:42:95:ca:d0:53:bb:28:a5:15:4a:9c:14:64:5b (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIMt8jE0CMbImPmdadSD5x0yuU9HV0ZU5FWVG5FcbJ2KV
80/tcp open  http    syn-ack ttl 64 Apache httpd 2.4.62 ((Debian))
|_http-title: Laboratorio de Cross-Site Scripting (XSS)
| http-methods: 
|_  Supported Methods: OPTIONS HEAD GET POST
|_http-server-header: Apache/2.4.62 (Debian)
MAC Address: 02:42:AC:11:00:02 (Unknown)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```


- **22/tcp** - OpenSSH 9.2: Indica que el servidor tiene SSH habilitado.
- **80/tcp** - Apache 2.4.62: Se encuentra un servicio web en este puerto con un título que indica un laboratorio de XSS.

![laboratorios_xss](https://github.com/Jean25-sys/CTFs_Wintx/blob/main/Writeups/dockerlabs/images/reflection/Laboratorios_xss.png)
Podemos ver 4 laboratorios de `Cross-Site Scripting (XSS)` y nos dice que para acceder a los diferentes niveles debemos inyectar distintos payloads en los campos de cada laboratorio

# EXPLOTACION
## Primer Laboratorio
Ok, entonces vamos a ver que nos trae el primer laboratorio de `XSS Reflected`, si probamos con el payload mas básico como:
```html
<script>alert('PruebaXSS');</script>
```
No muestra nada, al parecer hay algun fragmento de código que está sanitizado o esta aplicando algun filtrado que evita la ejecucion de esta etiqueta, felizmente que existen muchas maneras de poder saltar
estas reglas como:

Eventos HTML
```html
<img src=x onerror=alert('Pwned')>
```
![pwned](https://github.com/Jean25-sys/CTFs_Wintx/blob/main/Writeups/dockerlabs/images/reflection/pwned.png)

ByPASS de Filtros

```html
<a href=java&#1&#2&#3&#4&#5&#6&#7&#8&#11&#12script:javascript:alert(1)>XXX</a>
```

![haz_click_aqui](https://github.com/Jean25-sys/CTFs_Wintx/blob/main/Writeups/dockerlabs/images/reflection/haz_click_aqui.png)

Vemos que podemos ejecutarlo sin problemas, una vez echo esto, podemos hacer ataques como robo de cookies, Phishing mediante Pop Ups, Redireccion a sitios maliciosos,etc.

## Segundo Laboratorio

En el Segundo Laboratorio nos dice que podemos introducir cualquier contenido incluso si tiene la etiqueta `<script>`, en este caso como es `XSS Almacenado` todo payload que ingresemos lo almacenara
y se ejecutará en el navegador de otros usuarios, podemos enviar los mimos payloads anteriores
![xss_storage](https://github.com/Jean25-sys/CTFs_Wintx/blob/main/Writeups/dockerlabs/images/reflection/xss_storage.png)
![xss_storage1](https://github.com/Jean25-sys/CTFs_Wintx/blob/main/Writeups/dockerlabs/images/reflection/xss_storage1.png)

## Tercer Laboratorio

Tenemos 3 Opciones, donde elegimos ciertos parametros y enviamos, y como dice el Laboratorio vamnos a interceptar con BurpSuite para hacer algunas pruebas
![burpsuite1](https://github.com/Jean25-sys/CTFs_Wintx/blob/main/Writeups/dockerlabs/images/reflection/burpsuite1.png)
Podemos modificar el contenido de las Opciones, en BurpSuite tenemos que aplicar URL Encode a algunas etiquetas
![burpsuite2](https://github.com/Jean25-sys/CTFs_Wintx/blob/main/Writeups/dockerlabs/images/reflection/burpsuite2.png)

Pero si lo hacemos en la Web Original, no es tan necesario porque jalo de primera sin URL Encode
![BurpSuite3](https://github.com/Jean25-sys/CTFs_Wintx/blob/main/Writeups/dockerlabs/images/reflection/burpsuite3.png)

## Cuarto Laboratorio
Nos dice que debemos agregar el parametro `?data` en la url para reflejar el contenido, esta actuando como un parámetro de consulta y podemos cargar el payload
que venimos utilizando anteriormente
![lab4](https://github.com/Jean25-sys/CTFs_Wintx/blob/main/Writeups/dockerlabs/images/reflection/Lab4.png)

### UNA VEZ COMPLETADO LOS LABORATORIOS NOS PROPORCIONARÁ LAS CREDENCIALES QUE PODEMOS UTILIZAR EN SSH PORQUE TENIA EL PUERTO ABIERTO `balu:balulero`

![credenciales](https://github.com/Jean25-sys/CTFs_Wintx/blob/main/Writeups/dockerlabs/images/reflection/credenciales.png)
![balu_login](https://github.com/Jean25-sys/CTFs_Wintx/blob/main/Writeups/dockerlabs/images/reflection/balu_login.png)

# ESCALADA DE PRIVILEGIOS

Si hacemos un `sudo -l` no tenemos nada para aprovecharse entonces buscamos binarios con permisos SUID para ver si utilizamos alguno
```css
find / -perm -4000 2>/dev/null
```
![binario_env](https://github.com/Jean25-sys/CTFs_Wintx/blob/main/Writeups/dockerlabs/images/reflection/binario_env.png)

Para esta escalada podemos usar la pagina de ![GTFOBins](https://gtfobins.github.io/gtfobins/env/#sudo)
![gtfobins](https://github.com/Jean25-sys/CTFs_Wintx/blob/main/Writeups/dockerlabs/images/reflection/gtfobins.png)

![root](https://github.com/Jean25-sys/CTFs_Wintx/blob/main/Writeups/dockerlabs/images/reflection/rootpng.png)


# ELIMINAR LA MAQUINA

```ruby
ctrl + c
```


