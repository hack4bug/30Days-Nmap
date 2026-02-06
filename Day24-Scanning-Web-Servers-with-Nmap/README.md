# Day 24 - Scanning Web Servers with Nmap

## Day Overview
Welcome to Day 24! Today you will learn how to scan web servers using Nmap, starting from basic checks and moving into advanced, targeted techniques.
This guide is intentionally detailed and focused only on web server scanning concepts and commands.

## Learning Objectives
By completing Day 24, you will be able to:
- Identify common web service ports and protocols
- Run safe, basic web scans and interpret results
- Use version detection and HTTP-specific scripts
- Assess TLS/SSL configuration with Nmap scripts
- Understand how web scanning fits into broader recon workflows

## Important Ethics Note
Only scan web servers you own or have explicit permission to test.
Web scanning can expose sensitive information if done improperly.

## Web Server Scanning Basics
Web servers typically listen on HTTP or HTTPS ports, but they can also run on non-standard ports.
Common ports include:
- 80 (HTTP)
- 443 (HTTPS)
- 8080 (HTTP alternate)
- 8443 (HTTPS alternate)
- 8000 (HTTP alternate)

## Basic Port Scans for Web Servers
Start with a focused port scan:
```bash
nmap -p 80,443,8080,8443 target
```
Use fast scan to quickly detect common web ports:
```bash
nmap -F target
```

## Version Detection for Web Services
Once ports are found, use service version detection:
```bash
nmap -sV -p 80,443,8080,8443 target
```
This helps identify:
- Web server software (Apache, Nginx, IIS, etc.)
- Version numbers for patching status
- Proxy or load balancer clues

## Basic HTTP Scripts
Nmap includes many HTTP-related NSE scripts.
The safest starting scripts are:
- `http-title`
- `http-server-header`
- `http-methods`
- `http-headers`

Example:
```bash
nmap -p 80,443 --script http-title,http-server-header,http-methods target
```

## Understanding HTTP Headers
HTTP headers can reveal:
- Server software and version
- Security headers (HSTS, CSP, etc.)
- Redirects and load balancer hints

Example:
```bash
nmap -p 80,443 --script http-headers target
```

## Checking HTTP Methods
Use `http-methods` to discover allowed methods:
```bash
nmap -p 80,443 --script http-methods target
```
Look for risky methods such as PUT, DELETE, or TRACE.

## Discovering Web Directories
Use `http-enum` for common directories and files:
```bash
nmap -p 80,443 --script http-enum target
```
This finds:
- Admin panels
- Backup files
- Test pages
- Common application paths

## Web Application Fingerprinting
Nmap can fingerprint web technologies and applications.
Useful scripts:
- `http-generator`
- `http-php-version`
- `http-wordpress-users`
- `http-robots.txt`

Examples:
```bash
nmap -p 80,443 --script http-generator target
nmap -p 80,443 --script http-php-version target
nmap -p 80,443 --script http-robots.txt target
```

## HTTPS and TLS/SSL Scanning
TLS configuration is critical for web security.
Nmap scripts can check for:
- Supported protocols
- Weak ciphers
- Certificate details
- Known vulnerabilities

Common TLS scripts:
- `ssl-enum-ciphers`
- `ssl-cert`
- `ssl-dh-params`

Examples:
```bash
nmap -p 443 --script ssl-cert,ssl-enum-ciphers target
```

## Checking Certificates
`ssl-cert` provides:
- Subject and issuer info
- Validity dates
- Subject Alternative Names (SANs)

Example:
```bash
nmap -p 443 --script ssl-cert target
```

## Enumerating Virtual Hosts
Many servers host multiple sites on one IP.
You must scan with the correct hostname:
```bash
nmap -p 80,443 --script http-title --script-args http.host=example.com target
```
Use DNS enumeration from earlier days to discover hostnames.

## Handling Redirects
Some sites redirect HTTP to HTTPS or to another hostname.
Nmap scripts usually follow simple redirects, but sometimes manual checks are needed.

## Scanning Non-Standard Ports
Web servers often run on alternate ports.
Scan common alternates:
```bash
nmap -p 80,443,8000,8080,8443,8888 target
```
Or use version detection across a range:
```bash
nmap -sV -p 1-10000 target
```

