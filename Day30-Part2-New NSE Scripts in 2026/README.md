# Day 30 Part 2 - New NSE Scripts in 2026

## Day Overview
Day 30 Part 2 focuses on modern NSE usage patterns that matter most in 2026 environments.
This module is designed as a practical upgrade to your previous workflow with deeper detection and better cloud coverage.

## Learning Objectives
By the end of this section, you will be able to:
- Run focused CVE detection workflows with NSE
- Use brute-force scripts in a controlled and auditable way
- Improve service fingerprinting with stronger probe strategy
- Adapt Nmap + NSE for cloud-native infrastructure

## Important Ethics and Safety Note
- Only scan systems you own or are explicitly authorized to test
- Treat brute-force scripts as high-impact actions and get written approval
- Start with safe scripts, then move to intrusive checks only when allowed
- Keep logs for every command and output file

## Latest Focus Areas for 2026
This module adds detailed coverage for:
1. Focus on CVE Detection
2. Active Brute Force Capabilities
3. Improved Service Probes
4. Cloud-Native Scanning

## 1) Focus on CVE Detection

### Why This Matters
In 2026, patch cycles are faster but attack windows are still large due to:
- Exposed internet services
- Legacy internal systems
- Misconfigured cloud workloads

NSE helps you quickly map service versions and test known vulnerability patterns.

### CVE Detection Workflow (Detailed)
1. Build accurate service and version visibility
2. Run broad vulnerability checks
3. Run targeted CVE scripts for high-risk services
4. Prioritize by severity and exploitability
5. Validate findings before reporting

### Step A: Build Service Baseline
```bash
nmap -sV -O --version-all -iL targets.txt -oA outputs/01-service-baseline
```

### Step B: Run Broad Vulnerability Scripts
```bash
nmap -sV --script vuln -iL targets.txt -oA outputs/02-vuln-broad
```

### Step C: Target Critical CVE Families
```bash
nmap -p 80,443,445 --script "http-vuln-*,smb-vuln-*,ssl-heartbleed" -iL targets.txt -oA outputs/03-vuln-targeted
```

### Step D: Add CVE Enrichment (If Available)
```bash
nmap -sV --script vulners --script-args mincvss=7.0 -iL targets.txt -oA outputs/04-vulners
```

### CVE Triage Guidance
- High: confirmed vulnerable service + reachable from attacker path
- Medium: likely vulnerable but version/patch ambiguity exists
- Low: informational exposure with weak exploitability

### Example Output Snippet
```text
443/tcp open  https
| ssl-heartbleed:
|   VULNERABLE:
|   The Heartbleed Bug is a serious vulnerability in OpenSSL.
|_  State: VULNERABLE
```

## 2) Active Brute Force Capabilities

### Why It Is Different
Modern NSE brute-force scripts are still powerful for password auditing, but should be run with strict controls.
Use these only when your rules of engagement explicitly allow credential testing.

### Pre-Check Controls
- Define lockout-safe user accounts
- Limit thread count and runtime
- Use known test dictionaries first
- Monitor authentication logs during scans

### SSH Brute Force (Controlled)
```bash
nmap -p 22 --script ssh-brute --script-args userdb=users.txt,passdb=passes.txt,brute.threads=4,brute.firstonly=true,brute.delay=1s target.example.com -oA outputs/10-ssh-brute
```

### FTP + HTTP Brute Force
```bash
nmap -p 21,80,443 --script ftp-brute,http-brute --script-args userdb=users.txt,passdb=passes.txt,brute.threads=4,unpwdb.timelimit=10m -iL auth-targets.txt -oA outputs/11-ftp-http-brute
```

### SMB Brute Force
```bash
nmap -p 445 --script smb-brute --script-args userdb=users.txt,passdb=passes.txt,brute.threads=3 -iL windows-targets.txt -oA outputs/12-smb-brute
```

### Brute Force Safety Flags
```bash
nmap -T2 --max-retries 2 --host-timeout 5m --script-timeout 90s ...
```

