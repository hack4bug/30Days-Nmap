# Day 04 - Basic Host Discovery

## Day Overview
Welcome to Day 4! Today, we'll learn about **Host Discovery** - the crucial first step in network reconnaissance. Host discovery is about finding "alive" or "responsive" devices on a network before performing detailed scanning.

---

## Learning Objectives
By completing Day 04, you will be able to:
- Understand what host discovery is and why it's important
- Use different host discovery techniques
- Perform ping sweeps and network discovery
- Interpret host discovery results
- Choose appropriate discovery methods for different scenarios

---

## What is Host Discovery?

### **Basic Concept**
Host discovery (also called "host enumeration" or "ping sweeping") is the process of identifying which IP addresses in a network range correspond to active, reachable devices. It answers the question: **"What devices are currently online and responsive?"**

### **Why Host Discovery Matters**
```cmd
Without Host Discovery:
- Scan 254 IPs in a /24 network
- Waste time scanning dead/unresponsive hosts
- Generate unnecessary network traffic
- Longer scan completion time

With Host Discovery:
- Identify only 45 active hosts
- Scan only those 45 hosts
- Faster results
- Less network noise
```

---

## Host Discovery Techniques

Nmap offers multiple host discovery methods. Let's explore each:

### **1. ICMP Echo Request (Standard Ping)**

**What it is:** The classic "ping" method everyone knows
```cmd
# Basic ICMP ping
nmap -PE 192.168.1.0/24

# Or simply (ICMP is often default)
nmap -sn 192.168.1.0/24
```

**How it works:**
1. Sends ICMP Echo Request packets
2. Waits for ICMP Echo Reply packets
3. If reply received → Host is alive

**Pros:**
- Simple and reliable
- Works on most networks
- Fast and lightweight

**Cons:**
- Often blocked by firewalls
- Some hosts disable ICMP responses
- Not stealthy

### **2. ICMP Timestamp Request**

**What it is:** Alternative ICMP method when echo is blocked
```cmd
nmap -PP 192.168.1.0/24
```

**How it works:**
1. Sends ICMP Timestamp Request
2. Receives ICMP Timestamp Reply
3. Uses different ICMP type than standard ping

**Use Case:** When standard ping (ICMP Echo) is blocked but other ICMP types are allowed

### **3. ICMP Address Mask Request**

**What it is:** Another ICMP alternative method
```cmd
nmap -PM 192.168.1.0/24
```

**How it works:**
1. Sends ICMP Address Mask Request
2. Receives ICMP Address Mask Reply
3. Very old method, rarely used today

**Note:** Most modern systems don't respond to this by default

### **4. TCP SYN Ping**

**What it is:** Uses TCP connection attempts for discovery
```cmd
# To port 80 (HTTP)
nmap -PS80 192.168.1.0/24

# To port 443 (HTTPS)
nmap -PS443 192.168.1.0/24

# Multiple ports
nmap -PS22,80,443 192.168.1.0/24
```

**How it works:**
1. Sends TCP SYN packet to specified port
2. If SYN-ACK received → Host is alive (then sends RST to close)
3. If RST received → Host is alive but port closed
4. If no response → Host may be dead or filtered

**Advantages:**
- Often works when ICMP is blocked
- More stealthy than ICMP
- Can target common open ports

### **5. TCP ACK Ping**

**What it is:** Uses TCP ACK packets for discovery
```cmd
nmap -PA80 192.168.1.0/24
```

**How it works:**
1. Sends TCP ACK packet
2. If RST received → Host is alive
3. Different response than SYN ping

**Use Case:** Testing firewall rules and stateful inspection

### **6. UDP Ping**

**What it is:** Uses UDP packets for discovery
```cmd
nmap -PU53 192.168.1.0/24  # DNS port
nmap -PU161 192.168.1.0/24 # SNMP port
```

**How it works:**
1. Sends UDP packet to closed port
2. If ICMP Port Unreachable received → Host is alive
3. If no response → Host may be dead or filtered

**Advantages:**
- Bypasses some firewalls
- Works when both ICMP and TCP are blocked

### **7. ARP Ping (Local Networks Only)**

**What it is:** Layer 2 discovery for local networks
```cmd
# Automatically used for local networks
nmap -PR 192.168.1.0/24
```

**How it works:**
1. Sends ARP Request "Who has IP X?"
2. Receives ARP Reply "IP X is at MAC Y"
3. Fastest and most reliable for local networks

**Important:** Only works on same subnet/LAN

---

## Practical Host Discovery Commands

### **Basic Discovery Scenarios**

**Scenario 1: Simple Network Scan**
```cmd
# Basic ping sweep of entire network
nmap -sn 192.168.1.0/24

# Output shows:
# - IP addresses that responded
# - MAC addresses (if local)
# - Hostnames (if DNS resolution on)
```

**Scenario 2: Stealthy Discovery**
```cmd
# Use multiple methods together
nmap -sn -PE -PS22,80,443 -PU53 192.168.1.0/24

# Combines:
# - ICMP Echo
# - TCP SYN to common ports
# - UDP to DNS port
```

