# Day 30 - Final Practical Challenge

## Day Overview
Welcome to Day 30! Today is a final practical challenge that combines everything you have learned in the last 30 days.
You will plan and execute a full Nmap recon workflow in a controlled, authorized environment, then document results clearly.

## Learning Objectives
By completing Day 30, you will be able to:
- Build a complete recon plan from scope to reporting
- Execute a safe, phased scan workflow
- Use automation to keep scans consistent and repeatable
- Validate results and extract actionable findings
- Produce a professional final report

## Important Ethics Note
Only scan systems you own or have explicit written permission to test.
Do not run this challenge on real networks without authorization.

## Challenge Scenario
You are asked to perform an internal recon assessment for a small organization.
You are given permission to scan the following lab ranges:
- `192.168.50.0/24` (user subnet)
- `192.168.60.0/24` (server subnet)
The goal is to build an asset inventory, identify exposed services, and highlight misconfigurations.

### Example Targets (Lab)
Use one Windows target and one Linux target for focused validation steps:
- Windows: `192.168.60.10`
- Linux: `172.16.60.20`

Note: These are example lab IPs. Replace with your authorized targets if different.

## Rules of Engagement
- Scan only the approved subnets
- Use safe scripts by default
- Avoid brute-force or intrusive scripts
- Document all commands and outputs
- Provide evidence for each finding

## Required Tools
- Nmap
- Basic shell tools (awk, grep)
- Python or PowerShell for automation (optional)

## Step 1: Build a Recon Plan
Document:
- Scope and targets
- Scan phases and timing
- Output structure
- Validation method

## Step 2: Host Discovery
Run discovery scans for each subnet.
```bash
nmap -sn -PR 192.168.50.0/24 -oA outputs/01-discovery-users
nmap -sn -PR 192.168.60.0/24 -oA outputs/01-discovery-servers
```
Extract live hosts:
```bash
awk '/Up$/{print $2}' outputs/01-discovery-users.gnmap > outputs/users-live.txt
awk '/Up$/{print $2}' outputs/01-discovery-servers.gnmap > outputs/servers-live.txt
```

## Step 3: Service Enumeration
Scan common services on live hosts.
```bash
nmap -sV --top-ports 1000 -iL outputs/users-live.txt -oA outputs/02-services-users
nmap -sV --top-ports 1000 -iL outputs/servers-live.txt -oA outputs/02-services-servers
```

## Step 4: Protocol-Specific Deep Dives
Focus on key protocols by subnet.

### SMB and Windows Services
```bash
nmap -p 445 --script smb-os-discovery,smb-enum-shares,smb-security-mode -iL outputs/servers-live.txt -oA outputs/03-smb
```

### Web Services
```bash
nmap -p 80,443,8080,8443 -sV --script http-title,http-headers -iL outputs/servers-live.txt -oA outputs/04-web
```

### TLS and Certificates
```bash
nmap -p 443,8443 --script ssl-cert,ssl-enum-ciphers -iL outputs/servers-live.txt -oA outputs/05-tls
```

### Databases
```bash
nmap -p 1433,1521,3306,5432,27017,6379 -sV --script ms-sql-info,oracle-tns-version,mysql-info,pgsql-info,mongodb-info,redis-info -iL outputs/servers-live.txt -oA outputs/06-db
```

### DNS and Infrastructure
```bash
nmap -p 53 --script dns-recursion,dns-service-discovery -iL outputs/servers-live.txt -oA outputs/07-dns
```

### SNMP (Network Devices)
```bash
nmap -sU -p 161 --script snmp-info -iL outputs/servers-live.txt -oA outputs/08-snmp
```

## Command Output Examples (Realistic Samples)
The outputs below show what the terminal can look like for the commands used in this challenge.
They are examples for the Windows target `192.168.60.10` and Linux target `172.16.60.20`.

### Host Discovery (Users Subnet)
Command:
```bash
nmap -sn -PR 192.168.50.0/24 -oA outputs/01-discovery-users
```
Example output:
```text
Starting Nmap 7.94 ( https://nmap.org ) at 2026-02-06 18:00 UTC
Nmap scan report for 192.168.50.5
Host is up (0.0016s latency).
MAC Address: 00:25:96:FF:EE:DD (Vendor)
Nmap scan report for 192.168.50.22
Host is up (0.0021s latency).
MAC Address: 20:47:47:AA:BB:CC (Vendor)
Nmap done: 256 IP addresses (2 hosts up) scanned in 3.95 seconds
```