## Identifying Web Servers by Service Detection
Nmap detects services even on unusual ports:
```bash
nmap -sV -p 1-65535 target
```
This can be slow; use with caution and permission.

## Web Vulnerability Checks (Safe Subset)
Nmap includes vulnerability scripts, but use them carefully.
Beginner-friendly examples:
- `http-vuln-cve2014-3704` (Drupalgeddon)
- `http-vuln-cve2015-1635` (IIS range)
- `http-vuln-cve2017-5638` (Struts)

Example:
```bash
nmap -p 80,443 --script http-vuln-cve2015-1635 target
```
Only run vuln scripts with explicit permission.

## Safe Script Categories for Web Scanning
Use script categories to control risk:
- `safe` for low-risk checks
- `default` for standard scripts
- `vuln` for vulnerability checks (use with permission)

Example:
```bash
nmap -p 80,443 --script safe target
```

## Combining Scripts for a Web Scan Bundle
```bash
nmap -p 80,443,8080,8443 -sV --script http-title,http-headers,http-methods,http-enum target
```

## Scanning Web Servers at Scale
Use target lists for multiple sites:
```bash
nmap -p 80,443,8080,8443 -sV --script http-title,http-headers -iL targets.txt -oA outputs/web-batch
```
Combine with Day 19 multi-target strategies to manage scope.

## Output Management for Web Scanning
Always use `-oA` so you can parse and report:
```bash
nmap -p 80,443 -sV --script http-title,http-headers -oA outputs/web-scan target
```

## Parsing Results (Day 17 Skills Applied)
Extract open web ports:
```bash
grep "/open/" outputs/web-scan.gnmap
```
Extract titles:
```bash
grep -i "http-title" outputs/web-scan.nmap
```

## Handling Authentication
Some scripts support credentials for authenticated checks.
Always handle credentials securely and only with permission.

## Web Server Types and Indicators
Common server types and clues:
- Apache: `Server: Apache`
- Nginx: `Server: nginx`
- IIS: `Server: Microsoft-IIS`
- Caddy: `Server: Caddy`

## HTTP vs HTTPS Differences
- HTTP is unencrypted
- HTTPS adds TLS security
- HTTPS requires certificate validation
- Some services only respond properly over HTTPS

## TLS Protocol Enumeration
```bash
nmap -p 443 --script ssl-enum-ciphers target
```
This lists supported TLS versions and ciphers.

## Checking HSTS and Security Headers
Use `http-headers` to identify:
- Strict-Transport-Security
- Content-Security-Policy
- X-Frame-Options
- X-Content-Type-Options

## Checking for Robots.txt
```bash
nmap -p 80,443 --script http-robots.txt target
```
Robots files often reveal sensitive directories.

## Identifying CMS Platforms
Nmap scripts can find CMS hints, but results are not always reliable.
Useful scripts:
- `http-wordpress-users`
- `http-joomla-brute` (risky, requires permission)

## Brute-Force Scripts Warning
Some web scripts are intrusive (brute-force, login checks).
Only run those in approved testing with explicit permission.

## Web Proxy and Load Balancer Detection
Some headers indicate proxies or load balancers.
Check for:
- `Via` headers
- `X-Forwarded-For`
- Vendor-specific headers

## Virtual Host Scanning Concept
If multiple sites exist on one IP, you must scan each hostname separately.
Use `http.host` script args to set host header.

## Combining with Automation (Day 18)
```bash
#!/usr/bin/env bash
set -euo pipefail
TARGET_FILE="${1:-targets.txt}"
RUN_DIR="outputs/$(date +%Y%m%d-%H%M%S)"
mkdir -p "$RUN_DIR"
nmap -p 80,443,8080,8443 -sV --script http-title,http-headers -iL "$TARGET_FILE" -oA "$RUN_DIR/web"
```

## Handling WAFs (Web Application Firewalls)
WAFs can block or rate-limit scans.
Use conservative timing and avoid aggressive scripts.

## Error Pages and False Positives
Some servers return identical responses for missing pages.
Be cautious when interpreting enumeration results.

## HTTP Status Codes You Should Know
- 200 OK: Success
- 301/302: Redirect
- 401: Unauthorized
- 403: Forbidden
- 404: Not Found
- 500: Server error

