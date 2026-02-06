# Day 07 - UDP Scanning

## Day Overview
Welcome to Day 7! Today, we explore **UDP Scanning** - a crucial but often overlooked aspect of network reconnaissance. While most services use TCP, many important network services run over UDP, making UDP scanning essential for comprehensive security assessments.

## Learning Objectives
By completing Day 07, you will be able to:
- Understand UDP protocol fundamentals
- Perform UDP port scans using Nmap
- Interpret UDP scan results correctly
- Identify common UDP services and their ports
- Handle UDP scanning challenges and limitations

## Understanding UDP Protocol

### TCP vs UDP - Key Differences

```cmd
TCP (Transmission Control Protocol):
- Connection-oriented (handshake required)
- Reliable delivery (retransmits lost packets)
- Ordered data delivery
- Flow control and congestion management
- Example: Web browsing, email, file transfers

UDP (User Datagram Protocol):
- Connectionless (no handshake)
- Unreliable delivery (no retransmission)
- Unordered data delivery
- Faster and simpler
- Example: DNS queries, video streaming, VoIP
```

### Why UDP Scanning Matters
1. **Critical services use UDP**: DNS (53), DHCP (67,68), SNMP (161), NTP (123)
2. **Different security posture**: UDP services often have weaker security
3. **Different discovery**: UDP ports behave differently than TCP ports
4. **Comprehensive assessment**: Missing UDP services means incomplete security picture

## UDP Scan Basics

### Basic UDP Scan Command

```cmd
# Basic UDP scan
sudo nmap -sU 192.168.1.1

# UDP scan requires root/admin privileges
# Default scans top 1000 UDP ports
```

### Why Root Privileges are Required
UDP scanning requires raw socket access to:
- Send custom UDP packets
- Receive ICMP responses
- Handle different response types
- Without privileges, Nmap cannot perform proper UDP scans

## How UDP Scanning Works

### The UDP Scan Process

```cmd
UDP Scan Mechanism:
Step 1: Nmap → Target: UDP packet to port
Step 2: If port CLOSED: Target → Nmap: ICMP Port Unreachable
Step 3: If port OPEN: No response (usually) or service response
Step 4: If FILTERED: No response (firewall blocking)

Challenge: No response could mean OPEN or FILTERED!
```

### Understanding UDP Responses

```cmd
UDP Response Types:

1. ICMP Port Unreachable (Type 3, Code 3)
   - Means: Port is CLOSED
   - Certainty: High

2. UDP Response from Service
   - Means: Port is OPEN
   - Certainty: High
   - Example: DNS server responds to query

3. No Response
   - Could mean: OPEN (service doesn't respond)
   - Could mean: FILTERED (firewall blocking)
   - Certainty: Low - requires further investigation
```

## Common UDP Scan Commands

### Basic UDP Scans

```cmd
# Scan single target
sudo nmap -sU 192.168.1.1

# Scan specific UDP ports
sudo nmap -sU -p 53,67,68,161,162 192.168.1.1

# Scan UDP port range
sudo nmap -sU -p 1-100 192.168.1.1
```

### UDP Scan with Timing Control

```cmd
# Faster UDP scan (less reliable)
sudo nmap -sU -T4 192.168.1.1

# Slower, more reliable UDP scan
sudo nmap -sU -T2 192.168.1.1

# Maximum retries for reliability
sudo nmap -sU --max-retries 3 192.168.1.1
```

### Comprehensive UDP Scan

```cmd
# UDP scan with version detection
sudo nmap -sU -sV 192.168.1.1

# UDP scan with NSE scripts
sudo nmap -sU --script=default 192.168.1.1

# Combined TCP and UDP scan
sudo nmap -sS -sU 192.168.1.1
```

## Important UDP Ports to Know

### Critical UDP Services

```cmd
Port 53:   DNS (Domain Name System)
Port 67:   DHCP Server
Port 68:   DHCP Client
Port 69:   TFTP (Trivial File Transfer)
Port 123:  NTP (Network Time Protocol)
Port 137:  NetBIOS Name Service
Port 138:  NetBIOS Datagram Service
Port 161:  SNMP (Simple Network Management)
Port 162:  SNMP Trap
Port 500:  ISAKMP/IKE (VPN)
Port 514:  Syslog
Port 520:  RIP (Routing Information Protocol)
Port 1900: UPnP (Universal Plug and Play)
Port 5353: mDNS (Multicast DNS)
Port 5355: LLMNR (Link-Local Multicast Name Resolution)
```

### Security Significance of Common UDP Ports