### Host Discovery (Servers Subnet)
Command:
```bash
nmap -sn -PR 192.168.60.0/24 -oA outputs/01-discovery-servers
```
Example output:
```text
Starting Nmap 7.94 ( https://nmap.org ) at 2026-02-06 18:02 UTC
Nmap scan report for 192.168.60.10
Host is up (0.0012s latency).
MAC Address: 6C:4B:90:11:22:33 (Vendor)
Nmap scan report for 172.16.60.20
Host is up (0.0019s latency).
Nmap done: 256 IP addresses (2 hosts up) scanned in 3.80 seconds
```

### Service Enumeration (Servers)
Command:
```bash
nmap -sV --top-ports 1000 -iL outputs/servers-live.txt -oA outputs/02-services-servers
```
Example output (snippet):
```text
PORT     STATE SERVICE      VERSION
22/tcp   open  ssh          OpenSSH 8.2p1 Ubuntu 4ubuntu0.7
80/tcp   open  http         Apache httpd 2.4.41 ((Ubuntu))
445/tcp  open  microsoft-ds Windows Server 2019 Standard 17763
3389/tcp open  ms-wbt-server Microsoft Terminal Services
```

### SMB Enumeration (Windows Target)
Command:
```bash
nmap -p 445 --script smb-os-discovery,smb-enum-shares,smb-security-mode 192.168.60.10
```
Example output:
```text
PORT    STATE SERVICE
445/tcp open  microsoft-ds
| smb-os-discovery:
|   OS: Windows Server 2019 Standard 17763
|   Computer name: WIN-SRV01
|   Domain name: CORP
|_  System time: 2026-02-06T18:05:22+00:00
| smb-enum-shares:
|   Public:
|     Type: Disk
|_    Comment: Public Share
| smb-security-mode:
|_  message_signing: enabled but not required
```

### Web Enumeration (Linux Target)
Command:
```bash
nmap -p 80,443,8080,8443 -sV --script http-title,http-headers 172.16.60.20
```
Example output:
```text
PORT    STATE SERVICE VERSION
80/tcp  open  http    Apache httpd 2.4.41
| http-title: Internal App
|_Requested resource was /
| http-headers:
|   Server: Apache/2.4.41 (Ubuntu)
|_  Content-Type: text/html; charset=UTF-8
443/tcp open  https   Apache httpd 2.4.41
```

### TLS Enumeration (Linux Target)
Command:
```bash
nmap -p 443,8443 --script ssl-cert,ssl-enum-ciphers 172.16.60.20
```
Example output:
```text
443/tcp open  https
| ssl-cert: Subject: CN=app.corp.local
|_Not valid after: 2026-12-31
| ssl-enum-ciphers:
|   TLSv1.2:
|_    ciphers: ...
```

### Database Enumeration (Servers)
Command:
```bash
nmap -p 1433,1521,3306,5432,27017,6379 -sV --script ms-sql-info,oracle-tns-version,mysql-info,pgsql-info,mongodb-info,redis-info -iL outputs/servers-live.txt -oA outputs/06-db
```
Example output (snippet):
```text
3306/tcp open  mysql
| mysql-info:
|   Version: 8.0.33
|_  Protocol: 10
1433/tcp open  ms-sql-s
| ms-sql-info:
|   Version: 15.00.2000.05
|_  Product: Microsoft SQL Server 2019
```

### DNS Enumeration (Servers)
Command:
```bash
nmap -p 53 --script dns-recursion,dns-service-discovery -iL outputs/servers-live.txt -oA outputs/07-dns
```
Example output (snippet):
```text
53/tcp open  domain
| dns-recursion:
|_  Recursion appears to be enabled
```

### SNMP Enumeration (Servers)
Command:
```bash
nmap -sU -p 161 --script snmp-info -iL outputs/servers-live.txt -oA outputs/08-snmp
```
Example output (snippet):
```text
161/udp open  snmp
| snmp-info:
|   enterprise: Cisco Systems
|   location: DataCenter-1
|_  uptime: 12 days, 04:21:33
```

### Bash Automation Output
Command:
```bash
./recon.sh
```
Example output:
```text
Outputs in outputs/20260206-1800
```

### Python Automation Output
Command:
```bash
python recon.py
```
Example output (Linux):
```text
Running: nmap -sn -PR 192.168.50.0/24 -oA outputs/20260206-1805/01-discovery-users
Running: nmap -sn -PR 192.168.60.0/24 -oA outputs/20260206-1805/01-discovery-servers
Outputs in outputs/20260206-1805
```
Example output (Windows):
```text
Running: nmap -sn -PR 192.168.50.0/24 -oA outputs\20260206-1805\01-discovery-users
Running: nmap -sn -PR 192.168.60.0/24 -oA outputs\20260206-1805\01-discovery-servers
Outputs in outputs\20260206-1805
```

