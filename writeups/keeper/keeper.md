# 🧠 Machine Name: Keeper

## 📌 Overview

- **Platform:** Hack The Box
- **Difficulty:** Easy
- **OS:** Linux
- **IP:** 10.10.11.227

---

## 🎯 Objective

Get user and root flags.

---

## 🔍 Reconnaissance

### 🔹 Nmap Scan

```bash
nmap -sC -sV -oN nmap.txt 10.10.11.227
```

**Results:**

- **22/tcp**: open ssh (OpenSSH 8.9p1)
- **80/tcp**: open http (nginx 1.18.0)

👉 Standard ports. Let's check the web service.

---

## 🌐 Enumeration

### 🔹 Web Enumeration

Visiting `http://10.10.11.227` redirects to `http://tickets.keeper.htb/rt/`.

I'll add it to `/etc/hosts`:
```bash
echo "10.10.11.227 tickets.keeper.htb" | sudo tee -a /etc/hosts
```

![](./keeper_web.png)

The application is **Best Practical Request Tracker (RT)**.

### 🔹 Default Credentials

Searching for RT default credentials: `root:password`.

![](./keeper_login.png)

👉 **Success!** We are logged in as root in the Request Tracker.

### 🔹 Finding User Credentials

Searching through users and tickets, I found a user named `lnorton`.
In the user comments, there's a password mention: `Welcome2023!`.

![](./keeper_creds.png)

---

## 🚨 Initial Access

### 🔹 SSH Login

Trying the credentials `lnorton:Welcome2023!` via SSH.

```bash
ssh lnorton@tickets.keeper.htb
```

👉 **Access Granted!** Managed to grab `user.txt`.

---

## 🧍‍♂️ Privilege Escalation

### 🔹 Enumeration

Inside `lnorton`'s home directory, I found a zip file: `user_management.zip`.
Unzipping it reveals:
- `KeePassDumpFull.dmp`
- `passcodes.kdbx`

👉 It's a KeePass memory dump. This is likely vulnerable to **CVE-2023-32784**.

### 🔹 KeePass Password Extraction

I used a PoC for CVE-2023-32784 to extract the master password from the `.dmp` file.

```bash
python3 keepass_dump_exploit.py KeePassDumpFull.dmp
```

The tool recovered most characters: `rødgrød med fløde` (a Danish dessert).

### 🔹 Cracking the KDBX

With the master password, I opened the `passcodes.kdbx` file.
Inside, I found a root entry containing an SSH private key in PuTTY format (`.ppk`).

### 🔹 PuTTY to OpenSSH

Since I'm on Linux, I need to convert the `.ppk` to an OpenSSH key.

```bash
puttygen root.ppk -O private-openssh -o root.key
chmod 600 root.key
```

### 🔹 Root Access

```bash
ssh -i root.key root@tickets.keeper.htb
```

👉 **Root access attained!** flag captured.

---

## 🧪 Post-Exploitation

- Extracted all flags.
- Cleaned up the exploit scripts.

---

## 🧠 Lessons Learned

- **Default credentials** are still a huge thing in modern labs.
- **Memory forensics** (KeePass dumps) can reveal master passwords even if the database is locked.
- **Conversion of keys** (Putty to OpenSSH) is a necessary skill for cross-platform exploitation.

---

## 🧰 Tools Used

- nmap
- Request Tracker (RT)
- CVE-2023-32784 PoC
- keepassxc / kdbxviewer
- puttygen

---

![](./root_flag.png)
