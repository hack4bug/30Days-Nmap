# Day 27 - Internal Network Enumeration

## Day Overview
Welcome to Day 27! Today you will learn internal network enumeration from beginner to advanced, focusing on safe, authorized, and structured discovery of hosts, services, and trust relationships inside a network.
This guide covers concepts, theory, and practical Nmap approaches for internal environments.

## Learning Objectives
By completing Day 27, you will be able to:
- Understand internal network enumeration goals and scope
- Discover live hosts efficiently
- Enumerate services and versions
- Identify common internal protocols and assets
- Map trust relationships and segmentation boundaries
- Build repeatable internal enumeration workflows

## Important Ethics Note
Only enumerate networks you own or have explicit permission to test.
Internal scans can disrupt services if done improperly.

## Concept: What Is Internal Enumeration?
Internal enumeration is the process of discovering hosts, services, and relationships inside a private network.
It focuses on visibility, asset inventory, and exposure reduction.

## Concept: Internal vs External Scanning
Internal scans are typically:
- Higher trust and less filtered
- More likely to discover sensitive services
- Able to reach management interfaces not exposed externally

## Concept: Scope and Segmentation
Internal networks are often segmented into VLANs or subnets.
Enumeration should respect boundaries and approvals for each segment.

## Theory: Common Internal Targets
Typical internal assets include:
- Workstations
- Servers and file shares
- Directory services (AD/LDAP)
- Databases and internal APIs
- Network devices (routers, switches)
- Printers and IoT devices

## Theory: Internal Threat Model
Internal enumeration helps:
- Identify weak configurations
- Discover legacy services
- Validate segmentation and firewall rules
- Improve asset inventory accuracy

## Internal Enumeration Workflow
A safe internal workflow typically includes:
1. Scope definition and approvals
2. Host discovery
3. Service identification
4. Targeted enumeration by protocol
5. Relationship mapping
6. Reporting and remediation planning

## Step 1: Scope Definition
Define:
- Subnets and VLANs
- Allowed scan times
- Approved scan types
- Contacts for alerts

## Step 2: Host Discovery Basics
Use ping sweeps to find live hosts:
```bash
nmap -sn 192.168.1.0/24 -oA outputs/host-discovery
```

## Host Discovery Techniques
Common discovery probes:
- ICMP echo
- TCP SYN to common ports
- ARP requests on local subnet

## ARP Discovery on Local LAN
ARP discovery is fast and reliable on local networks:
```bash
sudo nmap -sn -PR 192.168.1.0/24 -oA outputs/arp-discovery
```

## TCP-Based Host Discovery
If ICMP is blocked, use TCP probes:
```bash
nmap -sn -PS22,80,443,445,3389 10.0.0.0/24 -oA outputs/tcp-discovery
```

## Step 3: Service Identification
Once live hosts are identified, scan for open ports and services:
```bash
nmap -sV -iL outputs/host-discovery.gnmap -oA outputs/service-scan
```

## Port Scanning Strategies
Choose based on scope and time:
- `-F` for top 100 ports
- `--top-ports 1000` for common services
- `-p 1-1000` for defined range
- `-p-` for full scan (slow)

## Example: Balanced Internal Scan
```bash
nmap -sV --top-ports 1000 -T3 -iL outputs/live.txt -oA outputs/balanced
```

## Step 4: Protocol-Specific Enumeration
Focus on protocols commonly found internally:
- SMB/NetBIOS
- LDAP/Active Directory
- DNS
- SNMP
- RDP/SSH
- HTTP/HTTPS
- Database services

## SMB Enumeration
```bash
nmap -p 445 --script smb-os-discovery,smb-enum-shares,smb-enum-users target
```
Look for:
- OS and domain info
- Share permissions
- Anonymous access

## LDAP Enumeration (Safe Checks)
```bash
nmap -p 389 --script ldap-rootdse target
```
This can reveal directory structure and capabilities.

## DNS Enumeration
```bash
nmap -p 53 --script dns-recursion,dns-service-discovery target
```
Check for:
- Open recursion
- Internal service discovery records

## SNMP Enumeration
```bash
nmap -sU -p 161 --script snmp-info target
```
SNMP can reveal system details, interfaces, and uptime.

## RDP and SSH Checks
```bash
nmap -p 22,3389 --script ssh-hostkey,rdp-enum-encryption target
```

## HTTP/HTTPS Enumeration
```bash
nmap -p 80,443 --script http-title,http-headers target
```

## Database Enumeration (Internal)
```bash
nmap -p 1433,1521,3306,5432,27017,6379 -sV --script ms-sql-info,oracle-tns-version,mysql-info,pgsql-info,mongodb-info,redis-info target
```

