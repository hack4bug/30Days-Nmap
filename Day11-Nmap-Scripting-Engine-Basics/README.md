# Day 11 - Nmap Scripting Engine (NSE) Basics

## Day Overview
Welcome to Day 11! Today, we begin exploring one of Nmap's most powerful features: the **Nmap Scripting Engine (NSE)**. This transforms Nmap from a simple scanner into a comprehensive security assessment platform. Think of NSE as giving Nmap "superpowers" - it can now not just find open doors, but also test locks, read signs, and even try doorknobs.

## Learning Objectives
By completing Day 11, you will be able to:
- Understand what NSE is and why it's important
- Learn the basic structure of NSE scripts
- Run basic scripts for common tasks
- Understand script categories and safety levels
- Begin exploring the NSE script library

## What is the Nmap Scripting Engine (NSE)?

### Basic Definition
The Nmap Scripting Engine (NSE) is a powerful framework that allows users to write (and share) scripts to automate a wide variety of networking tasks. These scripts extend Nmap's capabilities far beyond basic port scanning.

### The Evolution of Nmap

```cmd
Nmap without NSE:
1997-2006: Just a port scanner
Capabilities: Find open ports, detect services, guess OS

Nmap with NSE (2006+):
Extended capabilities: Vulnerability detection, exploit verification,
                      service enumeration, advanced discovery,
                      brute-force attacks, and much more
```

### Why NSE is Revolutionary

```cmd
Without NSE:
You discover: Port 80 is open (web server)
You need to: Manually test for vulnerabilities
              Check for default pages
              Test for directory traversal
              All manually - takes hours

With NSE:
You discover: Port 80 is open (web server)
NSE can: Automatically test for 50+ web vulnerabilities
         Check for default credentials
         Enumerate web technologies
         All automatically - takes minutes
```

## NSE Architecture Overview

### How NSE Works

```cmd
NSE Workflow:

1. Nmap performs initial scan
2. Discovers open ports and services
3. Passes information to NSE
4. NSE loads appropriate scripts
5. Scripts run and perform tests
6. Results reported back to user
```

### Key Components of NSE

1. **Scripts**: Individual Lua programs (.nse files)
2. **Categories**: Organized groups of scripts
3. **Libraries**: Reusable code modules
4. **Rules**: When scripts should run
5. **Output**: How results are displayed

## NSE Script Structure

### Basic Script Anatomy
Every NSE script follows a standard structure. Let's look at a simple example:

```cmd
-- Headers and metadata
description = [[
Simple script to check if a web server is running
]]

-- Categories this script belongs to
categories = {"default", "safe"}

-- Rule for when to run
portrule = function(host, port)
  return port.protocol == "tcp" 
     and port.number == 80 
     and port.state == "open"
end

-- Main action function
action = function(host, port)
  return "Web server detected on port 80"
end
```

### Key Sections Explained

**Description**: What the script does
**Categories**: Where the script fits (safe, intrusive, discovery, etc.)
**Port/Host Rules**: Conditions for when to run the script
**Action**: The main code that executes
**Output**: Results returned to user

## Running Your First NSE Scripts

### Basic Script Execution

```cmd
# Run a single script
nmap --script http-title target

# Run multiple scripts
nmap --script http-title,http-headers target

# Run all scripts in a category
nmap --script default target
```

### Common First Scripts to Try

#### **http-title Script**
Gets the title of web pages:
```cmd
nmap --script http-title -p 80,443 target
# Returns: Page titles like "Welcome to Apache"
```

#### **dns-brute Script**
Attempts DNS subdomain brute-forcing:
```cmd
nmap --script dns-brute target.com
# Tries: www, mail, ftp, admin, etc.
```

#### **ssh-hostkey Script**
Retrieves SSH host keys:
```cmd
nmap --script ssh-hostkey -p 22 target
# Returns: SSH key fingerprints
```

## Understanding Script Categories

### Major NSE Categories

