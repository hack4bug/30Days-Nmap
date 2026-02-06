# Day 26 - Cloud and External Perimeter Scanning

## Day Overview
Today you will learn how to scan cloud and external perimeters using Nmap in a safe, authorized, and structured way.
The focus is on real-world concepts, theory, and practical scanning approaches that are appropriate for external-facing assets.

## Learning Objectives
By completing Day 26, you will be able to:
- Understand what an external perimeter is and how it changes scanning strategy
- Identify cloud-specific considerations (shared IPs, CDNs, provider rules)
- Build a safe external scanning workflow from discovery to reporting
- Use Nmap options that reduce risk and false positives
- Interpret results in the context of cloud infrastructure

## Ethics and Authorization
External scanning requires explicit permission and a clearly defined scope.
Always get written approval and confirm allowed target ranges before scanning.

## Concept First: What Is an External Perimeter?
The external perimeter is the set of internet-facing assets that can be reached from outside the organization.
It includes:
- Public IP ranges owned or leased by the organization
- DNS records that resolve to public services
- Cloud services exposed to the internet
- Third-party hosted assets under your responsibility

## Concept: Why External Scanning Is Different
External scanning has different risks and constraints compared to internal scanning:
- Lower trust networks and more monitoring by ISPs and providers
- Shared infrastructure like CDNs and cloud load balancers
- Strict provider policies on scanning activity
- Higher likelihood of rate limiting and blocking

## Concept: Cloud Ownership vs Exposure
Cloud environments make ownership and exposure complex:
- One IP can serve multiple customers (shared services)
- Assets can change rapidly due to autoscaling
- DNS records can point to managed services rather than VMs

## Concept: DNS as the External Inventory
DNS is a primary inventory source for external assets.
Public DNS records reveal hostnames and service endpoints.

## Concept: The Perimeter Is a Moving Target
Cloud infrastructure changes quickly:
- New instances appear and disappear
- Load balancers rotate IPs
- CDN endpoints are shared and dynamic

## Theory: External Recon Workflow
A safe external scanning workflow generally follows these stages:
1. Define scope and rules of engagement
2. Collect DNS and IP inventory
3. Validate ownership and cloud provider constraints
4. Perform low-noise discovery
5. Run targeted service scans
6. Analyze results and document findings

## Stage 1: Define Scope and Permissions
Key items to define before scanning:
- Approved IP ranges and domains
- Time windows for scanning
- Port ranges and scan types allowed
- Contact points for alerts

## Stage 2: Collect DNS Inventory
Sources for external hostnames:
- Public DNS zone records
- Certificate transparency logs
- Asset management records
- Historical scan results

## Stage 3: Map DNS to IPs
Resolve hostnames to public IPs, and track changes over time.
Be aware of CDN and load balancer endpoints.

## Stage 4: Low-Noise Discovery
Use ping sweeps or limited port checks to minimize noise.
External networks may block ICMP, so use TCP-based discovery if needed.

## Stage 5: Targeted Service Scans
Focus on common exposed services:
- Web servers (80, 443, 8080, 8443)
- Mail services (25, 465, 587, 110, 143, 993, 995)
- VPN and remote access (1194, 443, 500, 4500, 1723, 3389, 22)

## Stage 6: Analyze and Report
Interpret results with cloud context in mind and provide actionable findings.

## Cloud Provider Considerations
Each cloud provider has rules about scanning.
Always check provider policies or obtain explicit written approval.

## CDN and WAF Considerations
CDNs and WAFs can:
- Mask origin servers
- Rate-limit scanning
- Return generic responses that hide real services

## External Port Strategy
Use a tiered approach:
- Tier 1: Most common web and remote access ports
- Tier 2: Common infrastructure ports (DNS, SMTP, SSH)
- Tier 3: Full port sweep only when explicitly approved

## Concept: Accuracy vs Noise
External scanning should balance accuracy and noise to avoid triggering blocks.
Use conservative timing and reduced port lists first.