## Practical Web Scanning Workflow (Theory)
1. Identify web ports
2. Run version detection
3. Collect headers and titles
4. Enumerate directories
5. Check TLS configuration
6. Validate findings with a browser or curl

## Web Scanning in Large Environments
- Use target lists to control scope
- Randomize host order
- Limit scripts to safe ones
- Store output with timestamps

## Advanced Script Bundles
Example for broad web enumeration:
```bash
nmap -p 80,443,8080,8443 -sV --script "http-title,http-headers,http-methods,http-enum,http-server-header" target
```

## Cross-Checking with TLS on Non-Standard Ports
Some HTTPS services run on 8443 or 9443.
Check TLS on those ports explicitly.

## Using `--script-args` with HTTP Scripts
Many HTTP scripts support arguments.
Example:
```bash
nmap -p 80 --script http-enum --script-args http-enum.category=common target
```

## Combining with Multi-Target Scanning (Day 19)
```bash
nmap -p 80,443,8080,8443 -sV --script http-title,http-headers -iL targets.txt -oA outputs/web-multi
```

## Reporting Web Scan Results
Document:
- Targets scanned and ports checked
- Server software and versions
- TLS configuration results
- Any risky methods or headers

## Common Misconfigurations Found by Nmap
- Outdated web server versions
- Missing TLS security headers
- Weak TLS ciphers
- Exposed admin panels
- Allowed risky HTTP methods

## Defensive Use Cases
- Inventory public web servers
- Validate TLS upgrades
- Detect misconfigured headers
- Identify forgotten test sites

## Limitations of Nmap for Web Scanning
- Not a full web vulnerability scanner
- Limited application logic testing
- Best used for reconnaissance and surface mapping

## Best Practices
- Start with safe scripts
- Use version detection to focus checks
- Save output in multiple formats
- Avoid intrusive scripts unless permitted
- Verify results with a browser or curl

## Advanced Concepts (Theory)
### Virtual Hosts and SNI
Many HTTPS servers require SNI for correct responses.
If you do not provide the hostname, results can be misleading.

### HTTP/2 and HTTP/3
Modern servers may support HTTP/2 or HTTP/3.
Nmap can detect some of these, but specialized tools may be better.

### Reverse Proxies
Reverse proxies can hide backend server details.
Nmap will only see the proxy front.

### CDN Considerations
CDNs can mask origin servers and present shared IPs.
Be careful about attribution and scope.

## Real-World Web Scan Strategy
- Discover all web ports
- Map hostnames to IPs
- Scan each hostname with correct Host header
- Enumerate directories and headers
- Check TLS configuration
- Report findings with evidence

## Step-by-Step Scan Examples 
### Example A: Basic Web Recon
```bash
nmap -p 80,443 -sV --script http-title,http-headers target -oA outputs/basic-web
```

### Example B: TLS-Focused Scan
```bash
nmap -p 443 --script ssl-cert,ssl-enum-ciphers target -oA outputs/tls
```

### Example C: Directory Enumeration
```bash
nmap -p 80,443 --script http-enum target -oA outputs/http-enum
```

### Example D: Web Batch Scan
```bash
nmap -p 80,443,8080,8443 -sV --script http-title,http-headers -iL targets.txt -oA outputs/web-batch
```

## Concept Deep-Dive: What Nmap Can and Cannot Tell You
- Can tell: open ports, service banners, TLS configs, common directories
- Cannot tell: full application logic flaws, authentication bypasses, business logic issues

## Reading and Interpreting Output
Look for:
- `open` web ports
- Service banners and versions
- HTTP titles and server headers
- TLS protocol and cipher support

## Errors and Pitfalls
- False positives from error pages
- Missing virtual host headers
- Scanning the wrong port for HTTPS
- Overly aggressive scripts that trigger blocks

## Checklist for Web Scans
- Confirm permission and scope
- Identify likely web ports
- Run version detection
- Use safe scripts first
- Check TLS configuration
- Save output for reporting

## Reflection Questions
1. Why is Host header important when scanning virtual hosts?
2. Which scripts are safest for initial web scans?
3. Why is TLS scanning important for HTTPS services?
4. What limitations does Nmap have for web application testing?

## Next Steps
Tomorrow (Day 25) you will learn about database and SMB enumeration.

**Congratulations on completing Day 24!** You now have a complete, structured approach to scanning web servers with Nmap.