#### **safe (-sC or --script=safe)**
```cmd
Purpose: Information gathering only
Examples: http-title, ssh-hostkey, smb-os-discovery
Risk: Low - won't crash services or cause damage
Command: nmap -sC target  # Runs all safe scripts
```

#### **intrusive**
```cmd
Purpose: More aggressive testing
Examples: brute-force attacks, vulnerability exploitation
Risk: Medium-High - may crash services or trigger alarms
Use: Only with explicit permission
```

#### **vuln**
```cmd
Purpose: Vulnerability detection
Examples: Tests for known CVEs, misconfigurations
Scripts: http-vuln*, smb-vuln*, ssh-vuln*
Command: nmap --script vuln target
```

#### **exploit**
```cmd
Purpose: Attempt actual exploitation
Examples: Metasploit-like exploit attempts
Risk: High - may compromise systems
Use: Only in controlled environments with permission
```

#### **discovery**
```cmd
Purpose: Network and service discovery
Examples: Find additional hosts, services, information
Scripts: broadcast*, targets-sniffer, targets-xml
Command: nmap --script discovery target
```

#### **auth**
```cmd
Purpose: Authentication testing
Examples: Default credential checks, brute-force
Scripts: http-auth, ssh-brute, mysql-brute
Risk: Medium - may lock accounts
```

#### **version**
```cmd
Purpose: Enhanced version detection
Examples: More detailed service fingerprinting
Works with: -sV flag for better detection
```

### Category Quick Reference

| Category | Purpose | Risk Level | Example Scripts |
|----------|---------|------------|-----------------|
| safe | Information gathering | Low | http-title, ssh-hostkey |
| intrusive | Aggressive testing | Medium-High | http-brute, smb-enum-users |
| vuln | Vulnerability checking | Medium | http-vuln-cve2017-5638 |
| exploit | Actual exploitation | High | smb-vuln-ms17-010 |
| discovery | Network discovery | Low | broadcast-dhcp-discover |
| auth | Authentication tests | Medium | ssh-brute, http-auth |
| version | Enhanced detection | Low | banner, http-server-header |

## NSE Script Locations and Management

### Finding Installed Scripts

```cmd
# List all available scripts
ls /usr/share/nmap/scripts/*.nse

# Count how many scripts you have
ls /usr/share/nmap/scripts/*.nse | wc -l

# Typical installation: 600+ scripts
```

### Common Script Locations

```cmd
Linux: /usr/share/nmap/scripts/
Windows: C:\Program Files\Nmap\scripts\
macOS: /usr/local/share/nmap/scripts/

Your scripts: ~/nmap/scripts/ (custom location)
```

### Updating NSE Scripts

```cmd
# Update Nmap (includes scripts)
sudo apt update && sudo apt upgrade nmap

# Update scripts database
sudo nmap --script-updatedb

# Check script database status
nmap --script-help | head -5
```

## Basic Script Execution Patterns

### Pattern 1: Service-Specific Scripts

```cmd
# Run web-related scripts
nmap --script "http-*" -p 80,443,8080,8443 target

# Run SSH-related scripts
nmap --script "ssh-*" -p 22 target

# Run SMB-related scripts
nmap --script "smb-*" -p 445 target
```

### Pattern 2: Category-Based Execution

```cmd
# Run all safe scripts
nmap -sC target

# Run vulnerability scripts
nmap --script vuln target

# Run discovery scripts
nmap --script discovery target
```

### Pattern 3: Targeted Script Execution

```cmd
# Run specific scripts by name
nmap --script http-title,http-headers,http-methods target

# Run scripts with arguments
nmap --script http-title --script-args http-title.url=/admin target
```

## Script Arguments and Configuration

### Passing Arguments to Scripts

```cmd
# Basic argument passing
nmap --script http-title --script-args http-title.url=/index.html target

# Multiple arguments
nmap --script dns-brute --script-args dns-brute.domain=example.com,dns-brute.threads=10 target

# Common useful arguments
--script-args userdb=users.txt,passdb=passwords.txt
--script-args http.useragent="Mozilla/5.0"
--script-args smbdomain=DOMAIN
```

### Common Script Arguments

