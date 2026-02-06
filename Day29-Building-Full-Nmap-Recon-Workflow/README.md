# Day 29 - Building a Full Nmap Recon Workflow

## Day Overview
Welcome to Day 29! Today you will build a full Nmap reconnaissance workflow that you can reuse across engagements.
This guide combines planning, discovery, enumeration, validation, and reporting into one repeatable process.

## Learning Objectives
By completing Day 29, you will be able to:
- Design a full recon workflow with clear phases
- Create consistent output structure for analysis
- Automate multi-step scans with scripts
- Combine safe and advanced scans responsibly
- Produce results ready for reporting and remediation

## Important Ethics Note
Only scan systems you own or have explicit written permission to test.
Always follow rules of engagement and document all actions.

## Workflow Summary
A full recon workflow usually includes:
1. Scope and planning
2. Target inventory
3. Host discovery
4. Service enumeration
5. Protocol-specific deep dives
6. Validation and verification
7. Reporting and remediation guidance

## Phase 1: Scope and Planning
Define and document:
- Approved IP ranges and domains
- Out-of-scope systems
- Allowed scan types and scripts
- Time windows and notification contacts

## Phase 2: Target Inventory
Build a single, validated target list:
- DNS records and subdomains
- Asset inventory exports
- Cloud inventory and external endpoints

Example `targets.txt`:
```text
192.168.10.0/24
10.0.5.0/24
web01.corp.local
db01.corp.local
```

## Phase 3: Host Discovery
Use low-noise discovery to find live hosts:
```bash
nmap -sn -PR 192.168.10.0/24 -oA outputs/01-discovery
```
Extract live hosts:
```bash
awk '/Up$/{print $2}' outputs/01-discovery.gnmap > outputs/live.txt
```

## Phase 4: Service Enumeration
Scan common services on live hosts:
```bash
nmap -sV --top-ports 1000 -iL outputs/live.txt -oA outputs/02-services
```

## Phase 5: Protocol-Specific Deep Dives
Target common internal protocols and exposed services.

### SMB and Windows Services
```bash
nmap -p 445 --script smb-os-discovery,smb-enum-shares,smb-security-mode -iL outputs/live.txt -oA outputs/03-smb
```

### Web Services
```bash
nmap -p 80,443,8080,8443 -sV --script http-title,http-headers -iL outputs/live.txt -oA outputs/04-web
```

### TLS and Certificates
```bash
nmap -p 443,8443 --script ssl-cert,ssl-enum-ciphers -iL outputs/live.txt -oA outputs/05-tls
```

### Databases
```bash
nmap -p 1433,1521,3306,5432,27017,6379 -sV --script ms-sql-info,oracle-tns-version,mysql-info,pgsql-info,mongodb-info,redis-info -iL outputs/live.txt -oA outputs/06-db
```

### DNS and Infrastructure
```bash
nmap -p 53 --script dns-recursion,dns-service-discovery -iL outputs/live.txt -oA outputs/07-dns
```

### SNMP (Network Devices)
```bash
nmap -sU -p 161 --script snmp-info -iL outputs/live.txt -oA outputs/08-snmp
```

## Phase 6: Validation and Verification
Validate critical findings with:
- Repeat scans with safe timing
- Manual checks (browser, curl)
- Cross-checking DNS and certificates

## Phase 7: Reporting and Remediation
Summarize:
- Scope and timing
- Open ports and services
- High-risk exposures
- Remediation guidance

## Output Structure (Recommended)
```text
outputs/
  01-discovery.*
  02-services.*
  03-smb.*
  04-web.*
  05-tls.*
  06-db.*
  07-dns.*
  08-snmp.*
  live.txt
```