**Scenario 3: Fast Large Network Scan**
```cmd
# Scan Class B network (65,534 hosts)
nmap -sn -T4 10.0.0.0/16

# -T4 for faster timing
# Only host discovery (-sn)
```

**Scenario 4: List Scan (No Packets Sent)**
```cmd
# Just list targets, don't actually scan
nmap -sL 192.168.1.0/24

# Useful for:
# - Creating target lists
# - Testing IP ranges
# - Dry runs before actual scanning
```

---

## Understanding Discovery Results

### **Sample Output Analysis**
```cmd
$ nmap -sn 192.168.1.0/24
Starting Nmap 7.93 ( https://nmap.org ) at 2024-01-15 10:00 EST
Nmap scan report for 192.168.1.1
Host is up (0.005s latency).
MAC Address: AA:BB:CC:DD:EE:FF (Router Manufacturer)
Nmap scan report for 192.168.1.100
Host is up (0.002s latency).
MAC Address: 11:22:33:44:55:66 (Computer Manufacturer)
Nmap scan report for 192.168.1.101
Host is up (0.003s latency).
Nmap scan report for 192.168.1.150
Host is up (0.001s latency).
MAC Address: 77:88:99:AA:BB:CC (Phone Manufacturer)
Nmap done: 256 IP addresses (4 hosts up) scanned in 3.45 seconds
```

### **What the Output Tells Us:**
1. **4 hosts found alive** out of 256 possible
2. **Router identified**: 192.168.1.1 (common gateway)
3. **Devices identified by MAC**: Can infer device types
4. **Response times**: All under 0.01s (local network)
5. **Scan duration**: 3.45 seconds for entire /24 network

### **Common Discovery Output Patterns**

**Local Network (Home/Office):**
- 5-50 hosts typically
- Gateway router (usually .1 or .254)
- Computers, phones, printers, IoT devices
- Fast response times (<0.01s)

**External Network Scan:**
- Fewer responsive hosts
- Slower response times
- No MAC addresses (Layer 3 only)
- More filtered/unresponsive hosts

---

## Advanced Discovery Techniques

### **Combining Multiple Methods**
```cmd
# Comprehensive discovery
nmap -sn -PE -PP -PS22,80,443 -PA80 -PU53 192.168.1.0/24

# This tries:
# 1. ICMP Echo (PE)
# 2. ICMP Timestamp (PP)
# 3. TCP SYN to common ports (PS)
# 4. TCP ACK to port 80 (PA)
# 5. UDP to DNS port (PU)
```

**Why Combine Methods:**
- Increases detection rate
- Bypasses different firewall rules
- More reliable results
- Understands network filtering better

### **Discovery with Output Control**
```cmd
# Save results to file
nmap -sn -oG discovery_results.txt 192.168.1.0/24

# XML output for processing
nmap -sn -oX discovery.xml 192.168.1.0/24

# Only show hosts that are up
nmap -sn --open 192.168.1.0/24
```

### **Timing and Performance Tuning**
```cmd
# Fast scan (aggressive timing)
nmap -sn -T4 192.168.1.0/24

# Slow scan (stealthy)
nmap -sn -T2 192.168.1.0/24

# Paranoid scan (very slow)
nmap -sn -T0 192.168.1.0/24

# Custom timing
nmap -sn --max-rtt-timeout 100ms --max-retries 1 192.168.1.0/24
```

---

## How Nmap Decides "Host is Up"

### **The Logic Behind Discovery**
```cmd
Nmap's decision process:
1. Send multiple probe types
2. Wait for ANY positive response
3. Mark host "up" if any response received
4. Mark host "down" if all probes fail

Important: A single positive response is enough!
```

### **Response Interpretation Table**

| Probe Type | Positive Response | Meaning |
|------------|-------------------|---------|
| ICMP Echo | Echo Reply | Host is up |
| TCP SYN | SYN-ACK | Host is up, port open |
| TCP SYN | RST | Host is up, port closed |
| TCP ACK | RST | Host is up |
| UDP | ICMP Port Unreachable | Host is up |
| ARP | ARP Reply | Host is up (local only) |
| Any | No Response | Host may be down |

---

## Practical Exercises

### **Exercise 1: Basic Network Discovery**
```cmd
# Task: Discover all devices on your local network
# Step 1: Find your network range
ipconfig (Windows) or ifconfig (Linux/macOS)

# Step 2: Perform discovery
nmap -sn [your-network]/24

# Step 3: Document findings
1. How many hosts found?
2. Identify likely device types
3. Note response times
```

### **Exercise 2: Method Comparison**
```cmd
# Compare different discovery methods
# Method 1: ICMP only
nmap -sn -PE 192.168.1.0/24

# Method 2: TCP only
nmap -sn -PS22,80,443 192.168.1.0/24

# Method 3: Combined
nmap -sn -PE -PS22,80,443 192.168.1.0/24

# Questions:
1. Which method found the most hosts?
2. Which was fastest?
3. Any hosts only found by specific methods?
```

