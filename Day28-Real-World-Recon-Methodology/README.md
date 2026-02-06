# Day 28 - Real-World Recon Methodology

## Day Overview
Welcome to Day 28! Today you will learn a real-world reconnaissance methodology that starts with safe basics and scales to advanced scanning techniques.
This guide focuses on practical workflows, decision points, and commands you can use in authorized environments.

## Learning Objectives
By completing Day 28, you will be able to:
- Build a recon plan aligned with scope and permissions
- Use a phased approach from discovery to deep enumeration
- Choose safe vs advanced scan options responsibly
- Manage outputs for analysis and reporting
- Apply recon methodology to real engagements

## Important Ethics Note
Reconnaissance must only be performed with explicit permission.
Always follow rules of engagement and document all actions.

## Concept: What Is Reconnaissance?
Recon is the process of collecting information about systems and networks to understand exposure and risk.
It combines passive discovery, active scanning, and careful analysis.

## Concept: Recon vs Enumeration
- Recon focuses on discovering what exists
- Enumeration focuses on details about what you found

## Concept: Real-World Recon Constraints
- Limited time windows
- Shared infrastructure and cloud services
- Monitoring and alerting systems
- Strict scope boundaries

## Recon Phases (High-Level)
1. Planning and scoping
2. Passive discovery
3. Active discovery
4. Targeted enumeration
5. Validation and reporting

## Phase 1: Planning and Scoping
Define:
- In-scope targets (IPs, domains, subnets)
- Out-of-scope targets
- Allowed scan types and time windows
- Notification contacts

## Phase 2: Passive Discovery
Gather information without direct probing when possible.
Examples:
- DNS records
- Certificate transparency logs
- Public asset inventories

## Phase 3: Active Discovery (Basic)
Start with low-noise discovery scans.
```bash
nmap -sn 10.0.0.0/24 -oA outputs/discovery
```

## Phase 3: Active Discovery (Advanced)
Use TCP probes if ICMP is blocked:
```bash
nmap -sn -PS22,80,443,445,3389 10.0.0.0/24 -oA outputs/tcp-discovery
```

## Phase 4: Targeted Enumeration (Basic)
Perform service scans on live hosts:
```bash
awk '/Up$/{print $2}' outputs/discovery.gnmap > outputs/live.txt
nmap -sV -iL outputs/live.txt -oA outputs/services
```

## Phase 4: Targeted Enumeration (Advanced)
Use script bundles for deeper insight:
```bash
nmap -sV --script safe -iL outputs/live.txt -oA outputs/safe-bundle
```

## Phase 5: Validation and Reporting
Validate key findings with manual checks and prepare a report.

## Real-World Recon Workflow (Step-by-Step)
### Step 1: Confirm Scope
Ensure you have written approval for all targets.

### Step 2: Build Target Inventory
Combine DNS, IP ranges, and asset lists into a unified inventory.

### Step 3: Discovery Scan
```bash
nmap -sn 192.168.10.0/24 -oA outputs/step1-discovery
```

### Step 4: Service Scan
```bash
awk '/Up$/{print $2}' outputs/step1-discovery.gnmap > outputs/step1-live.txt
nmap -sV --top-ports 1000 -iL outputs/step1-live.txt -oA outputs/step2-services
```

### Step 5: Web Enumeration
```bash
nmap -p 80,443,8080,8443 -sV --script http-title,http-headers -iL outputs/step1-live.txt -oA outputs/step3-web
```

### Step 6: SMB and AD Checks
```bash
nmap -p 445 --script smb-os-discovery,smb-enum-shares -iL outputs/step1-live.txt -oA outputs/step4-smb
nmap -p 389 --script ldap-rootdse -iL outputs/step1-live.txt -oA outputs/step4-ldap
```

### Step 7: Database and Service Enumeration
```bash
nmap -p 1433,1521,3306,5432,27017,6379 -sV --script ms-sql-info,oracle-tns-version,mysql-info,pgsql-info,mongodb-info,redis-info -iL outputs/step1-live.txt -oA outputs/step5-db
```

### Step 8: TLS and Security Checks
```bash
nmap -p 443,8443 --script ssl-cert,ssl-enum-ciphers -iL outputs/step1-live.txt -oA outputs/step6-tls
```

### Step 9: Validate Results
Use `curl` or browser checks for important services.

### Step 10: Report Findings
Summarize exposures, misconfigurations, and remediation steps.

## Basic Recon Commands
```bash
nmap -sn 10.0.0.0/24
nmap -sV 10.0.0.5
nmap -p 80,443 --script http-title 10.0.0.5
```