## Automation: Bash Workflow (Linux/macOS)
```bash
#!/usr/bin/env bash
set -euo pipefail
RANGE="${1:-192.168.10.0/24}"
OUTDIR="outputs/$(date +%Y%m%d-%H%M%S)"
mkdir -p "$OUTDIR"

nmap -sn -PR "$RANGE" -oA "$OUTDIR/01-discovery"
awk '/Up$/{print $2}' "$OUTDIR/01-discovery.gnmap" > "$OUTDIR/live.txt"
nmap -sV --top-ports 1000 -iL "$OUTDIR/live.txt" -oA "$OUTDIR/02-services"
nmap -p 445 --script smb-os-discovery,smb-enum-shares -iL "$OUTDIR/live.txt" -oA "$OUTDIR/03-smb"
nmap -p 80,443,8080,8443 -sV --script http-title,http-headers -iL "$OUTDIR/live.txt" -oA "$OUTDIR/04-web"
nmap -p 443,8443 --script ssl-cert,ssl-enum-ciphers -iL "$OUTDIR/live.txt" -oA "$OUTDIR/05-tls"
nmap -p 1433,1521,3306,5432,27017,6379 -sV --script ms-sql-info,oracle-tns-version,mysql-info,pgsql-info,mongodb-info,redis-info -iL "$OUTDIR/live.txt" -oA "$OUTDIR/06-db"
nmap -p 53 --script dns-recursion,dns-service-discovery -iL "$OUTDIR/live.txt" -oA "$OUTDIR/07-dns"
nmap -sU -p 161 --script snmp-info -iL "$OUTDIR/live.txt" -oA "$OUTDIR/08-snmp"
echo "Outputs in $OUTDIR"
```

## Automation: Python Workflow (Cross-Platform)
```python
import subprocess
from datetime import datetime
from pathlib import Path

range_cidr = "192.168.10.0/24"
outdir = Path("outputs") / datetime.now().strftime("%Y%m%d-%H%M%S")
outdir.mkdir(parents=True, exist_ok=True)

def run(cmd):
    print("Running:", " ".join(cmd))
    subprocess.run(cmd, check=True)

run(["nmap", "-sn", "-PR", range_cidr, "-oA", str(outdir / "01-discovery")])
run(["nmap", "-sV", "--top-ports", "1000", "-iL", str(outdir / "01-discovery.gnmap"), "-oA", str(outdir / "02-services")])
run(["nmap", "-p", "445", "--script", "smb-os-discovery,smb-enum-shares", "-iL", str(outdir / "01-discovery.gnmap"), "-oA", str(outdir / "03-smb")])
run(["nmap", "-p", "80,443,8080,8443", "-sV", "--script", "http-title,http-headers", "-iL", str(outdir / "01-discovery.gnmap"), "-oA", str(outdir / "04-web")])
run(["nmap", "-p", "443,8443", "--script", "ssl-cert,ssl-enum-ciphers", "-iL", str(outdir / "01-discovery.gnmap"), "-oA", str(outdir / "05-tls")])
run(["nmap", "-p", "1433,1521,3306,5432,27017,6379", "-sV", "--script", "ms-sql-info,oracle-tns-version,mysql-info,pgsql-info,mongodb-info,redis-info", "-iL", str(outdir / "01-discovery.gnmap"), "-oA", str(outdir / "06-db")])
run(["nmap", "-p", "53", "--script", "dns-recursion,dns-service-discovery", "-iL", str(outdir / "01-discovery.gnmap"), "-oA", str(outdir / "07-dns")])
run(["nmap", "-sU", "-p", "161", "--script", "snmp-info", "-iL", str(outdir / "01-discovery.gnmap"), "-oA", str(outdir / "08-snmp")])
print("Outputs in", outdir)
```

