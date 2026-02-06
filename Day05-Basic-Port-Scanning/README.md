# Day 05 - Basic Port Scanning

## Day Overview
Welcome to Day 5! Today, we dive into the core of Nmap's functionality: **Port Scanning**. After discovering which hosts are alive (Day 4), we now learn what services those hosts are offering by examining their open ports.

## Learning Objectives
By completing Day 05, you will be able to:
- Understand what port scanning is and why it's essential
- Perform basic TCP and UDP port scans
- Interpret different port states (open, closed, filtered)
- Use different scanning techniques
- Save and analyze scan results

## What is Port Scanning?

### Basic Concept
Port scanning is the process of checking which network ports on a target system are open and listening for connections. Think of it like checking which doors and windows in a building are unlocked and accessible.

### Why Port Scanning Matters

```cmd
Without Port Scanning:
You know: "192.168.1.100 is online"
But don't know: "What services does it run?"

With Port Scanning:
You know: "192.168.1.100 is online"
And know: "It has web server (port 80), SSH (port 22), etc."
```

### Port States Explained

Nmap categorizes ports into different states:

**Open**: Service is actively accepting connections
- Example: `80/tcp open http`
- Meaning: Web server is running and accessible

**Closed**: Port is accessible but no service listening
- Example: `81/tcp closed`
- Meaning: Port is reachable but nothing's running there

**Filtered**: Firewall or security device blocking access
- Example: `22/tcp filtered ssh`
- Meaning: Something is preventing connection attempts

**Unfiltered**: Port is accessible but state unknown
- Example: Rarely seen in basic scans
- Meaning: Requires further testing

## Basic Port Scanning Techniques

### 1. TCP Connect Scan (-sT)

**What it does**: Completes full TCP three-way handshake
```cmd
# Basic TCP Connect Scan
nmap -sT 192.168.1.1

# Scan specific ports
nmap -sT -p 80,443,22 192.168.1.1

# Scan port range
nmap -sT -p 1-100 192.168.1.1
```

**How it works**:
1. Sends SYN packet
2. Receives SYN-ACK (port open) or RST (port closed)
3. Sends ACK to complete handshake (if open)
4. Immediately sends RST to close connection

**Best for**:
- When you don't have root/admin privileges
- Basic network reconnaissance
- Simple connectivity testing

### 2. TCP SYN Scan (-sS) - Stealth Scan

