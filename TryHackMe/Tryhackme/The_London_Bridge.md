
-primero iniciamos con un escaneo en nmap : 

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

al subir una imagen, esta es la peticion web que obtenemos : 

POST /upload HTTP/1.1
Host: 10.66.143.62:8080
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:152.0) Gecko/20100101 Firefox/152.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.9
Accept-Encoding: gzip, deflate, br
Content-Type: multipart/form-data; boundary=----geckoformboundary1587d414b0cfa8fc5980b2d48789e99a
Content-Length: 15392
Origin: http://10.66.143.62:8080
Connection: keep-alive
Referer: http://10.66.143.62:8080/gallery
Upgrade-Insecure-Requests: 1
Priority: u=0, i

------geckoformboundary1587d414b0cfa8fc5980b2d48789e99a
Content-Disposition: form-data; name="file"; filename="caspian.jpeg"
Content-Type: image/jpeg

ÿØÿà

nada extraño por ahora o que no sd emas informacion, intentare ver si puedo obtener acceso a una shell, pero antes de crear una reverse shell y un puerto escuchando, primero validare si es posible y no hay validaciones por parte del servidor, empezare con subir algo con una extension .php y ver que pasa.

luego de subir la imagen el servidor responde  con : 

Uploaded file is not an image

al parecer solo puedo subir imagenes, en ese caso tendre que meter un payload dentro de una imagen, o tal vez solo cambiar la extension? 

cambie la extension, per aun asi reconocio que no es una imagen

Uploaded file is not an image

al ver el codigo interno de la pagina, puedo ver el comentario que dice : 

To devs: Make sure that people can also add images using link

Intente metodos para ingresar con un payload a reverse shell ,pero no funcionaron, intente hacer otro fuzzing a el sitio, esta vez con un diccionario mas grande 

encintre otro directroio :

view_image           
dejaview 


dentro del otro directorio me permite enviar urls de imagenes al servidor para renderizarlas en el navegador, puede ser un indicio de SSRF (Server Side Request Forgery) .

