### PowerShell Automation Output
Command:
```powershell
.\recon.ps1
```
Example output:
```text
Outputs in outputs\20260206-1810
```



## Step 5: Validation and Evidence
Validate critical findings with manual checks or repeated scans.
Document evidence for each finding (output lines, screenshots).

## Step 6: Reporting
Your report should include:
- Scope and scan dates
- Asset inventory summary
- Open ports and service versions
- High-risk exposures and misconfigurations
- Recommended remediation steps

## Example Reporting Template
```text
Title: Internal Recon Assessment
Date: 2026-02-06
Scope: 192.168.50.0/24, 192.168.60.0/24
Summary: X hosts discovered, Y services enumerated
Findings:
- [High] SMBv1 enabled on FILE01
- [Medium] TLS 1.0 supported on intranet server
- [Low] HTTP server banner exposes version
Recommendations:
- Disable SMBv1
- Enforce TLS 1.2+
- Remove unnecessary banners
```

## Automation Option: Bash Workflow
```bash
#!/usr/bin/env bash
set -euo pipefail
OUTDIR="outputs/$(date +%Y%m%d-%H%M%S)"
mkdir -p "$OUTDIR"

nmap -sn -PR 192.168.50.0/24 -oA "$OUTDIR/01-discovery-users"
nmap -sn -PR 192.168.60.0/24 -oA "$OUTDIR/01-discovery-servers"
awk '/Up$/{print $2}' "$OUTDIR/01-discovery-users.gnmap" > "$OUTDIR/users-live.txt"
awk '/Up$/{print $2}' "$OUTDIR/01-discovery-servers.gnmap" > "$OUTDIR/servers-live.txt"
nmap -sV --top-ports 1000 -iL "$OUTDIR/users-live.txt" -oA "$OUTDIR/02-services-users"
nmap -sV --top-ports 1000 -iL "$OUTDIR/servers-live.txt" -oA "$OUTDIR/02-services-servers"
nmap -p 445 --script smb-os-discovery,smb-enum-shares,smb-security-mode -iL "$OUTDIR/servers-live.txt" -oA "$OUTDIR/03-smb"
nmap -p 80,443,8080,8443 -sV --script http-title,http-headers -iL "$OUTDIR/servers-live.txt" -oA "$OUTDIR/04-web"
nmap -p 443,8443 --script ssl-cert,ssl-enum-ciphers -iL "$OUTDIR/servers-live.txt" -oA "$OUTDIR/05-tls"
nmap -p 1433,1521,3306,5432,27017,6379 -sV --script ms-sql-info,oracle-tns-version,mysql-info,pgsql-info,mongodb-info,redis-info -iL "$OUTDIR/servers-live.txt" -oA "$OUTDIR/06-db"
nmap -p 53 --script dns-recursion,dns-service-discovery -iL "$OUTDIR/servers-live.txt" -oA "$OUTDIR/07-dns"
nmap -sU -p 161 --script snmp-info -iL "$OUTDIR/servers-live.txt" -oA "$OUTDIR/08-snmp"
echo "Outputs in $OUTDIR"
```

## Automation Option: Python Workflow
```python
import subprocess
from datetime import datetime
from pathlib import Path

outdir = Path("outputs") / datetime.now().strftime("%Y%m%d-%H%M%S")
outdir.mkdir(parents=True, exist_ok=True)

def run(cmd):
    print("Running:", " ".join(cmd))
    subprocess.run(cmd, check=True)

run(["nmap", "-sn", "-PR", "192.168.50.0/24", "-oA", str(outdir / "01-discovery-users")])
run(["nmap", "-sn", "-PR", "192.168.60.0/24", "-oA", str(outdir / "01-discovery-servers")])
run(["nmap", "-sV", "--top-ports", "1000", "-iL", str(outdir / "01-discovery-users.gnmap"), "-oA", str(outdir / "02-services-users")])
run(["nmap", "-sV", "--top-ports", "1000", "-iL", str(outdir / "01-discovery-servers.gnmap"), "-oA", str(outdir / "02-services-servers")])
run(["nmap", "-p", "445", "--script", "smb-os-discovery,smb-enum-shares,smb-security-mode", "-iL", str(outdir / "01-discovery-servers.gnmap"), "-oA", str(outdir / "03-smb")])
run(["nmap", "-p", "80,443,8080,8443", "-sV", "--script", "http-title,http-headers", "-iL", str(outdir / "01-discovery-servers.gnmap"), "-oA", str(outdir / "04-web")])
run(["nmap", "-p", "443,8443", "--script", "ssl-cert,ssl-enum-ciphers", "-iL", str(outdir / "01-discovery-servers.gnmap"), "-oA", str(outdir / "05-tls")])
run(["nmap", "-p", "1433,1521,3306,5432,27017,6379", "-sV", "--script", "ms-sql-info,oracle-tns-version,mysql-info,pgsql-info,mongodb-info,redis-info", "-iL", str(outdir / "01-discovery-servers.gnmap"), "-oA", str(outdir / "06-db")])
run(["nmap", "-p", "53", "--script", "dns-recursion,dns-service-discovery", "-iL", str(outdir / "01-discovery-servers.gnmap"), "-oA", str(outdir / "07-dns")])
run(["nmap", "-sU", "-p", "161", "--script", "snmp-info", "-iL", str(outdir / "01-discovery-servers.gnmap"), "-oA", str(outdir / "08-snmp")])
print("Outputs in", outdir)
```