**What it does**: Half-open scanning (doesn't complete handshake)
```cmd
# Requires root/admin privileges
sudo nmap -sS 192.168.1.1

# SYN scan with specific ports
sudo nmap -sS -p 21,22,23,25,80 192.168.1.1
```

**How it works**:
1. Sends SYN packet
2. Receives SYN-ACK (port open) or RST (port closed)
3. Sends RST instead of ACK (never completes handshake)

**Best for**:
- Stealthier scanning
- When you have root/admin access
- Avoiding connection logs

### 3. UDP Scan (-sU)

**What it does**: Scans UDP ports (different protocol than TCP)
```cmd
# UDP scan (slower than TCP)
sudo nmap -sU 192.168.1.1

# UDP scan of common ports
sudo nmap -sU -p 53,67,68,69,123 192.168.1.1
```

**How it works**:
1. Sends UDP packet to each port
2. Waits for ICMP "port unreachable" response (port closed)
3. No response often means port open (UDP is connectionless)

**Best for**:
- DNS servers (port 53)
- DHCP services (ports 67, 68)
- SNMP (port 161)
- Time services (port 123)

### 4. Comprehensive Scan (-sC)

**What it does**: Combines multiple scan types
```cmd
# Comprehensive scan
nmap -sC 192.168.1.1

# Equivalent to multiple flags
nmap -sS -sV -O --script=default 192.168.1.1
```

## Understanding Port Ranges

### Default Port Scan
By default, Nmap scans the top 1000 most common ports:
```cmd
# This scans top 1000 ports
nmap 192.168.1.1
```

### Common Port Range Options

**Scan all ports (1-65535)**:
```cmd
nmap -p- 192.168.1.1
# or
nmap -p 1-65535 192.168.1.1
```

**Scan specific ports**:
```cmd
nmap -p 80,443,8080 192.168.1.1
```

**Scan port range**:
```cmd
nmap -p 1-100 192.168.1.1
```

**Scan common service ports**:
```cmd
nmap -p http,https,ssh,ftp 192.168.1.1
```

## Reading and Interpreting Scan Results

### Sample Scan Output
```cmd
$ nmap 192.168.1.1
Starting Nmap 7.93 ( https://nmap.org ) at 2024-01-15 10:00 EST
Nmap scan report for router.local (192.168.1.1)
Host is up (0.005s latency).
Not shown: 996 closed ports
PORT     STATE SERVICE
22/tcp   open  ssh
53/tcp   open  domain
80/tcp   open  http
443/tcp  open  https
8080/tcp open  http-proxy

Nmap done: 1 IP address (1 host up) scanned in 1.23 seconds
```

### Breaking Down the Output

**Header Information**:
- Nmap version and source
- Scan timestamp
- Target identification

**Host Status**:
- Host is up (responsive)
- Latency measurement (0.005s = fast local network)

**Port Summary**:
- "Not shown: 996 closed ports" - Most ports were closed
- Only showing interesting (open) ports

**Port Details Table**:
  ```cmd
PORT STATE SERVICE
22/tcp open ssh
53/tcp open domain
80/tcp open http
443/tcp open https
8080/tcp open http-proxy
  ```

  
**Footer**:
- Total hosts scanned
- Scan duration

## Practical Scanning Examples

### Example 1: Basic Home Router Scan
```cmd
# Scan common home router
nmap 192.168.1.1

# Expected common findings:
# - Port 80: Web administration
# - Port 443: Secure admin (if enabled)
# - Port 53: DNS server
# - Port 22: SSH (if enabled)
```

### Example 2: Web Server Scan
```cmd
# Focus on web-related ports
nmap -p 80,443,8080,8443 192.168.1.100

# Add version detection
nmap -sV -p 80,443,8080,8443 192.168.1.100
```

### Example 3: Multiple Target Scan
```cmd
# Scan multiple IPs
nmap 192.168.1.1 192.168.1.100 192.168.1.150

# Scan IP range
nmap 192.168.1.1-50

# Scan entire subnet
nmap 192.168.1.0/24
```

## Output Options for Saving Results

### Save to Different Formats
```cmd
# Normal text output
nmap -oN scan_results.txt 192.168.1.1

# XML format (good for tools)
nmap -oX scan_results.xml 192.168.1.1

# Grepable format (easy parsing)
nmap -oG scan_results.gnmap 192.168.1.1

# All formats at once
nmap -oA scan_results 192.168.1.1
```

### View Saved Results
```cmd
# View text results
cat scan_results.txt

# Count open ports
grep "open" scan_results.txt | wc -l

# List all open ports
grep "open" scan_results.txt
```

## Timing and Performance Options

### Timing Templates (-T)
```cmd
# Paranoid (T0) - Very slow, very stealthy
nmap -T0 192.168.1.1

# Sneaky (T1) - Slow, stealthy
nmap -T1 192.168.1.1

# Polite (T2) - Default for many
nmap -T2 192.168.1.1

# Normal (T3) - Default
nmap -T3 192.168.1.1

# Aggressive (T4) - Faster
nmap -T4 192.168.1.1

# Insane (T5) - Very fast, may be detected
nmap -T5 192.168.1.1
```

### Custom Timing
```cmd
# Set packet delay
nmap --max-rtt-timeout 100ms 192.168.1.1

# Set retry attempts
nmap --max-retries 2 192.168.1.1

# Set scan delay
nmap --scan-delay 1s 192.168.1.1
```

## Common Scanning Patterns

### Pattern 1: Quick Overview
```cmd
# Fast scan of common ports
nmap -F 192.168.1.1
# Scans top 100 ports instead of 1000
```

### Pattern 2: Comprehensive Scan
```cmd
# Full scan with version detection
nmap -sV -sC -O 192.168.1.1
# -sV: Version detection
# -sC: Default scripts
# -O: OS detection
```

### Pattern 3: Service-Specific Scan
```cmd
# Web server focused
nmap -p 80,443,8080,8443 -sV --script=http* 192.168.1.1
```

## Practice Exercises

### Exercise 1: Basic Port Scan
```cmd
# Task: Scan your localhost
nmap 127.0.0.1

# Questions:
1. How many ports are open?
2. What services are running?
3. Are any ports unexpected?
4. Save the results to a file
```

### Exercise 2: Compare Scan Types
```cmd
# Compare different scan methods
# TCP Connect Scan
nmap -sT 127.0.0.1

# TCP SYN Scan (if you have privileges)
sudo nmap -sS 127.0.0.1

# Questions:
1. Do both show the same ports?
2. Which was faster?
3. Any differences in results?
```

### Exercise 3: Port Range Practice
```cmd
# Practice with different port ranges
# Scan well-known ports (1-1024)
nmap -p 1-1024 127.0.0.1

# Scan specific service ports
nmap -p 21,22,23,25,53,80,110,143,443 127.0.0.1

# Scan all ports (be patient!)
nmap -p- 127.0.0.1

# Questions:
1. How many more ports did you find with -p-?
2. Were there services on high-numbered ports?
```

### Exercise 4: Output Management
```cmd
# Practice saving and analyzing results
# Save in all formats
nmap -oA my_scan 127.0.0.1

# Analyze the results
cat my_scan.nmap
cat my_scan.gnmap | grep "open"

# Questions:
1. Which format is easiest to read?
2. Which format would be best for automation?
```

## Troubleshooting Common Issues

### Issue 1: "You requested a scan type which requires root privileges"
```cmd
# Solution: Use sudo or different scan type
sudo nmap -sS 192.168.1.1
# or use non-privileged scan
nmap -sT 192.168.1.1
```

### Issue 2: Slow Scanning
```cmd
# Solution: Adjust timing or ports
nmap -T4 -F 192.168.1.1  # Faster timing, fewer ports
nmap --max-rtt-timeout 500ms 192.168.1.1  # Increase timeout
```

### Issue 3: No Results or Timeouts
```cmd
# Solution: Check connectivity and firewall
ping 192.168.1.1
nmap -Pn 192.168.1.1  # Skip host discovery
nmap -p 80 192.168.1.1  # Test specific port
```

## Best Practices

### 1. Start with Fast Scans
```cmd
# Begin with quick overview
nmap -F target
# Then focus on interesting findings
```

### 2. Be Mindful of Network Impact
- Scan during off-hours for production systems
- Use appropriate timing templates
- Monitor network performance during scans

### 3. Document Everything
- Save all scan results
- Note scan parameters used
- Record date/time of scans

### 4. Understand Legal Boundaries
- Only scan systems you own or have permission to test
- Be aware of network policies
- Respect privacy and data protection laws

## Next Steps

Tomorrow (Day 06) will cover:
- Advanced scanning techniques
- TCP SYN vs Connect scans in depth
- Port scanning strategies
- Real-world scanning scenarios

## Reflection Questions

1. What was the most surprising port you found open?
2. How does understanding port states help with security?
3. Which scanning technique worked best for your environment?
4. What challenges did you face during scanning?

## Need Help?

- Use `nmap -h` for command reference
- Check Nmap documentation at https://nmap.org/book/
- Practice in isolated lab environments
- Join cybersecurity learning communities

---

<br> 

**🎉 Congratulations on completing Day 05!** You've mastered the crucial first step in network reconnaissance. Tomorrow, we'll build on this by learning how to scan the services running on discovered hosts.


<br> 