<h1 align="center">Mr. Robot CTF Walkthrough 2021</h1>
 Esto es un write up de la CTF Mr. Robot de la plataforma Try Hack Me (También disponible en VulnHub). Te recomiendo encarecidamente que hagas esta CTF no sólo por la tematica de la serie sino por que es una máquina buena para práctica y es una máquina OSCP Like. Las flags no van a ser compartidas.

<p align="center"><img src="img/wall.jpg"></p>

En esta máquina vamos a ver dos formas (entre otras) de explotación de un servicio Wordpress a raíz de su desactualización e inseguridad, posteriormente escalaremos privilegios gracias a una version antigua de nmap.

**Todo ello lo veremos desde la perspectiva y metodología de un test de penetración.**

- Link de la máquina: [Mr. Robot](https://tryhackme.com/room/mrrobot)
- Dificultad asignada por THM: Media
- La IP de la máquina en mi caso será: 10.10.198.171 (Tú tendrás una ip distinta así que cambiala para todos los pasos)

Comencemos!!

<h1 align="center">Reconocimiento</h1>

Lo primero que haremos es escanear la máquina y ver que puertos están abiertos.
Para ello haremos uso de nmap y utilizaremos una serie de flags que hará que nuestro escaneo sea más rápido ya que el escaneo de puertos en ciertas máquinas puede llevar bastante tiempo.

```bash
nmap -p- --open -sS -Pn --min-rate 5000 -v -n 10.10.198.171
```

La explicación del significado de cada flag es la siguiente:

- ```-p-``` : indicamos que el escaneo se hará para todos los puertos.
- ```--open``` : indicamos que sólo nos interesan los puertos que esten abiertos.
- ```-sS``` : Esta flag indica que queremos hacer un "SYN Scan" lo cual significa que que los paquetes que mandaremos nunca completarán las conexiones TCP y eso hará que nuestro escaneo sea mucho menos intrusivo y más silencioso.
- ```-Pn``` : Con esta opción indicamos que no queremos hacer host discovery (ya que sabemos quien es nuestro objetivo).
- ```--min-rate 5000``` : Esta flag puede ser intercambiada por ```-T5```, ambas tienen la finalidad de hacer que nuestro escaneo sea más rapido (y ruidoso...). Para ser más detallista dicha flag indica que no queremos mandar menos de 5.000 paquetes por segundo.
- ```-v``` : (verbose) Para ir viendo que puertos nos aparecen sobre la marcha.
- ```-n``` : No queremos que se nos realice resolucion DNS, ya que a quien escaneamos es a una IP, no a un dominio.

Del cual obtenemos el siguiente output:

```
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times will be slower.
Starting Nmap 7.91 ( https://nmap.org ) at 2021-09-02 22:23 CEST
Initiating SYN Stealth Scan at 22:23
Scanning 10.10.198.171 [65535 ports]
Discovered open port 443/tcp on 10.10.198.171
Discovered open port 80/tcp on 10.10.198.171
Completed SYN Stealth Scan at 22:24, 26.28s elapsed (65535 total ports)
Nmap scan report for 10.10.198.171
Host is up (0.045s latency).
Not shown: 65532 filtered ports, 1 closed port
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT    STATE SERVICE
80/tcp  open  http
443/tcp open  https
```

En este punto sabemos que hay 2 puertos abiertos: 80 (HTTP) y 443 (HTTPS), viendo esto y que el puerto SSH no está abierto (almenos al exterior), se puede deducir que la única via de entrar en la máquina es a través de dichos servicios web.

El paso por excelencia una vez que sabemos que puertos estan abiertos es realizar un analisis a esos puertos mediante la ejecución de una serie de scripts con el fin de obtener más información: version de servidor, tecnología, posibles vulnerabilidades a priori, etc...

```bash
nmap -sV -sC -p 80,443 -Pn -n -min-rate 5000 10.10.198.171
```

Donde :

- ```-sV``` : Nos mostrará si es posible la versión del servicio que esta corriendo en cada puerto.
- ```-A``` : Ejecutaremos sobre dichos puertos todos los scripts relevantes (proporcionados por nmap).
- ```-p 80,443```: Los puertos abiertos.

Obteniendo este output:

```
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times will be slower.
Starting Nmap 7.91 ( https://nmap.org ) at 2021-09-02 22:26 CEST
Nmap scan report for 10.10.198.171
Host is up (0.044s latency).

PORT    STATE SERVICE  VERSION
80/tcp  open  http     Apache httpd
|_http-server-header: Apache
|_http-title: Site doesn't have a title (text/html).
443/tcp open  ssl/http Apache httpd
|_http-server-header: Apache
|_http-title: Site doesn't have a title (text/html).
| ssl-cert: Subject: commonName=www.example.com
| Not valid before: 2015-09-16T10:45:03
|_Not valid after:  2025-09-13T10:45:03
```

Para saber con que tratamos vamos a ejecutar **WhatWeb**

```http://10.10.198.171 [200 OK] Apache, Country[RESERVED][ZZ], HTML5, HTTPServer[Apache], IP[10.10.198.171], Script, UncommonHeaders[x-mod-pagespeed], X-Frame-Options[SAMEORIGIN]```

Parece que no nos ha arrojado mucha información...

Así que vamos a visitar la web... Después de una impresionante introducción muy hacker, nos encontramos con este menú.

<p align="center"><img src="img/1.png"></p>

Después de probar cada uno de los comandos y ver varios videos, no encontre nada relevante, por otro lado viendo el código fuente de la página podrémos encontrar en la línea 15 una IP que actualmente no nos da ninguna información adicional.

En este punto conocemos las siguientes rutas:

```
/prepare
/fsociety
/inform
/question
/wakeup
/join
```

Ya que no parece haber más información a la vista, procederemos a hacer *Fuzzing*, lo cual consiste en hacer peticiones al servidor de diversas rutas extraidas de un diccionario con el objetivo de obtener rutas que existan. Para ello usaremos *Wfuzz* aunque otra potente herramienta es *ffuf*.

```bash
wfuzz -c -L --hc=404 -t 200 -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt http://10.10.198.171//FUZZ
```

- Con ffuf sería:
 * ```bash
 ffuf -u http://10.10.198.171/FUZZ -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -mc 200 -c -t 200
  ```









