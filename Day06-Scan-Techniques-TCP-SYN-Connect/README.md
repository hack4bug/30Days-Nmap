# Day 06 - Scan Techniques: TCP SYN vs Connect

## Day Overview
Welcome to Day 6! Today, we dive deep into two of the most important port scanning techniques: **TCP SYN Scan** and **TCP Connect Scan**. Understanding the differences between these methods is crucial for effective and appropriate network reconnaissance.

## Learning Objectives
By completing Day 06, you will be able to:
- Understand the fundamental differences between SYN and Connect scans
- Choose the appropriate scan type for different situations
- Execute both scan types correctly
- Interpret results from each method
- Understand the technical mechanics behind each scan

## TCP Fundamentals Review

### The TCP Three-Way Handshake
Before understanding scan types, let's review how normal TCP connections work:

```cmd
Normal TCP Connection:
Step 1: Client → Server: SYN (Synchronize)
Step 2: Server → Client: SYN-ACK (Synchronize-Acknowledge)
Step 3: Client → Server: ACK (Acknowledge)

Result: Connection established, data transfer can begin
```

## TCP Connect Scan (-sT)

### What is TCP Connect Scan?
The Connect scan is the most straightforward scanning method - it completes the full TCP three-way handshake.

### How It Works
```cmd
TCP Connect Scan Process:
Step 1: Nmap → Target: SYN packet
Step 2: Target → Nmap: SYN-ACK (if port open)
Step 3: Nmap → Target: ACK (completes handshake)
Step 4: Nmap → Target: RST (immediately closes connection)

If port closed: Target sends RST at Step 2
```

### Command Examples
```cmd
# Basic Connect Scan
nmap -sT 192.168.1.1

# Connect scan with specific ports
nmap -sT -p 22,80,443 192.168.1.1

# Connect scan with timing control
nmap -sT -T4 192.168.1.1
```

### Advantages of Connect Scan
1. **No special privileges needed** - Works as regular user
2. **Simple and reliable** - Uses standard TCP connections
3. **Less likely to trigger alarms** - Looks like normal traffic
4. **Works through most firewalls** - Uses established TCP connections

### Disadvantages of Connect Scan
1. **Creates full connections** - Can be logged by target systems
2. **Slower than SYN scans** - Complete handshake takes time
3. **More detectable** - Each scan creates a logged connection
4. **Resource intensive** - Maintains connection state

## TCP SYN Scan (-sS) - Stealth Scan

### What is TCP SYN Scan?
The SYN scan is often called a "half-open" or "stealth" scan because it doesn't complete the TCP handshake.

### How It Works
```cmd
TCP SYN Scan Process:
Step 1: Nmap → Target: SYN packet
Step 2: Target → Nmap: SYN-ACK (if port open)
Step 3: Nmap → Target: RST (instead of ACK - breaks handshake)

If port closed: Target sends RST at Step 2
```

### Command Examples
```cmd
# SYN Scan requires root/admin privileges
sudo nmap -sS 192.168.1.1

# SYN scan with port specification
sudo nmap -sS -p 1-100 192.168.1.1

# SYN scan with output options
sudo nmap -sS -oN syn_scan.txt 192.168.1.1
```

### Advantages of SYN Scan
1. **Stealthier** - Doesn't create full connections
2. **Faster** - No handshake completion needed
3. **Less likely to be logged** - Many systems don't log half-open attempts
4. **More efficient** - Uses fewer resources

### Disadvantages of SYN Scan
1. **Requires privileges** - Needs root/admin access
2. **May be blocked** - Some firewalls detect and block SYN scans
3. **Can trigger IDS/IPS** - Security systems may flag as suspicious
4. **Not as reliable** - Some systems handle SYN packets differently

## Side-by-Side Comparison