**Port 53 (DNS)**: Critical infrastructure service, often misconfigured
**Port 69 (TFTP)**: No authentication, can expose sensitive files
**Port 161 (SNMP)**: Default community strings, information disclosure
**Port 500 (ISAKMP)**: VPN configuration, potential vulnerabilities
**Port 1900 (UPnP)**: Device discovery, often exploited in attacks

## UDP Scan Results Interpretation

### Sample UDP Scan Output

```cmd
$ sudo nmap -sU 192.168.1.1
Starting Nmap 7.93 ( https://nmap.org ) at 2024-01-15 10:00 EST
Nmap scan report for router.local (192.168.1.1)
Host is up (0.005s latency).
Not shown: 996 open|filtered udp ports (no-response)
PORT      STATE         SERVICE
53/udp    open          domain
67/udp    open|filtered dhcps
68/udp    open|filtered dhcpc
161/udp   open          snmp
500/udp   open|filtered isakmp

Nmap done: 1 IP address (1 host up) scanned in 105.32 seconds
```

### Understanding the Output

**Key Observations:**
1. **Scan duration**: 105.32 seconds (much slower than TCP)
2. **Port states**: Mixed "open" and "open|filtered"
3. **No-response ports**: 996 ports showed no response
4. **Definite opens**: Port 53 (DNS) and 161 (SNMP) confirmed open

### Common UDP Port States

**open**: Service responded (e.g., DNS query got reply)
**open|filtered**: No response - could be open or filtered
**closed**: ICMP Port Unreachable received
**filtered**: ICMP unreachable filtered (rare)

## UDP Scanning Challenges

### Why UDP Scans Are Slow

```cmd
Reasons for UDP Scan Slowness:

1. No response timing: Wait for possible ICMP response
2. Retransmissions: Nmap retries when no response
3. Rate limiting: Avoid flooding target with UDP packets
4. Firewall delays: Some firewalls delay ICMP responses

Default timeout: 5 seconds per port (adjustable)
```

### Dealing with No Responses

```cmd
# Strategies for handling open|filtered results:

1. Use version detection
   sudo nmap -sU -sV -p 53,161 target

2. Increase timeout for specific ports
   sudo nmap -sU --host-timeout 30m target

3. Use service-specific probes
   sudo nmap -sU --script dns-brute target -p 53

4. Test with actual client tools
   dig @target example.com  # Test DNS
   snmpwalk -v1 -c public target  # Test SNMP
```

## Advanced UDP Scanning Techniques

### UDP Scan with Version Detection

```cmd
# Version detection on UDP ports
sudo nmap -sU -sV target

# Version detection on specific UDP ports
sudo nmap -sU -sV -p 53,161,500 target

# Aggressive version detection
sudo nmap -sU -sV --version-intensity 9 target
```

### UDP Scan with NSE Scripts

```cmd
# Run default NSE scripts on UDP ports
sudo nmap -sU --script=default target

# Run specific UDP scripts
sudo nmap -sU --script=dns*,snmp* target

# Run vulnerability scripts on UDP ports
sudo nmap -sU --script=vuln target
```

### Performance Optimization

```cmd
# Increase scan speed
sudo nmap -sU -T4 --max-rtt-timeout 100ms target

# Reduce retries for speed
sudo nmap -sU --max-retries 0 target

# Scan fewer ports but faster
sudo nmap -sU -F target  # Top 100 UDP ports

# Parallel scanning
sudo nmap -sU --min-parallelism 10 target
```

## Practical UDP Scanning Scenarios

### Scenario 1: DNS Server Assessment

```cmd
# Comprehensive DNS server check
sudo nmap -sU -sV -p 53 --script=dns* dns.server.com

# Check for DNS amplification vulnerability
sudo nmap -sU -p 53 --script=dns-recursion dns.server.com

# Test DNS zone transfer
sudo nmap -sU -p 53 --script=dns-zone-transfer dns.server.com
```

### Scenario 2: Network Device Discovery

```cmd
# Scan for network devices via SNMP
sudo nmap -sU -p 161 --script=snmp* 192.168.1.0/24

# Check for default SNMP community strings
sudo nmap -sU -p 161 --script=snmp-brute target

# Enumerate SNMP information
sudo nmap -sU -p 161 --script=snmp-info target
```

### Scenario 3: Time Server Check

```cmd
# Scan for NTP servers
sudo nmap -sU -p 123 --script=ntp* target

# Check NTP monlist vulnerability
sudo nmap -sU -p 123 --script=ntp-monlist target

# Enumerate NTP information
sudo nmap -sU -p 123 --script=ntp-info target
```

## Practice Exercises

### Exercise 1: Basic UDP Scan

```cmd
# Task: Perform UDP scan on your router/gateway
# Find your default gateway:
# Windows: ipconfig
# Linux/Mac: ip route | grep default

# Perform UDP scan:
sudo nmap -sU [your-gateway-ip]

# Questions:
1. How many UDP ports were found?
2. Which services are running on UDP?
3. How long did the scan take?
```

