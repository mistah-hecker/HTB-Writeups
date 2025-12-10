# HTB: Expressway Write-Up

This is my personal write-up of the HTB Expressway machine. I’m still a beginner, so this write-up is written in a simple way. My goal was to document what I did, what commands I used, and what I understood from the machine.

---

## Overview

**Difficulty:** Easy

**Objective:** Get user and root flags

**Techniques Used:** UDP enumeration, IKE Aggressive Mode, PSK cracking, sudo misconfiguration

---

## 1. Information Gathering

### 🔍 Nmap Scans (TCP & UDP)

I started with basic service discovery.

**Commands used:**

```
nmap -sV -O <TARGET_IP>
nmap -sU -sV --top-ports 100 <TARGET_IP>
```

* `-sV` was used to check service versions
* `-O` to detect the OS
* `-sU` to scan UDP
* `--top-ports 100` because UDP scans are slow

### Findings

* **Port 22 (SSH)** — will be used later to get inside
* **Port 500/UDP (ISAKMP / IKE)** — this is the interesting one

At this point, I didn’t fully understand IKE or IPSec, but I learned that IKE is part of VPN technology.

---

## 2. Service Enumeration (IKE / IPSec)

IKE (Internet Key Exchange) is used in IPSec VPNs. It decides things like:

* What hashing algorithm to use
* What encryption method
* What authentication method

IKE has two modes:

* **Main Mode** — secure
* **Aggressive Mode** — faster but leaks too much info

The target machine had Aggressive Mode enabled.

### Checking IKE

```
sudo ike-scan <TARGET_IP>
```

This showed weak algorithms:

* 3DES (weak)
* SHA1 (weak)

### Trying Aggressive Mode

```
sudo ike-scan --aggressive <TARGET_IP>
```

This leaked the identity:

* Something like: `ike@expressway.htb`

### Extracting the PSK Hash

```
ike-scan -M -A <TARGET_IP> --pskcrack=output.txt
```

* `-M` makes the output readable
* `-A` forces Aggressive Mode
* `--pskcrack` saves the hash for cracking

### Cracking the PSK

```
hashcat -m 5400 -a 0 output.txt /usr/share/wordlists/rockyou.txt
```

* Mode 5400 is for IKE-PSK (SHA1)
* `rockyou.txt` is the wordlist

**The password was cracked successfully.**

---

## 3. Getting User Access

Now I had the username and the cracked password.

SSH into the machine:

```
ssh user@<TARGET_IP>
```

I entered the cracked password and got in.

I listed files:

```
ls
```

Then read the user flag:

```
cat user.txt
```

---

## 4. Privilege Escalation

I checked sudo permissions:

```
sudo -l
```

There was a binary/script that the user could run as root without a password.

This binary was misconfigured and allowed privilege escalation.

I executed it:

```
sudo /path/to/binary
```

This gave me root access.

I moved to the root directory:

```
cd /root
ls
cat root.txt
```

And grabbed the root flag.

---

## Final Thoughts

This was one of my first machines involving VPN/IKE and I honestly struggled at first. But it helped me understand:

* UDP enumeration
* What IKE and Aggressive Mode are
* Offline hash cracking
* Checking sudo misconfigurations

