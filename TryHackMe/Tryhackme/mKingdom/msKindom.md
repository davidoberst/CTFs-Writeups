primero ejecuto un escaneo de puertos : 


davidoberst@archlinux ~]$ nmap -sV -O 10.65.149.61
TCP/IP fingerprinting (for OS scan) requires root privileges.
QUITTING!
[davidoberst@archlinux ~]$ sudo nmap -sV -O 10.65.149.61
[sudo] password for davidoberst: 
Starting Nmap 7.99 ( https://nmap.org ) at 2026-06-29 14:28 -0500
Nmap scan report for 10.65.149.61
Host is up (0.069s latency).
Not shown: 999 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
85/tcp open  http    Apache httpd 2.4.7 ((Ubuntu))
No exact OS matches for host (If you know what OS is running on it, see https://nmap.org/submit/ ).
TCP/IP fingerprint:
OS:SCAN(V=7.99%E=4%D=6/29%OT=85%CT=1%CU=37937%PV=Y%DS=3%DC=I%G=Y%TM=6A42C77
OS:2%P=x86_64-pc-linux-gnu)SEQ(SP=104%GCD=1%ISR=10D%TI=Z%CI=I%II=I%TS=8)SEQ
OS:(SP=108%GCD=1%ISR=107%TI=Z%CI=I%II=I%TS=8)SEQ(SP=108%GCD=1%ISR=109%TI=Z%
OS:CI=I%II=I%TS=8)SEQ(SP=F7%GCD=1%ISR=10A%TI=Z%CI=I%II=I%TS=8)SEQ(SP=FB%GCD
OS:=1%ISR=104%TI=Z%CI=I%II=I%TS=8)OPS(O1=M4E8ST11NW7%O2=M4E8ST11NW7%O3=M4E8
OS:NNT11NW7%O4=M4E8ST11NW7%O5=M4E8ST11NW7%O6=M4E8ST11)WIN(W1=68DF%W2=68DF%W
OS:3=68DF%W4=68DF%W5=68DF%W6=68DF)ECN(R=Y%DF=Y%T=40%W=6903%O=M4E8NNSNW7%CC=
OS:Y%Q=)T1(R=Y%DF=Y%T=40%S=O%A=S+%F=AS%RD=0%Q=)T2(R=N)T3(R=N)T4(R=Y%DF=Y%T=
OS:40%W=0%S=A%A=Z%F=R%O=%RD=0%Q=)T5(R=Y%DF=Y%T=40%W=0%S=Z%A=S+%F=AR%O=%RD=0
OS:%Q=)T6(R=Y%DF=Y%T=40%W=0%S=A%A=Z%F=R%O=%RD=0%Q=)T7(R=Y%DF=Y%T=40%W=0%S=Z
OS:%A=S+%F=AR%O=%RD=0%Q=)U1(R=Y%DF=N%T=40%IPL=164%UN=0%RIPL=G%RID=G%RIPCK=G
OS:%RUCK=G%RUD=G)IE(R=Y%DFI=N%T=40%CD=S)

Network Distance: 3 hops




al entrar, puedo ver la app, muestra un mensaje que dice :

Bwa, ha, ha, pathetic, you'll never learn!

hare un listado de directorios con gobster para ver si hay mas directrorios : 

[davidoberst@archlinux Web-Content]$ gobuster dir -u http://10.65.149.61:85/ -w common.txt
===============================================================
Gobuster v3.8.2
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.65.149.61:85/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                common.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.8.2
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
.htpasswd            (Status: 403) [Size: 288]
.hta                 (Status: 403) [Size: 283]
.htaccess            (Status: 403) [Size: 288]
app                  (Status: 301) [Size: 312] [--> http://10.65.149.61:85/app/]
index.html           (Status: 200) [Size: 647]
server-status        (Status: 403) [Size: 292]
Progress: 4751 / 4751 (100.00%)
===============================================================
Finished
===============================================================
[davidoberst@archlinux Web-Content]$ 


puedo ver que app, esta disonible, al entrar me muestra una url : 
http://10.65.149.61:85/app/castle/index.php
donde se ve un blog, usando como tcnologia PHP,Apache,Ubuntu y COncrete CMS

hare otro fuzzing a : 
http://10.65.149.61:85/app/castle/
con el comando: 
gobuster dir -u http://10.65.149.61:85/app/castle/ -w common.txt

el fuzzing encontro cosas interesantes : 

[+] Url:                     http://10.65.149.61:85/app/castle/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                common.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.8.2
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
.hta                 (Status: 403) [Size: 294]
.htaccess            (Status: 403) [Size: 299]
.htpasswd            (Status: 403) [Size: 299]
application          (Status: 301) [Size: 331] [--> http://10.65.149.61:85/app/castle/application/]
concrete             (Status: 301) [Size: 328] [--> http://10.65.149.61:85/app/castle/concrete/]
index.php            (Status: 200) [Size: 12047]
packages             (Status: 301) [Size: 328] [--> http://10.65.149.61:85/app/castle/packages/]
robots.txt           (Status: 200) [Size: 532]
updates              (Status: 301) [Size: 327] [--> http://10.65.149.61:85/app/castle/updates/]
Progress: 4751 / 4751 (100.00%)


al listar los driectorios activos, tenemos : 

http://10.65.149.61:85/app/castle/application/ ---> Una pagina en blanco, interesante.

http://10.65.149.61:85/app/castle/concrete/ ---> Toda la estructura de archivos del servidor web.

http://10.65.149.61:85/app/castle/packages/ ---> direcotio de Apache sin contenido. 

http://10.65.149.61:85/app/castle/updates/ ---> direcotio de Apache sin contenido. 

robots.txt   ---> mapeo de la pagina, muestra el contenido : 

User-agent: *
Disallow: /application/attributes
Disallow: /application/authentication
Disallow: /application/bootstrap
Disallow: /application/config
Disallow: /application/controllers
Disallow: /application/elements
Disallow: /application/helpers
Disallow: /application/jobs
Disallow: /application/languages
Disallow: /application/mail
Disallow: /application/models
Disallow: /application/page_types
Disallow: /application/single_pages
Disallow: /application/tools
Disallow: /application/views
Disallow: /ccm/system/captcha/picture


----REVISION PROFUNDA DE LAS URLS ENCONTRADAS -------

 http://10.65.149.61:85/app/castle/concrete/ muestra toda la estructutra php del servidor web,sin embago, en todas dice acces denied, excepto en los archivos js.

tambien contre un directorio login.php

al entrar pude  acceder con las credenciales admin password