## Advanced Recon Commands
```bash
nmap -sn -PS22,80,443,445,3389 10.0.0.0/24
nmap -sV --top-ports 1000 -iL outputs/live.txt
nmap -p 445 --script smb-os-discovery,smb-enum-shares target
nmap -p 389 --script ldap-rootdse target
nmap -p 443 --script ssl-cert,ssl-enum-ciphers target
nmap -sV --script safe -iL outputs/live.txt
```

## Real-World Recon: Basic to Advanced (Commands, Flags, Scripts)
This section provides practical commands, flags, and scripts, plus automation examples in Bash, Python, and PowerShell.
Each example includes a realistic **sample output** for both Linux and Windows.

### 1. Basic Discovery Scan (ICMP + ARP on Local LAN)
Command:
```bash
nmap -sn -PR 192.168.1.0/24
```
Linux example output:
```text
Starting Nmap 7.94 ( https://nmap.org ) at 2026-02-06 16:00 UTC
Nmap scan report for 192.168.1.1
Host is up (0.0010s latency).
MAC Address: 00:11:22:33:44:55 (Vendor)
Nmap scan report for 192.168.1.20
Host is up (0.0018s latency).
MAC Address: AA:BB:CC:DD:EE:FF (Vendor)
Nmap done: 256 IP addresses (2 hosts up) scanned in 3.40 seconds
```
Windows example output:
```text
Starting Nmap 7.94 ( https://nmap.org ) at 2026-02-06 16:00 UTC
Nmap scan report for 192.168.1.1
Host is up (0.0023s latency).
Nmap scan report for 192.168.1.20
Host is up (0.0032s latency).
Nmap done: 256 IP addresses (2 hosts up) scanned in 4.10 seconds
```

### 2. Basic Service Scan (Top Ports + Version Detection)
Command:
```bash
nmap -sV --top-ports 100 192.168.1.20
```
Linux example output:
```text
Starting Nmap 7.94 ( https://nmap.org ) at 2026-02-06 16:05 UTC
Nmap scan report for 192.168.1.20
Host is up (0.0012s latency).
PORT    STATE SERVICE VERSION
22/tcp  open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.7
80/tcp  open  http    Apache httpd 2.4.41 ((Ubuntu))
443/tcp open  https   Apache httpd 2.4.41
Nmap done: 1 IP address (1 host up) scanned in 6.80 seconds
```
Windows example output:
```text
Starting Nmap 7.94 ( https://nmap.org ) at 2026-02-06 16:05 UTC
Nmap scan report for 192.168.1.20
Host is up (0.0020s latency).
PORT     STATE SERVICE      VERSION
135/tcp  open  msrpc        Microsoft Windows RPC
445/tcp  open  microsoft-ds Windows Server 2019 Standard 17763
3389/tcp open  ms-wbt-server Microsoft Terminal Services
Nmap done: 1 IP address (1 host up) scanned in 7.50 seconds
```

### 3. Advanced Safe Script Bundle
Command:
```bash
nmap -sV --script safe -iL outputs/live.txt -oA outputs/safe-bundle
```
Linux example output (snippet):
```text
PORT    STATE SERVICE VERSION
22/tcp  open  ssh     OpenSSH 8.2p1
| ssh-hostkey:
|   2048 SHA256:... (RSA)
80/tcp  open  http    Apache httpd 2.4.41
| http-title: Internal Portal
|_Requested resource was /
```
Windows example output (snippet):
```text
PORT    STATE SERVICE
445/tcp open  microsoft-ds
| smb-os-discovery:
|   OS: Windows Server 2016 Standard 14393
|_  Computer name: FILE01
```

### 4. SMB Enumeration (Advanced)
Command:
```bash
nmap -p 445 --script "smb-os-discovery,smb-enum-shares,smb-enum-users,smb-security-mode" 192.168.1.30
```
Linux example output (snippet):
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
Windows example output (snippet):
```text
PORT    STATE SERVICE
445/tcp open  microsoft-ds
| smb-security-mode:
|_  message_signing: enabled but not required
```

### 5. LDAP RootDSE (Directory Discovery)
Command:
```bash
nmap -p 389 --script ldap-rootdse 192.168.1.10
```
Linux example output (snippet):
```text
PORT    STATE SERVICE
389/tcp open  ldap
| ldap-rootdse:
|   supportedLDAPVersion: 3
|_  namingContexts: DC=corp,DC=local
```
Windows example output (snippet):
```text
PORT    STATE SERVICE
389/tcp open  ldap
| ldap-rootdse:
|_  defaultNamingContext: DC=corp,DC=local
```

### 6. Web + TLS Enumeration (Internal Web Apps)
Command:
```bash
nmap -p 80,443 --script http-title,http-headers,ssl-cert,ssl-enum-ciphers 192.168.1.50
```
Linux example output (snippet):
```text
80/tcp  open  http
| http-title: Intranet
|_Requested resource was /
443/tcp open  https
| ssl-cert: Subject: CN=intranet.corp.local
|_Not valid after: 2026-12-31
```
Windows example output (snippet):
```text
443/tcp open  https
| ssl-enum-ciphers:
|   TLSv1.2:
|_    ciphers: ...
```