## Theory: Timing and Rate Control
Important options for external scanning:
- `-T2` or `-T3` for safe timing
- `--max-rate` to reduce bursts
- `--scan-delay` to add spacing

## Theory: DNS Resolution Strategy
Decide whether to:
- Resolve DNS to understand hostnames
- Use `-n` to avoid DNS noise
Use consistent policy across scans for comparability.

## Theory: Host Discovery on the Internet
ICMP is often blocked externally, so consider:
- `-Pn` to skip host discovery and scan anyway
- TCP SYN probes to common ports


## Concept: External Service Catalog
A service catalog ties external IPs and hostnames to business owners.
It helps prioritize remediation and reduce unknown exposure.

## Theory: Hostname Coverage Gaps
DNS enumeration can miss assets if records are stale or hidden.
Combine DNS with cloud inventory for better coverage.

## Concept: Reverse DNS Limitations
Reverse DNS is unreliable for identifying ownership or service purpose.
Use reverse DNS only as a hint.

## Theory: Scan Window Selection
Choose time windows with low traffic to reduce disruption.
Coordinate with operations teams to avoid conflicts.

## Concept: External Log Correlation
Correlate scan times with logs to validate findings and reduce noise.

## Theory: Rate Limiting Detection
If responses slow or stop, you may be rate-limited.
Adjust timing and max-rate accordingly.

## Concept: Cloud Asset Tags
Asset tags help map scan results to owners and environments.

## Theory: External Service Hardening
Scanning reveals exposures that should be hardened with firewall rules, VPNs, or authentication.

## Concept: External Monitoring Sensitivity
Some SOCs treat any external scan as potential attack.
Advance notification prevents unnecessary escalation.

## Theory: External IP Reuse
Cloud IPs can be reassigned if resources are terminated.
Validate IP ownership before every scan.

## Concept: Naming Conventions
Consistent naming of output files improves long-term tracking.

## Theory: External Compliance Requirements
Regulations may require periodic external scanning and reporting.

## Concept: Trust Boundaries
External scanning happens outside the trust boundary of internal systems.

## Theory: Detection of Shadow IT
External scans can reveal unmanaged assets exposed to the internet.

## Concept: Asset Criticality
Prioritize scans based on business criticality and data sensitivity.

## Theory: Reconnaissance vs Assessment
Recon maps exposure, while assessment evaluates risk and vulnerabilities.

## Concept: External TLS Renewal Monitoring
Scan results can show expiring or misconfigured certificates.

## Theory: Cloud Metadata Services
Metadata services are internal but can be exposed through misconfigurations.

## Concept: External API Gateways
API gateways often concentrate exposure and should be prioritized.

## Theory: IPv6 Expansion Risk
IPv6 exposure can grow faster than IPv4 visibility.

## Concept: Perimeter Baselines
Create baseline scans to measure change over time.

## Theory: Consistent Tooling
Use consistent scan options and tooling to reduce variance in results.

## Concept: Scan Attribution
Use dedicated scanning IPs to avoid reputation impact on production systems.

## Theory: External Port Reduction
Reducing open ports lowers attack surface and scan noise.

## Concept: Application Owner Engagement
Engage owners early to validate services and plan remediation.

## Theory: Exposure Lifecycle
New services should be reviewed before public exposure.

## Concept: External Threat Modeling
Threat modeling helps prioritize which services to scan more deeply.

## Theory: Deconflicting Security Tests
Coordinate with other security tests to avoid overlapping alerts.

## Concept: Attack Surface Reduction Programs
External scanning supports ongoing attack surface reduction.

## Theory: Third-Party Risk
External assets hosted by third parties may have different security controls.

## Concept: Internal vs External DNS Split
Split DNS can show different records internally and externally.

## Theory: Service Availability vs Security
Some services must remain exposed, requiring additional hardening.