### Technical Comparison Table
| Aspect | TCP Connect Scan (-sT) | TCP SYN Scan (-sS) |
|--------|------------------------|-------------------|
| **Privileges** | Regular user | Root/Admin required |
| **TCP Handshake** | Completes full handshake | Half-open (no ACK) |
| **Speed** | Slower | Faster |
| **Stealth Level** | Lower (creates connections) | Higher (stealthier) |
| **Detection Risk** | Higher (logged connections) | Lower (often not logged) |
| **Reliability** | High (standard TCP) | High (but varies) |
| **Firewall Bypass** | Standard connections | May bypass some logging |
| **Resource Usage** | Higher (maintains state) | Lower |

### Network Traffic Comparison
```cmd
Connect Scan Traffic Pattern:
Client: SYN → Server
Server: SYN-ACK → Client
Client: ACK → Server
Client: RST → Server
(Two additional packets per port)

SYN Scan Traffic Pattern:
Client: SYN → Server
Server: SYN-ACK → Client
Client: RST → Server
(One fewer packet per port)
```

## Practical Scenarios

### Scenario 1: Internal Network Assessment
```cmd
# For internal trusted networks
# Connect scan is often sufficient
nmap -sT 192.168.1.0/24

# Or if you have privileges, use SYN
sudo nmap -sS 192.168.1.0/24
```

### Scenario 2: External Penetration Test
```cmd
# For external testing where stealth matters
sudo nmap -sS -T2 target.com

# Or combine with timing for better stealth
sudo nmap -sS -T1 --max-rate 10 target.com
```

### Scenario 3: Quick Internal Check
```cmd
# Quick check without privileges
nmap -sT -F 192.168.1.1  # Fast scan top 100 ports
```

## Advanced SYN Scan Techniques

### SYN Scan with Decoys
```cmd
# Hide your real IP among decoys
sudo nmap -sS -D 192.168.1.100,192.168.1.101,ME 192.168.1.1

# ME represents your real IP position
# Target sees scan from multiple sources
```

### SYN Scan with Spoofed Source
```cmd
# Spoof your source address (advanced)
sudo nmap -sS -S 192.168.1.50 -e eth0 192.168.1.1

# -S: Spoofed source IP
# -e: Specify network interface
# Note: Replies won't reach you!
```

### SYN Scan with Fragmentation
```cmd
# Fragment packets to evade detection
sudo nmap -sS -f 192.168.1.1

# Or specify fragment size
sudo nmap -sS --mtu 24 192.168.1.1
```

## Real-World Examples

### Example 1: Web Server Scan Comparison
```cmd
# Connect scan on web server
nmap -sT -p 80,443,8080,8443 webserver.com

# SYN scan on same server
sudo nmap -sS -p 80,443,8080,8443 webserver.com

# Compare:
# - Response times
# - Results consistency
# - Any differences in detected ports
```

### Example 2: Multi-target Strategy
```cmd
# Strategy: Use SYN for stealth, fallback to Connect
# Step 1: Try SYN (if privileged)
sudo nmap -sS target1 target2 target3

# Step 2: If no privilege, use Connect
nmap -sT target1 target2 target3
```

## Analyzing Results

### Reading Connect Scan Results
```cmd
$ nmap -sT 192.168.1.1
PORT     STATE SERVICE
22/tcp   open  ssh
80/tcp   open  http
443/tcp  open  https

# Connect scan creates actual connections
# These may appear in target's connection logs
```

### Reading SYN Scan Results
```cmd
$ sudo nmap -sS 192.168.1.1
PORT     STATE SERVICE
22/tcp   open  ssh
80/tcp   open  http
443/tcp  open  https

# Same results, but different method
# Less likely to appear in logs
```

### Identifying Differences
Sometimes SYN and Connect scans produce different results due to:
1. **Firewall rules** - May treat SYN differently
2. **Load balancers** - Can respond differently
3. **IDS/IPS systems** - May block one method but not the other
4. **TCP stack variations** - Different OS implementations

## Practice Exercises

### Exercise 1: Direct Comparison
```cmd
# Task: Compare both methods on localhost
# If you have root privileges:
sudo nmap -sS 127.0.0.1
nmap -sT 127.0.0.1

# Questions:
1. Were results identical?
2. Which was faster?
3. Check system logs - were connections logged?
```