## Step 5: Relationship Mapping
Internal enumeration should also map relationships such as:
- Domain membership
- Trust relationships
- File share dependencies
- Routing boundaries between subnets

## Network Topology Discovery
Use traceroute and scripting:
```bash
nmap --traceroute target
```

## VLAN and Subnet Awareness
Scanning across VLANs may be filtered or routed differently.
Document which segments are reachable.

## Timing and Performance
Internal scans can be faster than external, but still be careful:
- `-T3` is a safe default
- Use `--max-rate` for large scans
- Avoid overloading weak devices

## Internal Scan Examples
### Example A: Full Internal Discovery
```bash
nmap -sn 10.0.0.0/24 -oA outputs/internal-discovery
```

### Example B: Service Scan of Live Hosts
```bash
awk '/Up$/{print $2}' outputs/internal-discovery.gnmap > outputs/live.txt
nmap -sV -iL outputs/live.txt -oA outputs/internal-services
```

### Example C: SMB Focus
```bash
nmap -p 445 --script smb-os-discovery,smb-enum-shares -iL outputs/live.txt -oA outputs/smb-scan
```

### Example D: Internal Web Services
```bash
nmap -p 80,443,8080,8443 -sV --script http-title,http-headers -iL outputs/live.txt -oA outputs/internal-web
```

## Internal Asset Classes
### Workstations
- Often run SMB, RDP, and local management services
- Identify OS versions and patch levels

### Servers
- Host databases, directory services, and internal apps
- Require careful enumeration to avoid disruption

### Network Devices
- Routers, switches, firewalls often expose SNMP or web UI
- Use safe checks only

### Printers and IoT
- Often expose HTTP, SNMP, and proprietary services
- Be cautious, as they can be fragile

## Advanced Concepts
### Credentialed Enumeration
Some internal enumeration is more accurate with credentials.
Nmap supports limited authenticated scripts; use carefully and securely.

### Passively Correlating Data
Combine scan results with DHCP logs, AD records, or asset inventory to improve accuracy.

### Lateral Movement Risk
Enumeration can reveal paths for lateral movement if misconfigurations exist.

### Network Segmentation Validation
Use results to confirm that sensitive segments are isolated.

## Common Misconfigurations
- SMB shares with anonymous access
- Legacy SMBv1 enabled
- Open SNMP with default community strings
- Databases exposed without authentication
- RDP exposed broadly to workstation subnets

## Safe Script Categories
Use `safe` scripts for internal enumeration where possible:
```bash
nmap -p 445 --script safe target
```

## Risky Scripts Warning
Avoid intrusive or brute-force scripts unless explicitly approved.

## Logging and Reporting
Always record:
- Scan date and time
- Target ranges
- Scan options and scripts used
- Output file paths

## Command Output Examples (Linux and Windows)
The outputs below are **example outputs** to show what Nmap results look like on Linux and Windows.
Actual results will differ based on targets, permissions, and network conditions.

### Linux Example: Host Discovery (-sn)
Command:
```bash
nmap -sn 192.168.1.0/24
```
Example output:
```text
Starting Nmap 7.94 ( https://nmap.org ) at 2026-02-06 14:00 UTC
Nmap scan report for 192.168.1.1
Host is up (0.0020s latency).
Nmap scan report for 192.168.1.10
Host is up (0.0015s latency).
Nmap scan report for 192.168.1.20
Host is up (0.0031s latency).
Nmap done: 256 IP addresses (3 hosts up) scanned in 4.12 seconds
```

### Linux Example: Service Scan (-sV)
Command:
```bash
nmap -sV 192.168.1.10
```
Example output:
```text
Starting Nmap 7.94 ( https://nmap.org ) at 2026-02-06 14:05 UTC
Nmap scan report for 192.168.1.10
Host is up (0.0010s latency).
Not shown: 997 closed tcp ports (conn-refused)
PORT    STATE SERVICE  VERSION
22/tcp  open  ssh      OpenSSH 8.2p1 Ubuntu 4ubuntu0.7 (Ubuntu Linux; protocol 2.0)
80/tcp  open  http     Apache httpd 2.4.41 ((Ubuntu))
445/tcp open  microsoft-ds Windows Server 2019 Standard 17763 microsoft-ds
Service Info: OSs: Linux, Windows; CPE: cpe:/o:linux:linux_kernel
Nmap done: 1 IP address (1 host up) scanned in 7.32 seconds
```