### **Exercise 3: Discovery Analysis**
```cmd
# Perform discovery and analyze results
nmap -sn -oG discovery.txt 192.168.1.0/24

# Use grep to analyze
grep "Host is up" discovery.txt | wc -l
grep "MAC Address" discovery.txt
grep "latency" discovery.txt | sort -nk5

# Create summary report
```

### **Exercise 4: External Discovery**
```cmd
# Note: ONLY scan networks you own or have permission!
# Use your home network or a test lab

# Discover external-facing devices
# (Assuming you have a test environment)
nmap -sn -PE -PS80,443 [test-network]

# Compare with internal discovery
# How do results differ?
```

---

## Discovery Strategy Cheat Sheet

### **By Network Type**

**Home/Small Office Network:**
```cmd
# Fast and comprehensive
nmap -sn -PR -PE 192.168.1.0/24
# -PR: ARP ping (fast local)
# -PE: ICMP echo (backup)
```

**Corporate Network:**
```cmd
# Stealthy and thorough
nmap -sn -PS22,80,443,3389 -PU53 -T2 10.0.0.0/24
# -PS: Common corporate ports
# -PU: DNS for internal resolution
# -T2: Polite timing
```

**External/Internet Facing:**
```cmd
# Focused on common services
nmap -sn -PS80,443,22 -PE --max-retries 1 203.0.113.0/24
# Common internet ports only
# Reduced retries for speed
```

**Highly Restricted Network:**
```cmd
# Try everything stealthily
nmap -sn -PE -PP -PM -PS21,22,23,25,80,110,443 -PA80 -PU53 -T1
# Multiple protocol attempts
# Very slow timing
```

### **By Objective**

**Quick Inventory:**
```cmd
nmap -sn -T4 --max-rtt-timeout 100ms [network]
```

**Stealth Reconnaissance:**
```cmd
nmap -sn -PS80,443 -T2 --max-retries 0 [network]
```

**Comprehensive Discovery:**
```cmd
nmap -sn -PE -PP -PM -PS1-100 -PA80 -PU53 [network]
```

---

## Important Considerations

### **Ethical and Legal**
1. **Permission First**: Only scan networks you own or have explicit permission
2. **Scope Definition**: Know exactly what you're allowed to scan
3. **Impact Awareness**: Discovery scans can still trigger security alerts
4. **Documentation**: Keep records of authorization and scan parameters

### **Technical Limitations**
1. **Firewall Evasion**: Modern firewalls can detect and block discovery attempts
2. **Network Impact**: Even discovery scans generate traffic
3. **False Positives/Negatives**: No method is 100% accurate
4. **Timing Issues**: Slow networks may cause timeout problems

### **Best Practices**
1. **Start Small**: Test on a few hosts before large scans
2. **Monitor Responses**: Watch for any negative impact
3. **Use Appropriate Timing**: Match timing to network conditions
4. **Document Everything**: Record methods used and results obtained

---

## Advanced Topics Preview

### **What's Next After Discovery?**
Once you've found alive hosts, you typically proceed to:
1. **Port Scanning**: What services are running?
2. **Version Detection**: What software versions?
3. **OS Fingerprinting**: What operating systems?
4. **Vulnerability Assessment**: Any known weaknesses?

### **Discovery Integration**
Host discovery is often combined with other scans:
```cmd
# Discovery + port scan in one command
nmap -sS -sV [network]

# But better practice is:
# Step 1: Discovery
nmap -sn -oG alive_hosts.txt [network]

# Step 2: Port scan only alive hosts
nmap -sS -sV -iL alive_hosts.txt
```

---

## Additional Resources

### **Nmap Documentation**
- `man nmap` - Host discovery section
- `nmap -h` - Discovery option list
- https://nmap.org/book/man-host-discovery.html

### **Learning Tools**
- Wireshark for packet analysis during discovery
- Network simulation tools (GNS3, EVE-NG)
- Practice labs (TryHackMe, HackTheBox)

### **Reference Materials**
- RFC 792 (ICMP Protocol)
- RFC 793 (TCP Protocol)
- RFC 826 (ARP Protocol)

---

## Next Steps
Tomorrow (Day 05) will cover:
- Basic port scanning techniques
- Understanding port states
- Common scanning patterns
- Hands-on port scanning exercises

---

## Pro Tips

1. **Use List Scan First**: `nmap -sL` to verify target ranges
2. **Combine Methods**: No single method works everywhere
3. **Adjust Timing**: Fast for internal, slow for external
4. **Save Results**: Always save discovery results for reference
5. **Verify with Multiple Tools**: Cross-check with ping, arp, etc.

---

## Reflection Questions

1. Which discovery method worked best on your network?
2. Did you find any unexpected devices?
3. How does discovery help in security assessments?
4. What challenges did you face during discovery?

---

## Need Help?

- Check Nmap output for clues
- Use `-v` or `-vv` for verbose output
- Consult online forums and communities
- Practice in isolated lab environments

---

**🎉 Congratulations on completing Day 04!** You've mastered the crucial first step in network reconnaissance. Tomorrow, we'll build on this by learning how to scan the services running on discovered hosts.

---
*"First find the doors, then learn what's behind them. Today you've learned how to find the doors."*