#### **For Web Scripts**
```cmd
http-title.url: Specific path to check
http-headers.uri: URI for header check
http.useragent: Custom user agent string
http-auth.path: Authentication path
```

#### **For Brute-Force Scripts**
```cmd
userdb: Path to username file
passdb: Path to password file
brute.threads: Number of concurrent attempts
brute.retries: Number of retry attempts
```

#### **For Network Discovery**
```cmd
newtargets: Add discovered hosts to scan
max-newtargets: Limit new targets discovered
```

### Saving and Loading Script Arguments

```cmd
# Save commonly used arguments to file
echo "http.useragent='Mozilla/5.0'" > ~/nmap_args.txt
echo "smbdomain=WORKGROUP" >> ~/nmap_args.txt

# Use saved arguments
nmap --script smb-os-discovery --script-args-file ~/nmap_args.txt target
```

## Safety and Risk Management

### Understanding Script Risks

```cmd
Risk Levels:

Level 1 - Safe: Information gathering only
  Examples: http-title, ssh-hostkey
  Can run anywhere with permission

Level 2 - Intrusive: May trigger alarms
  Examples: http-brute, smb-enum-users
  Use only where expected/alerts acceptable

Level 3 - Dangerous: May crash services
  Examples: some vuln scripts
  Use in controlled environments only

Level 4 - Destructive: May cause damage
  Examples: exploit scripts
  Use only in isolated labs with permission
```

### Safe Testing Practices

```cmd
# Always start with safe scripts
nmap -sC target  # Safe scripts only

# Test on local systems first
nmap --script [new-script] 127.0.0.1

# Use dry-run mode for dangerous scripts
nmap --script [script] --script-trace target | head -100

# Monitor system/network during testing
```

### Script Safety Verification

```cmd
# Check script category before running
nmap --script-help http-brute

# Look for warning messages
nmap --script [script] --script-help | grep -i warning

# Test in isolated environment first
# Use virtual machines or lab networks
```

## Practical NSE Examples

### Example 1: Basic Web Server Analysis

```cmd
# Comprehensive web server check
nmap -sC -p 80,443,8080,8443 webserver.com

# This runs all safe scripts including:
- http-title: Gets page title
- http-headers: Shows HTTP headers
- http-methods: Tests HTTP methods
- http-server-header: Identifies server software
- ssl-cert: Shows SSL certificate
```

### Example 2: Network Service Enumeration

```cmd
# Enumerate SMB shares and users
nmap --script smb-enum-shares,smb-enum-users -p 445 target

# Results show:
- Available shares
- Share permissions
- User accounts
- Domain information
```

### Example 3: Vulnerability Checking

```cmd
# Check for common vulnerabilities
nmap --script vuln -p 80,443,22,445 target

# Tests for:
- Web application vulnerabilities
- SSH weaknesses
- SMB vulnerabilities (like EternalBlue)
- SSL/TLS issues
```

### Example 4: Service Brute-Forcing

```cmd
# Brute-force SSH (use with caution!)
nmap --script ssh-brute -p 22 --script-args userdb=users.txt,passdb=passwords.txt target

# Note: May lock accounts or trigger alarms
# Only use where explicitly permitted
```

## NSE Script Development Basics

### Creating Your First Script

```cmd
# Simple "Hello World" script
# Save as: hello-world.nse

description = [[
Simple Hello World script for NSE
]]

categories = {"safe", "discovery"}

portrule = function(host, port)
  return true  -- Runs on all open ports
end

action = function(host, port)
  return "Hello from NSE! Found: " .. port.service .. " on port " .. port.number
end
```

### Testing Your Script

```cmd
# Test your custom script
nmap --script ./hello-world.nse 127.0.0.1

# Add to Nmap scripts directory
sudo cp hello-world.nse /usr/share/nmap/scripts/
sudo nmap --script-updatedb

# Now use it like any other script
nmap --script hello-world target
```

### Useful NSE Libraries

