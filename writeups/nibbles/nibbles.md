# 🧠 Machine Name: Nibbles

## 📌 Overview

- **Platform:** Hack The Box
    
- **Difficulty:** Easy
    
- **OS:** Linux
    
- **Date:** 16/04/2026
    
- **IP:** 10.129.96.84
    

---

## 🎯 Objective

Get user and root flags.

---

## 🔍 Reconnaissance

### 🔹 Host Discovery

![](./Pasted%20image%2020260416173536.png)

- Host is up
    
- TTL ≈ 63 → likely Linux (default 64, -1 due to routing)
    

---

### 🔹 Nmap Scan

![](./Pasted%20image%2020260416173907.png)


**Results:**

- Open ports: 22 (SSH), 80 (HTTP)
    
- Services: OpenSSH 7.2p2, Apache 2.4.18
    

👉 SSH is not useful for now → focus on HTTP

---

### 🔹 Service Enumeration

![](./Pasted%20image%2020260416174243.png)

- Apache running on port 80
    

---

## 🌐 Enumeration

### 🔹 Web Enumeration

- [http://10.129.96.84](http://10.129.96.84/)
    
- [http://nibbles.htb](http://nibbles.htb/)
    

![](./Pasted%20image%2020260416174426.png)

- Nothing interesting at first glance (classic HTB moment)
    

---

### 🔹 Directory Fuzzing

![](./Pasted%20image%2020260416175302.png)

- No useful results → time to think, not just fuzz
    

---

### 🔹 Manual Inspection

Found reference to:

- `/nibbleblog`
    

![](./Pasted%20image%2020260416175515.png)

👉 That’s more like it

![](./Pasted%20image%2020260416175702.png)

- Blog application detected
    

---

### 🔹 Further Enumeration

![](./Pasted%20image%2020260416180648.png)

Interesting directories:

- `/README`
    
- `/admin.php`
    
- `/plugins`
    

---

### 🔹 Version Disclosure

![](./Pasted%20image%2020260416180752.png)

- Nibbleblog v4.0.3 (2014)
    

👉 Old + CMS = probably vulnerable  
👉 This is smelling like initial access already

---

### 🔹 Admin Panel

![](./Pasted%20image%2020260416185820.png)

- Login panel found
    

👉 Not brute-forcing yet, we still have intel to gather

---

### 🔹 User Enumeration

![](./Pasted%20image%2020260416191909.png)

- Found `users.xml`
    
- Username:
    
    - **admin**
        

---

## 🚨 Initial Access

### 🔹 Vulnerability Identified

- Weak credentials
    
- Vulnerable CMS (Nibbleblog)
    
- File upload via plugins → RCE
    

---

### 🔹 Exploitation Steps

1. Login:
    
    - Username: admin
        
    - Password: nibbles
        

👉 Yes… password is literally _nibbles_. Elite security.

2. Access admin panel
    

![](./Pasted%20image%2020260416192120.png)

👉 We are in. Time to break things.

3. Navigate to:
    
    - Plugins → My Image
        

![](./Pasted%20image%2020260416192153.png)

👉 Upload functionality detected = 🚨

---

### 🔹 Exploit / Payload

Tested upload → accepts PHP

```php
<?php system($_GET['cmd']); ?>
```

👉 “megumi.php” totally legit image btw 😏

![](./Pasted%20image%2020260416193035.png)

---

### 🔹 Result

- Web shell working  
    👉 OKAY WE'RE COOKING 🔥
    

Upgrade to reverse shell:

```bash
nc -lvnp 4444
```

![](./Pasted%20image%2020260416193107.png)

Payload executed → shell received

![](./Pasted%20image%2020260416193826.png)

👉 We are in.

- Retrieved `user.txt`
    

![](./Pasted%20image%2020260416194429.png)

---

## 🧍‍♂️ Privilege Escalation

### 🔹 Enumeration

![](./Pasted%20image%2020260416194728.png)

- Found interesting script:
    
    - `/home/nibbler/personal/stuff/monitor.sh`
        

👉 Suspicious path + script = worth checking

---

### 🔹 Exploit

![](./Pasted%20image%2020260416195428.png)

- Script executable with sudo
    
- Path issues → controllable
    

👉 Script says path doesn’t exist?  
👉 No problem… we create it 😈

Steps:

1. Create required path
    
2. Inject payload
    
3. Execute with sudo
    

---

### 🔹 Root Access

👉 Classic sudo misconfiguration abuse

![](./Pasted%20image%2020260416195631.png)

- Root shell obtained
    
- Root flag captured
    

👉 CHALLENGE COMPLETED 💀

---

## 🧪 Post-Exploitation

- User flag ✔️
    
- Root flag ✔️
    

---

## 🧠 Lessons Learned

### 🔹 What I Learned

- CMS enumeration is key
    
- File upload = easy RCE if misconfigured
    
- Always check sudo permissions
    
- Manual enumeration > blind fuzzing
    

---

### 🔹 Mistakes

- Wasted time fuzzing too early
    
- Didn’t check CMS version fast enough
    

---

### 🔹 Things to Improve

- Faster pattern recognition (CMS, versions)
    
- More efficient enumeration workflow
    

---

## 🧰 Tools Used

- nmap
    
- gobuster
    
- netcat
    
- linpeas
    
- browser + brain 🧠
    

---

## 📚 References

- Nibbleblog vulnerabilities
    
- GTFOBins
    

---

![](./Pasted%20image%2020260416200625.png)