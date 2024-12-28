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

# - [Aragog](#aragog)

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