### Exercise 2: DNS Server Investigation

```cmd
# Task: Investigate DNS server on your network
# Scan your DNS server (often same as gateway):
sudo nmap -sU -sV -p 53 --script=dns* [dns-server-ip]

# Questions:
1. What DNS software and version is running?
2. Does it support recursive queries?
3. Are there any security issues detected?
```

### Exercise 3: UDP vs TCP Comparison

```cmd
# Task: Compare UDP and TCP on same target
# TCP scan:
sudo nmap -sS target

# UDP scan:
sudo nmap -sU target

# Questions:
1. Which found more open ports?
2. How do scan durations compare?
3. Are there services running on both TCP and UDP?
```

### Exercise 4: Service Validation

```cmd
# Task: Validate UDP scan results with client tools
# For each open UDP port found, test with appropriate client:
# DNS (53): dig @target example.com
# NTP (123): ntpdate -q target
# SNMP (161): snmpwalk -v1 -c public target

# Questions:
1. Do client tools confirm Nmap findings?
2. Were there false positives in the scan?
3. What additional information did client tools provide?
```

## UDP Security Considerations

### Common UDP Vulnerabilities

1. **DNS Amplification**: UDP port 53 used in DDoS attacks
2. **SNMP Default Credentials**: Community strings like "public"
3. **NTP Amplification**: UDP port 123 used in DDoS
4. **TFTP Information Disclosure**: No authentication on port 69
5. **NetBIOS Enumeration**: Ports 137-138 expose system information

### Defensive Measures for UDP Services

```cmd
Best practices for securing UDP services:

1. Firewall rules: Restrict UDP access where not needed
2. Service hardening: Disable unnecessary features
3. Rate limiting: Prevent amplification attacks
4. Monitoring: Alert on unusual UDP traffic
5. Regular scanning: Check your own UDP exposure
```

## Troubleshooting UDP Scans

### Common Issues and Solutions

**Issue: Scan takes too long**
```cmd
# Solutions:
# Reduce port range
sudo nmap -sU -F target  # Top 100 ports

# Increase timeout
sudo nmap -sU --max-rtt-timeout 500ms target

# Skip no-response ports faster
sudo nmap -sU --max-retries 0 target
```

**Issue: No results or all ports open|filtered**
```cmd
# Solutions:
# Check firewall rules
# Verify network connectivity
# Try different timing templates
sudo nmap -sU -T2 target  # Slower, more reliable

# Use specific port probes
sudo nmap -sU -p 53,161 --script=default target
```

**Issue: Permission denied**
```cmd
# Solution: Use sudo/administrator privileges
sudo nmap -sU target

# If sudo not available, UDP scanning is limited
# Consider alternative tools or elevated privileges
```

## Best Practices for UDP Scanning

### Planning Your UDP Scan

1. **Start with common ports**: Focus on critical UDP services first
2. **Consider scan timing**: UDP scans are slow - plan accordingly
3. **Use appropriate privileges**: Root/admin access is required
4. **Validate findings**: Confirm with client tools when possible

### Scan Optimization Tips

```cmd
# Efficient UDP scanning strategy:

# Step 1: Quick common ports scan
sudo nmap -sU -F target

# Step 2: Focus on interesting findings
sudo nmap -sU -sV -p [found-ports] target

# Step 3: Run targeted NSE scripts
sudo nmap -sU --script=[relevant-scripts] target -p [ports]
```

### Documentation and Reporting

1. **Record scan parameters**: Note timing, ports, and options used
2. **Document findings**: Include both open and open|filtered ports
3. **Note limitations**: UDP scanning has inherent uncertainties
4. **Recommend follow-up**: Suggest manual validation for critical findings

## Next Steps

Tomorrow (Day 08) will cover:
- Service version detection techniques
- Banner grabbing and service fingerprinting
- Identifying software versions and vulnerabilities
- Version detection best practices

## Reflection Questions

1. What was the most surprising UDP service you discovered?
2. How does UDP scanning differ from TCP scanning in practice?
3. What challenges did you face with UDP scanning?
4. Why is understanding UDP services important for security?

## Need Help?

- Review UDP protocol basics if concepts are unclear
- Check Nmap UDP scanning documentation: `man nmap`
- Use `-v` or `-vv` flags for verbose output during scans
- Practice on isolated lab systems to understand behavior

**🎉 Congratulations on completing Day 07!** You've now mastered UDP scanning, completing your understanding of both major transport protocols. This knowledge is essential for comprehensive network security assessments.

*"TCP shows you the front door; UDP reveals the hidden entrances. A true security professional checks both."*