```cmd
Common libraries for script development:

stdnse: Standard NSE functions
shortport: Port rule helpers
http: HTTP client library
smb: SMB protocol library
ssh2: SSH2 protocol library
dns: DNS resolution library
json: JSON parsing library
```

## NSE Best Practices

### Script Execution Strategy

```cmd
Recommended workflow:

1. Start with safe scripts
   nmap -sC target

2. Review results, identify interesting services

3. Run service-specific scripts
   nmap --script "http-*" target

4. If permitted, run vulnerability scripts
   nmap --script vuln target

5. For deep testing, run intrusive scripts
   nmap --script intrusive target

6. Document all findings and scripts used
```

### Performance Considerations

```cmd
# NSE scripts can slow down scans significantly
# Manage performance:

# Limit script execution
nmap --script-timeout 5m target  # 5 minute timeout

# Run scripts only on interesting ports
nmap --script default -p 80,443,22 target

# Use faster timing templates with scripts
nmap -T4 --script default target

# Monitor script progress
nmap -v --script default target
```

### Output Management

```cmd
# Control script output
nmap --script default --script-trace target  # Detailed output

# Save script results
nmap --script default -oA script-results target

# Filter script output
nmap --script default target | grep -A5 "SCRIPT"

# Compare script runs
ndiff scan1.xml scan2.xml
```

## Troubleshooting Common NSE Issues

### Issue: Script Not Running

```cmd
# Check if script exists
ls /usr/share/nmap/scripts/ | grep script-name

# Check script rules
nmap --script-help script-name

# Force script execution
nmap --script script-name --script-args newtargets target

# Check for errors
nmap -d --script script-name target  # Debug mode
```

### Issue: Script Errors or Crashes

```cmd
# Run with debug output
nmap -d --script script-name target

# Check Lua errors
nmap --script script-name --script-trace target

# Update Nmap and scripts
sudo apt update && sudo apt upgrade nmap

# Report bugs: https://nmap.org/submit/
```

### Issue: Slow Script Performance

```cmd
# Set timeout limits
nmap --script script-name --script-timeout 2m target

# Run fewer scripts at once
nmap --script script1,script2 target  # Instead of all

# Use appropriate timing template
nmap -T4 --script script-name target

# Skip intensive scripts initially
```

## NSE Security Considerations

### Ethical Use Guidelines

1. **Permission First**: Only run scripts on systems you own or have permission
2. **Understand Risks**: Know what each script does before running
3. **Start Safe**: Begin with safe scripts, escalate only when appropriate
4. **Monitor Impact**: Watch for negative effects on target systems
5. **Respect Privacy**: Don't collect unnecessary personal data

### Legal Considerations

- Some scripts may violate computer fraud laws
- Brute-force scripts may violate terms of service
- Vulnerability scanning may require written authorization
- Always check local laws and regulations

### Defensive Applications

As a defender, use NSE to:
1. **Test your own security**: Find vulnerabilities before attackers do
2. **Monitor for changes**: Regular scanning to detect unauthorized services
3. **Validate patches**: Confirm vulnerabilities are fixed
4. **Compliance checking**: Ensure systems meet security standards

## Next Steps

Tomorrow (Day 12) will cover:
- Using NSE Discovery Scripts
- Network enumeration techniques
- Service discovery automation
- Advanced discovery strategies

## Reflection Questions

1. What was the most useful NSE script you discovered today?
2. How do NSE scripts change your approach to network scanning?
3. What safety precautions will you take when using NSE scripts?
4. How could you use NSE scripts to improve your own network security?

## Need Help?

- Use `nmap --script-help` to learn about scripts
- Check NSE documentation: https://nmap.org/book/nse.html
- Explore the scripts directory: `ls /usr/share/nmap/scripts/`
- Practice on your own systems first
- Join Nmap community forums for support

**🎉 Congratulations on completing Day 11!** You've now unlocked the true power of Nmap with the Scripting Engine. You're no longer just scanning ports - you're performing intelligent security assessments.

*"The Nmap Scripting Engine transforms a simple scanner into a Swiss Army knife of network security. Today you learned how to open the first few tools."*