### Reporting Requirements
- Record exact credential policy scope
- Include attempt limits used
- Highlight lockout or rate-limit behavior
- Report only validated weak credentials

## 3) Improved Service Probes

### Why Probe Strategy Matters
Accurate fingerprinting reduces false positives in CVE detection and speeds up validation.
In 2026 environments, services often run on non-standard ports, behind proxies, or in containers.

### Probe Modes
Light scan (fast triage):
```bash
nmap -sV --version-light -iL targets.txt -oA outputs/20-sv-light
```

Balanced scan (default production workflow):
```bash
nmap -sV --version-intensity 7 -iL targets.txt -oA outputs/21-sv-balanced
```

Deep scan (maximum fingerprint coverage):
```bash
nmap -sV --version-all --allports -iL targets.txt -oA outputs/22-sv-deep
```

### Service Context Enrichment
```bash
nmap -sV --script banner,http-title,http-headers,ssl-cert,ssh2-enum-algos -iL targets.txt -oA outputs/23-service-enriched
```

### UDP Service Precision
```bash
nmap -sU --top-ports 200 --version-intensity 8 --script snmp-info,dns-service-discovery,ntp-info -iL infra-targets.txt -oA outputs/24-udp-enriched
```

### Practical Tuning Tips
- Use `--allports` when services may hide on uncommon ports
- Increase `--version-intensity` for high-value hosts
- Pair `-sV` with protocol scripts to confirm service identity

## 4) Cloud-Native Scanning

### Cloud-Native Targets to Include
- Public load balancers and reverse proxies
- Cloud VM external and internal interfaces
- Kubernetes control-plane and node endpoints
- Managed databases and registry endpoints

### External Cloud Perimeter Scan
```bash
nmap -Pn -sV --top-ports 1000 --script http-title,http-headers,ssl-cert -iL cloud-public.txt -oA outputs/30-cloud-perimeter
```

### Kubernetes Exposure Check
```bash
nmap -Pn -p 443,6443,10250,10257,10259,2379,2380 --script ssl-cert,http-title -iL k8s-targets.txt -oA outputs/31-k8s-surface
```

### Container and Registry Surface
```bash
nmap -Pn -p 2375,2376,5000,8080,9000 --script banner,http-title -iL container-targets.txt -oA outputs/32-container-surface
```

### Managed Database Visibility
```bash
nmap -Pn -p 1433,3306,5432,6379,27017 --script ms-sql-info,mysql-info,pgsql-info,redis-info,mongodb-info -iL cloud-db.txt -oA outputs/33-cloud-db
```

### Cloud-Specific Safety
- Respect cloud provider acceptable use policies
- Avoid aggressive timing against internet-facing assets
- Separate external scans from internal VPC scans
- Re-scan frequently because cloud assets are ephemeral

## Combined Practical Workflow (Recommended)
1. Discovery and inventory:
```bash
nmap -sn -iL scope.txt -oA outputs/00-discovery
```
2. Improved service probes:
```bash
nmap -sV --version-intensity 7 -iL live.txt -oA outputs/01-services
```
3. CVE-focused detection:
```bash
nmap -sV --script vuln,vulners -iL live.txt -oA outputs/02-cve
```
4. Approved active credential testing:
```bash
nmap -p 22,21,445 --script ssh-brute,ftp-brute,smb-brute -iL auth-scope.txt -oA outputs/03-bruteforce
```
5. Cloud-native exposure checks:
```bash
nmap -Pn -iL cloud-assets.txt -oA outputs/04-cloud
```

## Recommended Output Structure
```text
outputs/
  00-discovery.*
  01-services.*
  02-cve.*
  03-bruteforce.*
  04-cloud.*
```

## Reflection Questions
1. Which scan settings improved detection accuracy the most?
2. What controls are required before enabling brute-force scripts?
3. How did cloud-native assets change your scanning strategy?
4. Which findings are urgent versus informational?

## Final Note
Day 30 Part 2 is about depth and precision: detect known vulnerabilities faster, test credentials responsibly, fingerprint services accurately, and handle cloud infrastructure safely.