### Exercise 2: Privilege Experiment
```cmd
# Task: Test what happens without privileges
# Try SYN scan as regular user
nmap -sS 127.0.0.1

# What error do you get?
# How does Nmap handle this?
```

### Exercise 3: Timing Comparison
```cmd
# Task: Measure speed difference
time nmap -sT -p 1-100 127.0.0.1
time sudo nmap -sS -p 1-100 127.0.0.1

# Questions:
1. What was the time difference?
2. Is the difference significant for 100 ports?
```

### Exercise 4: Logging Experiment
```cmd
# Task: Check if scans are logged
# On Linux, check connection logs:
sudo tail -f /var/log/syslog

# In another terminal, run:
nmap -sT 127.0.0.1
sudo nmap -sS 127.0.0.1

# Questions:
1. Which scan appeared in logs?
2. What information was logged?
```

## Common Issues and Solutions

### Issue 1: "You requested a scan type which requires root privileges"
```cmd
# Solution 1: Use sudo
sudo nmap -sS target

# Solution 2: Use Connect scan instead
nmap -sT target

# Solution 3: Set capabilities (Linux)
sudo setcap cap_net_raw,cap_net_admin+eip $(which nmap)
```

### Issue 2: Inconsistent Results
```cmd
# If SYN and Connect show different results:
# Solution: Use version detection
sudo nmap -sS -sV target
nmap -sT -sV target

# This may reveal why ports appear differently
```

### Issue 3: Firewall Blocking
```cmd
# If scans are being blocked:
# Try different ports
sudo nmap -sS -p 443,8443,9443 target

# Use different timing
sudo nmap -sS -T2 target

# Try through proxy
nmap -sT --proxies http://proxy:8080 target
```

## Best Practices

### When to Use Connect Scan (-sT)
1. **Internal network assessments** - Less need for stealth
2. **When you lack privileges** - No root/admin access
3. **Compliance scanning** - Some standards require full connections
4. **Testing application connectivity** - Simulates real client behavior

### When to Use SYN Scan (-sS)
1. **External penetration tests** - Stealth is important
2. **Security assessments** - Avoid logging where possible
3. **Large network scans** - Speed matters
4. **When you have privileges** - Always use if available

### General Guidelines
1. **Start with SYN** if you have privileges
2. **Validate with Connect** if results seem suspicious
3. **Consider the environment** - Corporate vs. home networks
4. **Document your choice** - Note why you selected each method

## Security Considerations

### Ethical Use
- Only scan systems you own or have permission to test
- Be aware that SYN scans may be considered more intrusive
- Consider the target's security posture
- Have proper authorization documentation

### Detection Risks
- Modern IDS/IPS systems detect both scan types
- Enterprise firewalls often log connection attempts
- Some cloud providers flag scanning activity
- Rate limiting can trigger blocks

### Defensive Perspective
As a defender, understand that:
1. Both scan types can be detected with proper monitoring
2. SYN scans are harder to trace back to source
3. Regular scanning of your own systems helps understand what attackers see
4. Logging and alerting should cover both scan types

## Next Steps

Tomorrow (Day 07) will cover:
- UDP scanning techniques
- Understanding UDP vs TCP differences
- Practical UDP scanning exercises
- Interpreting UDP scan results

## Reflection Questions

1. Which scan type felt more appropriate for your testing environment?
2. Did you encounter any differences in results between the two methods?
3. How does understanding these techniques help in security assessments?
4. What ethical considerations came to mind while testing these scans?

## Need Help?

- Review TCP fundamentals if concepts are unclear
- Check Nmap documentation: `man nmap`
- Practice on isolated lab systems
- Join online security communities for guidance

**🎉 Congratulations on completing Day 06!** You've now mastered the two fundamental TCP scanning techniques that form the backbone of network reconnaissance. Understanding when and why to use each method will make you a more effective security professional.

*"Knowing your tools is the first step; knowing when to use each tool is mastery."*