### 7. UDP Spot Checks (Core Infra)
Command:
```bash
nmap -sU -p 53,67,68,69,123,161,162,500,514 -sV 192.168.1.1
```
Linux example output (snippet):
```text
53/udp  open  domain
123/udp open  ntp
161/udp open  snmp
```
Windows example output (snippet):
```text
53/udp  open  domain
123/udp open  ntp
```

### 8. Advanced Bash Automation (Discovery -> Service -> Web)
Bash script:
```bash
#!/usr/bin/env bash
set -euo pipefail
RANGE="${1:-192.168.1.0/24}"
OUTDIR="outputs/$(date +%Y%m%d-%H%M%S)"
mkdir -p "$OUTDIR"
nmap -sn -PR "$RANGE" -oA "$OUTDIR/discovery"
awk '/Up$/{print $2}' "$OUTDIR/discovery.gnmap" > "$OUTDIR/live.txt"
nmap -sV --top-ports 1000 -iL "$OUTDIR/live.txt" -oA "$OUTDIR/services"
nmap -p 80,443,8080,8443 --script http-title,http-headers -iL "$OUTDIR/live.txt" -oA "$OUTDIR/web"
echo "Outputs in $OUTDIR"
```
Linux example output (script run):
```text
Outputs in outputs/20260206-1600
```
Windows example output:
```text
This Bash script is intended for Linux/macOS environments.
```

### 9. Python Automation (Cross-Platform)
Python script:
```python
import subprocess
from datetime import datetime
from pathlib import Path

range_cidr = "192.168.1.0/24"
outdir = Path("outputs") / datetime.now().strftime("%Y%m%d-%H%M%S")
outdir.mkdir(parents=True, exist_ok=True)

def run(cmd):
    print("Running:", " ".join(cmd))
    subprocess.run(cmd, check=True)

run(["nmap", "-sn", range_cidr, "-oA", str(outdir / "discovery")])
run(["nmap", "-sV", "--top-ports", "1000", "-iL", str(outdir / "discovery.gnmap"), "-oA", str(outdir / "services")])
run(["nmap", "-p", "80,443,8080,8443", "--script", "http-title,http-headers", "-iL", str(outdir / "discovery.gnmap"), "-oA", str(outdir / "web")])
print("Outputs in", outdir)
```
Linux example output:
```text
Running: nmap -sn 192.168.1.0/24 -oA outputs/20260206-1605/discovery
Running: nmap -sV --top-ports 1000 -iL outputs/20260206-1605/discovery.gnmap -oA outputs/20260206-1605/services
Running: nmap -p 80,443,8080,8443 --script http-title,http-headers -iL outputs/20260206-1605/discovery.gnmap -oA outputs/20260206-1605/web
Outputs in outputs/20260206-1605
```
Windows example output:
```text
Running: nmap -sn 192.168.1.0/24 -oA outputs\20260206-1605\discovery
Running: nmap -sV --top-ports 1000 -iL outputs\20260206-1605\discovery.gnmap -oA outputs\20260206-1605\services
Running: nmap -p 80,443,8080,8443 --script http-title,http-headers -iL outputs\20260206-1605\discovery.gnmap -oA outputs\20260206-1605\web
Outputs in outputs\20260206-1605
```

### 10. PowerShell Automation (Windows)
PowerShell script:
```powershell
$Range = "192.168.1.0/24"
$OutDir = "outputs\" + (Get-Date -Format "yyyyMMdd-HHmmss")
New-Item -ItemType Directory -Path $OutDir | Out-Null
nmap -sn $Range -oA "$OutDir\discovery"
Get-Content "$OutDir\discovery.gnmap" | ForEach-Object { if ($_ -match "Up$") { ($_ -split " ")[1] } } | Set-Content "$OutDir\live.txt"
nmap -sV --top-ports 1000 -iL "$OutDir\live.txt" -oA "$OutDir\services"
nmap -p 80,443,8080,8443 --script http-title,http-headers -iL "$OutDir\live.txt" -oA "$OutDir\web"
Write-Host "Outputs in $OutDir"
```
Windows example output:
```text
Outputs in outputs\20260206-1610
```
Linux example output:
```text
This PowerShell script is intended for Windows environments.
```

### 11. Advanced Flags Summary (When Approved)
Common advanced flags used in real-world recon:
- `-sS` SYN scan for speed and stealth on internal nets
- `-T2` or `-T3` for controlled timing
- `--max-rate` to reduce burst traffic
- `--randomize-hosts` to avoid sequential patterns
- `-Pn` for networks that block ICMP
- `-n` to skip DNS resolution



