<h1 align="center">Mr. Robot CTF Walkthrough 2021</h1>
 Esto es un write up de la CTF Mr. Robot de la plataforma Try Hack Me (También disponible en VulnHub). Te recomiendo encarecidamente que hagas esta CTF. Las flags no van a ser compartidas.

<p align="center"><img src="img/wall.jpg"></p>

En esta máquina vamos a ver dos formas (entre otras) de explotación de un servicio Wordpress a raíz de su desactualización e inseguridad, posteriormente escalaremos privilegios gracias a una version antigua de nmap.

**Todo ello lo veremos desde la perspectiva y metodología de un test de penetración.**

- Link de la máquina: [Mr. Robot](https://tryhackme.com/room/mrrobot)
- Dificultad asignada por THM: Media
- La IP de la máquina en mi caso será: 10.10.43.249 (Tú tendrás una ip distinta así que cambiala para todos los pasos)

Comencemos!!

<h1 align="center">Reconocimiento</h1>

Lo primero que haremos es escanear la máquina y ver que puertos están abiertos.
Para ello haremos uso de nmap y utilizaremos una serie de flags que hará que nuestro escaneo sea más rápido ya que el escaneo de puertos en ciertas máquinas puede llevar bastante tiempo.

```bash
nmap -p- --open -sS -Pn --min-rate 5000 -v -n 10.10.43.249
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
Scanning 10.10.43.249 [65535 ports]
Discovered open port 443/tcp on 10.10.43.249
Discovered open port 80/tcp on 10.10.43.249
Completed SYN Stealth Scan at 22:24, 26.28s elapsed (65535 total ports)
Nmap scan report for 10.10.43.249
Host is up (0.045s latency).
Not shown: 65532 filtered ports, 1 closed port
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT    STATE SERVICE
80/tcp  open  http
443/tcp open  https
```


