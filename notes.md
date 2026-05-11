## Step 2 — Reconnaissance & Scanning

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