### Linux Example: SMB Enumeration
Command:
```bash
nmap -p 445 --script smb-os-discovery,smb-enum-shares 192.168.1.20
```
Example output:
```text
Starting Nmap 7.94 ( https://nmap.org ) at 2026-02-06 14:10 UTC
Nmap scan report for 192.168.1.20
Host is up (0.0024s latency).
PORT    STATE SERVICE
445/tcp open  microsoft-ds
| smb-os-discovery:
|   OS: Windows Server 2019 Standard 17763 (Windows Server 2019 Standard 6.3)
|   Computer name: FILESERVER
|   NetBIOS computer name: FILESERVER 
|   Domain name: CORP
|   FQDN: FILESERVER.corp.local
|_  System time: 2026-02-06T14:10:32+00:00
| smb-enum-shares:
|   ADMIN$:
|     Type: Disk
|     Comment: Remote Admin
|   C$:
|     Type: Disk
|     Comment: Default share
|   Public:
|     Type: Disk
|_    Comment: Public Share
Nmap done: 1 IP address (1 host up) scanned in 9.01 seconds
```

### Windows Example: Host Discovery (-sn)
Command (PowerShell or CMD):
```cmd
nmap -sn 192.168.1.0/24
```
Example output:
```text
Starting Nmap 7.94 ( https://nmap.org ) at 2026-02-06 14:20 UTC
Nmap scan report for 192.168.1.1
Host is up (0.0030s latency).
Nmap scan report for 192.168.1.30
Host is up (0.0042s latency).
Nmap done: 256 IP addresses (2 hosts up) scanned in 5.10 seconds
```

### Windows Example: Service Scan (-sV)
Command:
```cmd
nmap -sV 192.168.1.30
```
Example output:
```text
Starting Nmap 7.94 ( https://nmap.org ) at 2026-02-06 14:25 UTC
Nmap scan report for 192.168.1.30
Host is up (0.0020s latency).
Not shown: 998 closed tcp ports (conn-refused)
PORT     STATE SERVICE      VERSION
135/tcp  open  msrpc        Microsoft Windows RPC
139/tcp  open  netbios-ssn  Microsoft Windows netbios-ssn
445/tcp  open  microsoft-ds Windows Server 2016 Standard 14393 microsoft-ds
3389/tcp open  ms-wbt-server Microsoft Terminal Services
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows
Nmap done: 1 IP address (1 host up) scanned in 8.40 seconds
```

### Windows Example: SMB Security Mode
Command:
```cmd
nmap -p 445 --script smb-security-mode,smb2-security-mode 192.168.1.30
```
Example output:
```text
Starting Nmap 7.94 ( https://nmap.org ) at 2026-02-06 14:30 UTC
Nmap scan report for 192.168.1.30
Host is up (0.0031s latency).
PORT    STATE SERVICE
445/tcp open  microsoft-ds
| smb-security-mode:
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-security-mode:
|   3.1.1:
|_    Message signing enabled but not required
Nmap done: 1 IP address (1 host up) scanned in 7.85 seconds
```

### Windows Example: DNS Service Check
Command:
```cmd
nmap -p 53 --script dns-recursion 192.168.1.2
```
Example output:
```text
Starting Nmap 7.94 ( https://nmap.org ) at 2026-02-06 14:35 UTC
Nmap scan report for 192.168.1.2
Host is up (0.0022s latency).
PORT   STATE SERVICE
53/tcp open  domain
| dns-recursion:
|   Recursion appears to be enabled
|_  WARNING: Recursion may be available to untrusted clients
Nmap done: 1 IP address (1 host up) scanned in 2.20 seconds
```



## Output Management
Use `-oA` for consistent outputs:
```bash
nmap -sV -oA outputs/internal-scan 10.0.0.0/24
```

## Parsing Results
Extract open ports:
```bash
grep "/open/" outputs/internal-scan.gnmap
```

## Change Detection
Use `ndiff` on XML outputs to detect changes:
```bash
ndiff outputs/internal-old.xml outputs/internal-new.xml > outputs/internal-diff.txt
```

## Internal Enumeration and Vulnerability Management
Enumeration provides the asset and service inventory needed for patching and risk management.

## Advanced Internal Enumeration Commands (Nmap + Scripts)
Below are advanced, **authorized** internal enumeration commands that use NSE scripts and safe automation patterns.
Use them only when scope and permissions are clear.

### 1. SMB Deep Enumeration Bundle
```bash
nmap -p 445 --script "smb-os-discovery,smb-enum-shares,smb-enum-users,smb-security-mode,smb2-security-mode,smb2-time" -oA outputs/smb-deep target
```

### 2. Active Directory and LDAP Safe Checks
```bash
nmap -p 389,636 --script "ldap-rootdse,ldap-search" --script-args "ldap.username=,ldap.password=,ldap.qfilter=(objectClass=*)" -oA outputs/ldap-safe target
```
Note: Only provide credentials if explicitly authorized. Empty credentials run anonymous checks where allowed.