## Automation: PowerShell Workflow (Windows)
```powershell
$Range = "192.168.10.0/24"
$OutDir = "outputs\" + (Get-Date -Format "yyyyMMdd-HHmmss")
New-Item -ItemType Directory -Path $OutDir | Out-Null

nmap -sn -PR $Range -oA "$OutDir\01-discovery"
Get-Content "$OutDir\01-discovery.gnmap" | ForEach-Object { if ($_ -match "Up$") { ($_ -split " ")[1] } } | Set-Content "$OutDir\live.txt"
nmap -sV --top-ports 1000 -iL "$OutDir\live.txt" -oA "$OutDir\02-services"
nmap -p 445 --script smb-os-discovery,smb-enum-shares -iL "$OutDir\live.txt" -oA "$OutDir\03-smb"
nmap -p 80,443,8080,8443 -sV --script http-title,http-headers -iL "$OutDir\live.txt" -oA "$OutDir\04-web"
nmap -p 443,8443 --script ssl-cert,ssl-enum-ciphers -iL "$OutDir\live.txt" -oA "$OutDir\05-tls"
nmap -p 1433,1521,3306,5432,27017,6379 -sV --script ms-sql-info,oracle-tns-version,mysql-info,pgsql-info,mongodb-info,redis-info -iL "$OutDir\live.txt" -oA "$OutDir\06-db"
nmap -p 53 --script dns-recursion,dns-service-discovery -iL "$OutDir\live.txt" -oA "$OutDir\07-dns"
nmap -sU -p 161 --script snmp-info -iL "$OutDir\live.txt" -oA "$OutDir\08-snmp"
Write-Host "Outputs in $OutDir"
```

## Example Outputs (Linux and Windows)
Linux example output (workflow run):
```text
Outputs in outputs/20260206-1700
```
Windows example output (workflow run):
```text
Outputs in outputs\20260206-1700
```

## Command and Script Output Examples
The outputs below are **example outputs** for the commands and automation in this workflow.
Actual results will vary based on environment, permissions, and targets.

### Example Output: Host Discovery
Command:
```bash
nmap -sn -PR 192.168.10.0/24 -oA outputs/01-discovery
```
Example output (Linux):
```text
Starting Nmap 7.94 ( https://nmap.org ) at 2026-02-06 17:00 UTC
Nmap scan report for 192.168.10.1
Host is up (0.0014s latency).
MAC Address: 00:11:22:33:44:55 (Vendor)
Nmap scan report for 192.168.10.10
Host is up (0.0019s latency).
MAC Address: AA:BB:CC:DD:EE:FF (Vendor)
Nmap done: 256 IP addresses (2 hosts up) scanned in 3.70 seconds
```
Example output (Windows):
```text
Starting Nmap 7.94 ( https://nmap.org ) at 2026-02-06 17:00 UTC
Nmap scan report for 192.168.10.1
Host is up (0.0025s latency).
Nmap scan report for 192.168.10.10
Host is up (0.0034s latency).
Nmap done: 256 IP addresses (2 hosts up) scanned in 4.20 seconds
```

### Example Output: Service Enumeration
Command:
```bash
nmap -sV --top-ports 1000 -iL outputs/live.txt -oA outputs/02-services
```
Example output (Linux snippet):
```text
PORT    STATE SERVICE VERSION
22/tcp  open  ssh     OpenSSH 8.2p1
80/tcp  open  http    Apache httpd 2.4.41
445/tcp open  microsoft-ds Windows Server 2019 Standard 17763
```
Example output (Windows snippet):
```text
PORT     STATE SERVICE      VERSION
135/tcp  open  msrpc        Microsoft Windows RPC
445/tcp  open  microsoft-ds Windows Server 2016 Standard 14393
3389/tcp open  ms-wbt-server Microsoft Terminal Services
```

### Example Output: SMB Deep Dive
Command:
```bash
nmap -p 445 --script smb-os-discovery,smb-enum-shares,smb-security-mode -iL outputs/live.txt -oA outputs/03-smb
```
Example output (Linux snippet):
```text
PORT    STATE SERVICE
445/tcp open  microsoft-ds
| smb-os-discovery:
|   OS: Windows Server 2019 Standard 17763
|_  Domain name: CORP
| smb-enum-shares:
|   Public:
|_    Comment: Public Share
```
Example output (Windows snippet):
```text
| smb-security-mode:
|_  message_signing: enabled but not required
```

### Example Output: Web Services
Command:
```bash
nmap -p 80,443,8080,8443 -sV --script http-title,http-headers -iL outputs/live.txt -oA outputs/04-web
```
Example output (Linux snippet):
```text
80/tcp  open  http
| http-title: Intranet Portal
|_Requested resource was /
| http-headers:
|   Server: nginx
|_  Content-Type: text/html
```
Example output (Windows snippet):
```text
443/tcp open  https
| http-headers:
|   Server: Microsoft-IIS/10.0
|_  X-Powered-By: ASP.NET
```

