
primero iniciamos con un escaneo en nmap : 


[davidoberst@archlinux ~]$ sudo nmap -sV -O -T5 10.66.143.62
[sudo] password for davidoberst: 
Starting Nmap 7.99 ( https://nmap.org ) at 2026-07-25 16:42 -0500
Nmap scan report for 10.66.143.62
Host is up (0.21s latency).
Not shown: 998 closed tcp ports (reset)
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.7 (Ubuntu Linux; protocol 2.0)
8080/tcp open  http    Gunicorn
Device type: general purpose|phone
Running (JUST GUESSING): Linux 5.X|6.X|4.X (96%), Google Android 10.X|11.X|12.X (93%)
OS CPE: cpe:/o:linux:linux_kernel:5 cpe:/o:linux:linux_kernel:6 cpe:/o:linux:linux_kernel:4 cpe:/o:google:android:10 cpe:/o:google:android:11 cpe:/o:google:android:12 cpe:/o:linux:linux_kernel:5.4
Aggressive OS guesses: Linux 5.14 - 6.8 (96%), Linux 4.15 - 5.19 (96%), Linux 4.15 (96%), Linux 5.4 - 5.15 (96%), Android 10 - 12 (Linux 4.14 - 4.19) (93%), Android 10 - 11 (Linux 4.9 - 4.14) (92%), Android 12 (Linux 5.4) (92%), Android 9 - 11 (Linux 4.9 - 4.14) (92%), Linux 2.6.32 (92%), Linux 2.6.39 - 3.2 (92%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 3 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel


entraremos a la pagina http://10.66.143.62:8080/ 

en la pagina no vi mucho, me interesaron unas funciones, permite subir imagenes, o archivos, revidsare luego para ver si es una posible entrada para una reverse shell, co nwhatweb pude ver que el sitio web esta sobre gunicorn 

[davidoberst@archlinux ~]$ whatweb http://10.66.143.62:8080
http://10.66.143.62:8080 [200 OK] Country[RESERVED][ZZ], HTML5, HTTPServer[gunicorn], IP[10.66.143.62], Title[Explore London]
[davidoberst@archlinux ~]$ 


hare un fuzzing al sitio con gobuster :

[davidoberst@archlinux Web-Content]$ gobuster dir -u http://10.66.143.62:8080 -w common.txt
===============================================================
Gobuster v3.8.2
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.66.143.62:8080
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                common.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.8.2
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
contact              (Status: 200) [Size: 1703]
feedback             (Status: 405) [Size: 178]
gallery              (Status: 200) [Size: 1722]
upload               (Status: 405) [Size: 178]
Progress: 4751 / 4751 (100.00%)
===============================================================
Finished
===============================================================
[davidoberst@archlinux Web-Content]$ 


al ver las paginas, parece que no hay mucho, solo lo que la pagina muestra a vista previa, hare un fuzzing a cada uno de los resultados obtenidos.

Nada encontrado en el fuzzing a los resultados anteriores, en ese caso continuare con lo que tengo, como mencione antes, me llama la atencion el diurectorio upload, que en realidad no responde a peticiones GET, solo POST, de todas formas la pagina tiene un boton de upload, antes de subir algo, debo entender, que quiero subir? tengo datos de un servidor Ubuntu, subire una foto normal primero para ver que pasa, ebo entender el servidor primero. Subire una imagen e interceptare la peticion con BurpSuite para verlo mas a fondo.