## Concept: External Service Inventory Accuracy
Scan results should be reconciled with inventory data.

## Theory: Secure Decommissioning
Retired services should have DNS and IP exposure removed promptly.

## Concept: Exposure Review Meetings
Regular reviews help track remediation progress and risk posture.

## Theory: Responsible Communication
Share findings with clear evidence and actionable remediation steps.


## Practical: Basic External Scan (Web Ports)
```bash
nmap -p 80,443,8080,8443 -sV -T2 --max-rate 100 target
```

## Practical: External Scan with Output Management
```bash
nmap -p 80,443,8080,8443 -sV -T2 --max-rate 100 -oA outputs/ext-web target
```

## Practical: External Scan Using Target List
```bash
nmap -p 80,443,8080,8443 -sV -T2 --max-rate 100 -iL targets.txt -oA outputs/ext-batch
```

## Practical: External Scan Without DNS Resolution
```bash
nmap -n -p 80,443,8080,8443 -sV -T2 --max-rate 100 target
```

## Practical: External Host Discovery with TCP
```bash
nmap -sn -PS80,443,22,3389 -T2 --max-rate 100 203.0.113.0/24
```

## Concept: Cloud Load Balancers
Load balancers can:
- Present a single IP for multiple services
- Terminate TLS and hide backend versions
- Rotate IPs dynamically

## Concept: Shared IPs and Multi-Tenancy
Public IPs can serve multiple tenants on shared infrastructure.
Avoid scanning IPs not explicitly in scope.

## Concept: Elastic IPs and Ephemeral Instances
Cloud resources can change quickly; scan results are time-sensitive.

## Theory: Evidence Quality
External results often need corroboration:
- Validate service banners
- Check TLS certificates
- Confirm hostnames in HTTP headers

## Practical: HTTP Header Recon
```bash
nmap -p 80,443 --script http-headers,http-title target
```

## Practical: TLS Certificate Recon
```bash
nmap -p 443 --script ssl-cert,ssl-enum-ciphers target
```

## Concept: TLS Offload
TLS may be terminated at load balancers, so backend versions might not be visible.

## Concept: Origin Server Discovery
CDNs and WAFs can hide origin servers.
Origin discovery should only be done with explicit authorization.

## Theory: External Email Services
Mail servers are often internet-facing and should be scanned carefully.
Common ports:
- SMTP: 25, 465, 587
- IMAP: 143, 993
- POP3: 110, 995

## Practical: Mail Service Scan
```bash
nmap -p 25,465,587,110,143,993,995 -sV -T2 --max-rate 50 target
```

## Theory: Remote Access Services
External exposure of SSH, RDP, VPN, or admin portals is common.
Scan carefully and limit port lists to the approved scope.

## Practical: Remote Access Scan
```bash
nmap -p 22,3389,443,1194,500,4500,1723 -sV -T2 --max-rate 50 target
```

## Concept: Rate Limiting and Blocking
External services may block scans that are too fast.
Use conservative timing and monitor for dropped responses.

## Theory: False Positives on External Scans
External scans can produce false positives due to:
- CDN or WAF responses
- Generic error pages
- Shared infrastructure banners

## Concept: Hostname Validation
Ensure that DNS names map to approved IPs before scanning.

## Practical: Scan by Hostname
```bash
nmap -p 80,443 -sV --script http-title,http-headers example.com
```

## Concept: Subdomain Explosion
Large organizations can have thousands of subdomains.
Use controlled discovery and validate scope carefully.

## Theory: Asset Ownership Verification
Verify that IPs belong to your organization or authorized partner.
Do not scan third-party IPs outside scope, even if DNS points there.

## Practical: Scope Enforcement with Exclusions
```bash
nmap -p 80,443 -sV 203.0.113.0/24 --exclude 203.0.113.5,203.0.113.10
```

## Theory: Perimeter Change Detection
Repeated scans help detect new exposures.
Use consistent timing and output formatting for comparison.