### 3. DNS Enumeration and Zone Transfer Check
```bash
nmap -p 53 --script "dns-recursion,dns-service-discovery,dns-zone-transfer" --script-args "dns-zone-transfer.domain=corp.local" -oA outputs/dns-enum target
```
Zone transfers must be authorized; most servers should block them.

### 4. SNMP Enumeration with Common Community Strings
```bash
nmap -sU -p 161 --script "snmp-info,snmp-interfaces,snmp-sysdescr" --script-args "snmpcommunity=public" -oA outputs/snmp-enum target
```
Only test communities you are permitted to use.

### 5. RDP Security Enumeration
```bash
nmap -p 3389 --script "rdp-enum-encryption,rdp-ntlm-info" -oA outputs/rdp-enum target
```

### 6. SSH Hardening Checks
```bash
nmap -p 22 --script "ssh-hostkey,ssh2-enum-algos" -oA outputs/ssh-enum target
```

### 7. HTTP/HTTPS Internal App Discovery
```bash
nmap -p 80,443,8080,8443 -sV --script "http-title,http-headers,http-methods,http-enum" -oA outputs/http-enum target
```

### 8. TLS and Certificate Enumeration
```bash
nmap -p 443,8443 --script "ssl-cert,ssl-enum-ciphers,ssl-dh-params" -oA outputs/tls-enum target
```

### 9. Database Enumeration Bundle (Internal)
```bash
nmap -sV -p 1433,1521,3306,5432,27017,6379 --script "ms-sql-info,oracle-tns-version,mysql-info,pgsql-info,mongodb-info,redis-info" -oA outputs/db-enum target
```

### 10. Network Device Enumeration (SNMP + HTTP)
```bash
nmap -sU -p 161 --script snmp-info -oA outputs/netdev-snmp target
nmap -p 80,443 --script http-title,http-headers -oA outputs/netdev-http target
```

### 11. Nmap Scripting Category Sweep (Safe)
```bash
nmap -sV --script safe -iL outputs/live.txt -oA outputs/safe-scripts
```

### 12. Host Discovery + Service Enumeration Pipeline
```bash
nmap -sn -PR 10.0.0.0/24 -oA outputs/lan-discovery
awk '/Up$/{print $2}' outputs/lan-discovery.gnmap > outputs/lan-live.txt
nmap -sV --top-ports 1000 -iL outputs/lan-live.txt -oA outputs/lan-services
```

### 13. Segmentation Validation Scan (Two Subnets)
```bash
nmap -sV -p 22,80,443,445,3389 10.0.10.0/24 -oA outputs/seg-a
nmap -sV -p 22,80,443,445,3389 10.0.20.0/24 -oA outputs/seg-b
ndiff outputs/seg-a.xml outputs/seg-b.xml > outputs/seg-diff.txt
```

### 14. Randomized Host Order for Large Internal Sweeps
```bash
nmap -sV --randomize-hosts -T3 --max-rate 200 -iL targets.txt -oA outputs/randomized-scan
```

### 15. Safe UDP Spot Checks for Common Services
```bash
nmap -sU -p 53,67,68,69,123,161,162,500,514 -sV -oA outputs/udp-spot
```

### 16. Internal Web Virtual Host Checks (Host Header)
```bash
nmap -p 80 --script http-title --script-args http.host=app.internal.local 10.0.0.50 -oA outputs/vhost-check
```

### 17. SMB Signing and SMBv1 Detection
```bash
nmap -p 445 --script "smb-security-mode,smb2-security-mode,smb-protocols" -oA outputs/smb-signing target
```

### 18. Kerberos Service Enumeration (Safe)
```bash
nmap -p 88 --script krb5-enum-users --script-args krb5-enum-users.realm=CORP.LOCAL -oA outputs/krb-enum target
```
Only use if explicitly approved, as user enumeration can be sensitive.

### 19. Passive Discovery with Targets Sniffer (Lab Only)
```bash
sudo nmap --script targets-sniffer -oA outputs/targets-sniffer
```

### 20. Full Internal Profile (Approved Only)
```bash
nmap -sV -T3 --max-rate 300 -p- -iL outputs/lan-live.txt -oA outputs/internal-full
```
Use only in small, approved scopes due to high impact.



## Best Practices
- Start with discovery, then scan services
- Use conservative timing on sensitive networks
- Validate results with inventory data
- Document and report findings clearly

## Reflection Questions
1. Why is internal enumeration critical for security?
2. Which protocols provide the most useful internal visibility?
3. How does segmentation affect enumeration results?
4. What steps ensure safe internal scanning?

## Next Steps
Tomorrow (Day 28) you will learn about real-world recon methodology.

**Congratulations on completing Day 27!** You now have a comprehensive internal network enumeration guide.