## Automation Option: PowerShell Workflow
```powershell
$OutDir = "outputs\" + (Get-Date -Format "yyyyMMdd-HHmmss")
New-Item -ItemType Directory -Path $OutDir | Out-Null

nmap -sn -PR 192.168.50.0/24 -oA "$OutDir\01-discovery-users"
nmap -sn -PR 192.168.60.0/24 -oA "$OutDir\01-discovery-servers"
nmap -sV --top-ports 1000 -iL "$OutDir\01-discovery-users.gnmap" -oA "$OutDir\02-services-users"
nmap -sV --top-ports 1000 -iL "$OutDir\01-discovery-servers.gnmap" -oA "$OutDir\02-services-servers"
nmap -p 445 --script smb-os-discovery,smb-enum-shares,smb-security-mode -iL "$OutDir\01-discovery-servers.gnmap" -oA "$OutDir\03-smb"
nmap -p 80,443,8080,8443 -sV --script http-title,http-headers -iL "$OutDir\01-discovery-servers.gnmap" -oA "$OutDir\04-web"
nmap -p 443,8443 --script ssl-cert,ssl-enum-ciphers -iL "$OutDir\01-discovery-servers.gnmap" -oA "$OutDir\05-tls"
nmap -p 1433,1521,3306,5432,27017,6379 -sV --script ms-sql-info,oracle-tns-version,mysql-info,pgsql-info,mongodb-info,redis-info -iL "$OutDir\01-discovery-servers.gnmap" -oA "$OutDir\06-db"
nmap -p 53 --script dns-recursion,dns-service-discovery -iL "$OutDir\01-discovery-servers.gnmap" -oA "$OutDir\07-dns"
nmap -sU -p 161 --script snmp-info -iL "$OutDir\01-discovery-servers.gnmap" -oA "$OutDir\08-snmp"
Write-Host "Outputs in $OutDir"
```

## Example Outputs (Linux and Windows)
Linux example output (workflow run):
```text
Outputs in outputs/20260206-1800
```
Windows example output (workflow run):
```text
Outputs in outputs\20260206-1800
```

## Tips for Completing the Challenge
- Start with discovery to avoid unnecessary scanning
- Use safe scripts first
- Keep outputs organized by phase
- Validate any high-risk exposures
- Provide clear remediation steps

## Final Checklist
- Scope confirmed and documented
- Discovery completed on all subnets
- Services enumerated and logged
- Protocol deep dives completed
- Findings validated and documented
- Final report created

## Post-Challenge Review
Use this section to reflect and lock in the workflow you built:
- What steps were slow or error‑prone
- Which scripts were most valuable
- Which outputs were hardest to interpret
- Where automation saved the most time

## Optional Extensions (If Time Allows)
These are safe follow‑ups to improve quality without expanding scope:
- Re-run scans with consistent timing to confirm stability
- Compare scans with `ndiff` to detect changes
- Validate TLS findings using a browser or `curl`
- Create a summary table of services by subnet

## Deliverables
At minimum, you should have:
- A clean target inventory
- A discovery output set
- Service enumeration results
- Protocol‑specific outputs (SMB, web, TLS, DB, DNS, SNMP)
- A concise report with evidence

## Proper Closing
You have completed a full, end‑to‑end Nmap reconnaissance workflow.
You can now plan scoped scans, execute safely, automate responsibly, and report clearly.

**Congratulations on completing the full 30‑day Nmap program.**
You now have a repeatable recon process you can use in real engagements with proper authorization.