## Practical: Save Output for Diffing
```bash
nmap -p 80,443 -sV -oA outputs/perimeter-20260206 203.0.113.0/24
```

## Theory: External Scan Scheduling
Schedule scans in approved windows to avoid business disruption.

## Concept: IPv6 External Perimeter
Many organizations expose IPv6 services unintentionally.
Include IPv6 in scope if authorized.

## Practical: IPv6 Scan Example
```bash
nmap -6 -p 80,443 -sV 2001:db8::/64
```

## Theory: Cloud Security Groups
Security groups determine which ports are exposed.
Scan results help validate security group configuration.

## Concept: External API Endpoints
APIs often run on HTTPS with specific paths.
Nmap can detect service banners but not API logic.

## Practical: External API Port Check
```bash
nmap -p 443 -sV --script http-headers,http-title api.example.com
```

## Theory: Managed Cloud Services
Managed databases, storage, and queues often expose endpoints with provider-controlled IPs.
Confirm provider scanning policies before testing.

## Concept: Cloud NAT and Outbound IPs
Outbound IPs are not necessarily inbound services.
Do not scan outbound-only IPs without evidence of inbound exposure.

## Theory: Interpreting Cloud Banners
Banners may show load balancer software rather than app servers.
Treat banners as hints, not proof of backend versions.

## Practical: Web Service Detection on Alternate Ports
```bash
nmap -p 8000,8080,8443,8888 -sV --script http-title,http-headers target
```

## Concept: External DNS Providers
DNS hosting is often external and separate from hosting infrastructure.
Do not assume DNS provider equals hosting provider.

## Theory: Risks of Aggressive Scans
Aggressive scans can:
- Trigger provider abuse alerts
- Block your source IP
- Impact production performance

## Practical: Conservative Timing Template
```bash
nmap -T2 --max-rate 50 -p 80,443 target
```

## Concept: Use of -Pn on External Hosts
If ICMP is blocked, use `-Pn` to skip discovery.

## Practical: -Pn Scan Example
```bash
nmap -Pn -p 80,443 -sV target
```

## Theory: Reputation and Source IPs
Scanning from a corporate IP range can trigger blacklisting.
Use approved scanning infrastructure with proper notification.

## Concept: Logging and Evidence
Maintain logs of scan options, timestamps, and outputs for audits.

## Practical: Output Management
```bash
nmap -p 80,443 -sV -oA outputs/external-web target
```

## Theory: Comparing Scan Results Over Time
Use `ndiff` with XML outputs to track changes.

## Practical: Ndiff Example
```bash
ndiff outputs/perimeter-20260201.xml outputs/perimeter-20260206.xml > outputs/perimeter-diff.txt
```

## Concept: External Vulnerability Scripts
Vulnerability scripts are risky externally.
Only run them with explicit authorization and limited scope.

## Theory: Script Categories for External Scans
Prefer `safe` and selected `default` scripts.
Avoid `intrusive` and `brute` categories unless explicitly approved.

## Practical: Safe Script Bundle
```bash
nmap -p 80,443 --script safe -sV target
```

## Concept: External SMB or Database Exposure
Any external SMB or database ports are high risk.
Report immediately if discovered.

## Practical: High-Risk Port Check
```bash
nmap -p 137,138,139,445,1433,1521,3306,5432,27017,6379 -sV -T2 target
```

## Theory: DDoS Protection Systems
DDoS protection can modify responses or rate-limit scans.
Interpret results carefully and verify with other sources.

## Concept: Scanning SaaS Endpoints
Many SaaS endpoints are out of scope.
Only scan those you are explicitly authorized to test.

## Practical: Hostname-Based Scan for SaaS Subdomain
```bash
nmap -p 443 -sV --script http-title,ssl-cert saas.example.com
```

## Theory: Cloud Logging and Alerts
Expect provider monitoring to detect scans.
Proactively notify provider contacts if required by policy.