## Decision Points in Real Recon
- When to stop discovery and move to enumeration
- When to reduce port scope to protect stability
- When to use advanced scripts (only with permission)
- When to validate findings manually

## Command Output Examples (Linux, Windows, macOS)
The outputs below are **example outputs** to show how Nmap results look across platforms.
Actual results depend on targets and network conditions.

### Linux Example: Discovery Scan
Command:
```bash
nmap -sn 10.0.0.0/24
```
Example output:
```text
Starting Nmap 7.94 ( https://nmap.org ) at 2026-02-06 15:00 UTC
Nmap scan report for 10.0.0.1
Host is up (0.0018s latency).
Nmap scan report for 10.0.0.10
Host is up (0.0025s latency).
Nmap done: 256 IP addresses (2 hosts up) scanned in 3.90 seconds
```

### Linux Example: Service Scan
Command:
```bash
nmap -sV 10.0.0.10
```
Example output:
```text
Starting Nmap 7.94 ( https://nmap.org ) at 2026-02-06 15:05 UTC
Nmap scan report for 10.0.0.10
Host is up (0.0012s latency).
Not shown: 997 closed tcp ports (conn-refused)
PORT    STATE SERVICE VERSION
22/tcp  open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.7 (Ubuntu Linux; protocol 2.0)
80/tcp  open  http    Apache httpd 2.4.41 ((Ubuntu))
Nmap done: 1 IP address (1 host up) scanned in 6.90 seconds
```

### Windows Example: Discovery Scan
Command (PowerShell or CMD):
```cmd
nmap -sn 10.0.0.0/24
```
Example output:
```text
Starting Nmap 7.94 ( https://nmap.org ) at 2026-02-06 15:10 UTC
Nmap scan report for 10.0.0.5
Host is up (0.0031s latency).
Nmap scan report for 10.0.0.20
Host is up (0.0040s latency).
Nmap done: 256 IP addresses (2 hosts up) scanned in 4.80 seconds
```

### Windows Example: SMB Enumeration
Command:
```cmd
nmap -p 445 --script smb-os-discovery,smb-enum-shares 10.0.0.20
```
Example output:
```text
Starting Nmap 7.94 ( https://nmap.org ) at 2026-02-06 15:15 UTC
Nmap scan report for 10.0.0.20
Host is up (0.0028s latency).
PORT    STATE SERVICE
445/tcp open  microsoft-ds
| smb-os-discovery:
|   OS: Windows Server 2019 Standard 17763 (Windows Server 2019 Standard 6.3)
|   Computer name: FILE01
|   Domain name: CORP
|_  System time: 2026-02-06T15:15:12+00:00
| smb-enum-shares:
|   Public:
|     Type: Disk
|_    Comment: Public Share
Nmap done: 1 IP address (1 host up) scanned in 8.12 seconds
```

### macOS Example: Discovery Scan
Command:
```bash
nmap -sn 10.0.0.0/24
```
Example output:
```text
Starting Nmap 7.94 ( https://nmap.org ) at 2026-02-06 15:20 UTC
Nmap scan report for 10.0.0.2
Host is up (0.0021s latency).
Nmap scan report for 10.0.0.12
Host is up (0.0029s latency).
Nmap done: 256 IP addresses (2 hosts up) scanned in 4.20 seconds
```

### macOS Example: Web Enumeration
Command:
```bash
nmap -p 80,443 --script http-title,http-headers 10.0.0.12
```
Example output:
```text
Starting Nmap 7.94 ( https://nmap.org ) at 2026-02-06 15:25 UTC
Nmap scan report for 10.0.0.12
Host is up (0.0019s latency).
PORT    STATE SERVICE
80/tcp  open  http
| http-title: Intranet Portal
|_Requested resource was /
| http-headers:
|   Server: nginx
|   Content-Type: text/html
|_  (Request type: GET)
Nmap done: 1 IP address (1 host up) scanned in 5.70 seconds
```



## Managing Outputs
Always use `-oA` and date-based folders to keep results organized.

## Common Pitfalls
- Scanning out of scope
- Using aggressive timing on fragile systems
- Failing to document options and outputs
- Ignoring DNS and virtual host context

## Best Practices
- Start small, expand carefully
- Use safe scripts before advanced ones
- Keep scan options consistent for comparisons
- Communicate with stakeholders

## Reflection Questions
1. Why is a phased recon approach safer?
2. Which recon phase provides the most value?
3. How do you manage scope changes during recon?
4. What makes a recon report useful for remediation?

## Next Steps
Tomorrow (Day 29) you will learn about building a full Nmap recon workflow.

**Congratulations on completing Day 28!** You now have a real-world recon methodology.