### Example Output: TLS Checks
Command:
```bash
nmap -p 443,8443 --script ssl-cert,ssl-enum-ciphers -iL outputs/live.txt -oA outputs/05-tls
```
Example output (Linux snippet):
```text
443/tcp open  https
| ssl-cert: Subject: CN=intranet.corp.local
|_Not valid after: 2026-12-31
| ssl-enum-ciphers:
|   TLSv1.2:
|_    ciphers: ...
```
Example output (Windows snippet):
```text
8443/tcp open  https
| ssl-enum-ciphers:
|   TLSv1.3:
|_    ciphers: ...
```

### Example Output: Database Enumeration
Command:
```bash
nmap -p 1433,1521,3306,5432,27017,6379 -sV --script ms-sql-info,oracle-tns-version,mysql-info,pgsql-info,mongodb-info,redis-info -iL outputs/live.txt -oA outputs/06-db
```
Example output (Linux snippet):
```text
3306/tcp open  mysql
| mysql-info:
|   Version: 8.0.33
|_  Protocol: 10
5432/tcp open  postgresql
| pgsql-info:
|_  Version: 13.9
```
Example output (Windows snippet):
```text
1433/tcp open  ms-sql-s
| ms-sql-info:
|   Version: 15.00.2000.05
|_  Product: Microsoft SQL Server 2019
```

### Example Output: DNS Checks
Command:
```bash
nmap -p 53 --script dns-recursion,dns-service-discovery -iL outputs/live.txt -oA outputs/07-dns
```
Example output (Linux snippet):
```text
53/tcp open  domain
| dns-recursion:
|_  Recursion appears to be enabled
```
Example output (Windows snippet):
```text
53/tcp open  domain
| dns-service-discovery:
|_  \_ldap._tcp: 192.168.10.10
```

### Example Output: SNMP Checks
Command:
```bash
nmap -sU -p 161 --script snmp-info -iL outputs/live.txt -oA outputs/08-snmp
```
Example output (Linux snippet):
```text
161/udp open  snmp
| snmp-info:
|   enterprise: Cisco Systems
|   location: DataCenter-1
|_  uptime: 12 days, 04:21:33
```
Example output (Windows snippet):
```text
161/udp open  snmp
| snmp-info:
|   enterprise: Microsoft
|_  uptime: 3 days, 02:11:05
```

### Example Output: Bash Workflow Script
Command:
```bash
./recon.sh 192.168.10.0/24
```
Example output (Linux/macOS):
```text
Outputs in outputs/20260206-1700
```

### Example Output: Python Workflow Script
Command:
```bash
python recon.py
```
Example output (Linux):
```text
Running: nmap -sn -PR 192.168.10.0/24 -oA outputs/20260206-1705/01-discovery
Running: nmap -sV --top-ports 1000 -iL outputs/20260206-1705/01-discovery.gnmap -oA outputs/20260206-1705/02-services
Outputs in outputs/20260206-1705
```
Example output (Windows):
```text
Running: nmap -sn -PR 192.168.10.0/24 -oA outputs\20260206-1705\01-discovery
Running: nmap -sV --top-ports 1000 -iL outputs\20260206-1705\01-discovery.gnmap -oA outputs\20260206-1705\02-services
Outputs in outputs\20260206-1705
```

### Example Output: PowerShell Workflow Script
Command:
```powershell
.\recon.ps1
```
Example output (Windows):
```text
Outputs in outputs\20260206-1710
```



## Common Pitfalls
- Scanning out of scope
- Using aggressive timing on fragile systems
- Forgetting to save outputs
- Skipping validation of high-risk findings

## Best Practices
- Start small, expand carefully
- Use safe scripts before advanced ones
- Keep scan options consistent
- Communicate with stakeholders

## Reflection Questions
1. Why is a phased workflow important?
2. Which phase provides the most value for asset visibility?
3. How do you ensure results are reproducible?
4. What makes a recon report actionable?

## Next Steps
Tomorrow (Day 30) you will complete a final practical challenge.

**Congratulations on completing Day 29!** You now have a full Nmap recon workflow you can reuse.