## Concept: External Perimeter Reporting
External reports should prioritize:
- Unexpected open ports
- Deprecated protocols or weak TLS
- Exposed admin interfaces
- Services outside documented inventory

## Practical: Reporting Summary Template (Theory)
- Scope and dates
- Targets scanned
- Open ports and services
- High-risk exposures
- Recommendations

## Theory: IPv4 Exhaustion and CGNAT
Some services sit behind carrier-grade NAT.
Public IPs may not map to a single host.

## Concept: Service Identification on Shared IPs
Use host headers and TLS SNI for proper identification.

## Practical: Host Header Example
```bash
nmap -p 80 --script http-title --script-args http.host=www.example.com 203.0.113.10
```

## Theory: TLS SNI for HTTPS
If SNI is required, scanning by IP may return default certificates.

## Concept: Automation for External Scans
Automation ensures consistent options and safe timing across scans.

## Practical: External Scan Automation Script
```bash
#!/usr/bin/env bash
set -euo pipefail
TARGET_FILE="${1:-targets.txt}"
RUN_DIR="outputs/$(date +%Y%m%d-%H%M%S)"
mkdir -p "$RUN_DIR"
nmap -p 80,443,8080,8443 -sV -T2 --max-rate 100 -iL "$TARGET_FILE" -oA "$RUN_DIR/external-web"
```

## Theory: External Perimeter Reduction
A smaller perimeter reduces risk. Scans can reveal unused services that should be removed.

## Concept: External Attack Surface Management
External scanning feeds into attack surface management and asset inventory.

## Theory: Results Validation
Always validate critical findings with:
- Repeat scans with safe settings
- Manual checks using curl or browser
- Cross-checking DNS and certificates

## Practical: Manual Validation via Curl (Example)
```bash
curl -I https://example.com
```

## Concept: External Scanning and Change Control
Ensure scanning is aligned with change windows and operations teams.

## Theory: Responsible Disclosure Internally
Report findings with evidence and clear remediation steps.

## Concept: Scope Creep Risk
External scans can easily exceed scope due to DNS or shared IPs.
Always verify scope before scanning new targets.

## Theory: When to Use Full Port Scans
Full port scans (`-p-`) are high impact and should be approved explicitly.
Use them only on a small, validated scope.

## Practical: Limited Full-Port Scan with Conservative Timing
```bash
nmap -p- -sV -T2 --max-rate 50 target
```

## Concept: External Scanning and Incident Response
Scans can resemble attacks. Notify SOC or monitoring teams before scanning.

## Theory: Measuring Exposure Over Time
Track the count of externally exposed services and aim to reduce it.

## Concept: External Asset Tagging
Tag assets by function (web, mail, VPN) to prioritize scanning.

## Theory: External Scan Frequency
Determine frequency based on change rate and risk profile.

## Concept: Third-Party Hosted Assets
Third-party hosting requires explicit authorization and clear boundaries.

## Theory: Shared Responsibility Model
In cloud, you control configuration while provider controls infrastructure.
Scan results should be interpreted with shared responsibility in mind.

## Practical: Discovery Then Targeted Scan
```bash
nmap -sn -PS80,443 203.0.113.0/24 -oA outputs/ext-discovery
awk '/Up$/{print $2}' outputs/ext-discovery.gnmap > outputs/ext-live.txt
nmap -p 80,443,8080,8443 -sV -iL outputs/ext-live.txt -oA outputs/ext-services
```

## Concept: Host-Based Firewalls
Host firewalls can block probes even if perimeter allows traffic.
Interpret closed and filtered states carefully.

## Theory: Closed vs Filtered
Closed ports respond with reset; filtered ports drop packets.
External scans often show filtered due to firewalls.

## Practical: Use -vv for Debugging
```bash
nmap -vv -p 80,443 target
```

## Concept: Output Formats and Audits
Use `-oA` for consistent audit trails and easy parsing.

## Practical: Grepable Output Parsing
```bash
grep "/open/" outputs/ext-services.gnmap
```

## Theory: External Scanning in Regulated Environments
Regulated industries may have strict scanning rules and reporting requirements.

## Concept: Minimizing Business Impact
Use conservative options and avoid heavy scripts in production hours.

## Theory: Network Path Awareness
External traffic crosses multiple networks, which can alter or drop packets.

## Concept: BGP and IP Ownership
IP ownership can change. Validate ownership before scanning.

## Theory: External Perimeter Metrics
Useful metrics:
- Number of public IPs
- Number of open services
- TLS configuration grade distribution
- Trend of exposures over time

## Concept: Scan Profiles
Define scan profiles for repeatability:
- Light: top web ports only
- Standard: web + mail + remote access
- Full: full port scan (approved only)

## Practical: Light Profile Command
```bash
nmap -p 80,443,8080,8443 -sV -T2 --max-rate 100 target
```

## Practical: Standard Profile Command
```bash
nmap -p 22,25,80,443,465,587,110,143,993,995,3389,8080,8443 -sV -T2 --max-rate 80 target
```

## Practical: Full Profile Command (Approved Only)
```bash
nmap -p- -sV -T2 --max-rate 50 target
```

## Concept: External Perimeter Remediation
Findings should map to remediation steps:
- Close unused ports
- Restrict admin services to VPN
- Update TLS and server versions

## Theory: External Exposure of Databases
Databases should not be exposed to the public internet.
If discovered, report immediately as critical risk.

## Concept: External SMB Exposure
SMB should never be public. Report immediately.

## Practical: High-Risk Exposure Check
```bash
nmap -p 139,445,1433,3306,5432,27017,6379 -sV -T2 --max-rate 50 target
```

## Theory: TLS Weakness Detection
Use TLS scripts to find weak protocols or ciphers.

## Practical: TLS Script Bundle
```bash
nmap -p 443 --script ssl-cert,ssl-enum-ciphers,ssl-dh-params target
```

## Concept: Web Application Firewalls
WAFs can block scanning. Use safe scripts and conservative timing.

## Theory: Result Normalization
Normalize outputs across scans to compare apples to apples.

## Concept: External Perimeter in Dev vs Prod
Dev environments often expose more services by mistake.
Scan dev and prod separately with clear labels.

## Theory: Data Retention
Store scan outputs securely and limit access to sensitive data.

## Concept: Change Tracking
Use consistent file naming with dates to track changes.

## Practical: Date-Based Output Example
```bash
RUN_DIR="outputs/$(date +%Y%m%d)"
mkdir -p "$RUN_DIR"
nmap -p 80,443 -sV -oA "$RUN_DIR/external-web" target
```

## Theory: External Scan Governance
Define who can scan, when, and how results are reported.

## Concept: Notification and Incident Handling
Coordinate with SOC or IT teams to reduce false alarms.

## Theory: External Perimeter Risk Prioritization
Prioritize findings based on impact and exposure.

## Concept: Exposure vs Exploitability
An open port is an exposure, but not always exploitable.
Context matters for severity.

## Theory: Verification Steps
Use secondary tools or manual checks to verify critical findings.

## Concept: Legal Boundaries
External scanning can cross legal boundaries if scope is unclear.
Always confirm ownership and authorization.

## Summary
External perimeter scanning in cloud environments requires careful planning, conservative scanning, and strong documentation.
When done properly, it provides critical visibility into internet-facing risk.

## Reflection Questions
1. Why is external scanning riskier than internal scanning?
2. How do CDNs and load balancers affect scan results?
3. Which scan profile would you use first and why?
4. How do you validate ownership before scanning?

## Next Steps
Tomorrow (Day 27) you will learn about internal network enumeration.

**Congratulations on completing Day 26!** You now understand cloud and external perimeter scanning principles.
