# Day 12 - Using NSE Discovery Scripts

## Day Overview
Welcome to Day 12! Today, we dive deep into **NSE Discovery Scripts** - the powerful automation tools that turn Nmap from a simple port scanner into an intelligent network reconnaissance platform. Discovery scripts help you automatically find hosts, services, and information that regular scanning might miss.

## Learning Objectives
By completing Day 12, you will be able to:
- Understand different types of NSE discovery scripts
- Use scripts for network host discovery beyond basic ping
- Automate service enumeration and information gathering
- Discover network shares, users, and system information
- Apply discovery scripts in real-world scenarios

## What Are Discovery Scripts?

### Basic Definition
Discovery scripts are NSE scripts specifically designed to find and enumerate network resources, hosts, services, and information automatically. They go beyond basic port scanning to uncover hidden network elements.

### Why Discovery Scripts Matter

```cmd
Regular Nmap Scanning:
Finds: Open ports on KNOWN hosts
Misses: Hosts that don't respond to standard probes
       Hidden services on unusual ports
       Network relationships and dependencies
       Additional network information

NSE Discovery Scripts:
Finds: ALL of the above, plus:
       - Hosts that ignore normal probes
       - Services on non-standard ports
       - Network shares and resources
       - System information and configurations
       - Network topology relationships
```

## Major Categories of Discovery Scripts

### Host Discovery Scripts
These scripts find hosts that regular Nmap scanning might miss.

### Service Discovery Scripts
These scripts identify services and gather detailed information about them.

### Network Information Scripts
These scripts discover network configuration, shares, users, and other information.

### Topology Discovery Scripts
These scripts map relationships between systems and network paths.

## Essential Discovery Scripts You Should Know

### 1. broadcast-ping
Discovers hosts using various broadcast ping methods.

```cmd
# Discover hosts on local network using broadcast
sudo nmap --script broadcast-ping

# Output shows:
- Hosts responding to broadcast pings
- MAC addresses and manufacturers
- Response times and methods

# Useful for:
- Finding devices that ignore direct probes
- Network inventory without intrusive scanning
- Discovering IoT and embedded devices
```

### 2. broadcast-dhcp-discover
Discovers DHCP servers and gets network information.

```cmd
# Find DHCP servers and get lease information
sudo nmap --script broadcast-dhcp-discover

# Information obtained:
- DHCP server IP addresses
- Offered IP ranges
- DNS servers
- Gateway addresses
- Lease times
- Domain names

# Real-world use:
- Network configuration discovery
- Identifying rogue DHCP servers
- Understanding network segmentation
```

### 3. targets-sniffer
Passively discovers hosts by sniffing network traffic.

```cmd
# Passive host discovery (no packets sent)
sudo nmap --script targets-sniffer

# How it works:
- Listens on network interface
- Analyzes passing traffic
- Identifies active hosts
- Completely stealthy

# Best for:
- Stealth reconnaissance
- Network monitoring
- Identifying all active devices
```

### 4. dns-brute
Brute-forces DNS subdomains to discover hosts.

```cmd
# Discover subdomains via DNS brute-force
nmap --script dns-brute target-domain.com

# Common discoveries:
- www.target-domain.com
- mail.target-domain.com
- ftp.target-domain.com
- admin.target-domain.com
- dev.target-domain.com
- test.target-domain.com

# Customization:
nmap --script dns-brute --script-args dns-brute.domain=example.com,dns-brute.threads=20
```

### 5. smb-os-discovery
Discovers Windows/SMB system information.

```cmd
# Get detailed Windows system information
nmap --script smb-os-discovery -p 445 target

# Information revealed:
- Operating system name and version
- Computer name
- Domain/workgroup name
- System time
- Forest/domain functional level
- Signing requirements

# Security implications:
- Identifies Windows versions for targeting
- Reveals domain structure
- Shows patch level information
```

### 6. smb-enum-shares
Enumerates SMB shares on Windows systems.

```cmd
# Discover shared folders and permissions
nmap --script smb-enum-shares -p 445 target

# Finds:
- Share names (ADMIN$, C$, IPC$, etc.)
- Share types (Disk, Printer, IPC, etc.)
- Share comments/descriptions
- Current user access permissions

# Security assessment:
- Identify sensitive shares
- Check for overly permissive shares
- Find hidden administrative shares
```

### 7. http-enum
Discovers web application directories and files.

```cmd
# Find common web directories and files
nmap --script http-enum -p 80,443,8080,8443 target

# Commonly finds:
- /admin/ - Administration interfaces
- /backup/ - Backup directories
- /config/ - Configuration files
- /phpinfo.php - PHP information
- /robots.txt - Directory hints
- /test/ - Testing directories

# Custom wordlists:
nmap --script http-enum --script-args http-enum.fingerprintfile=my_wordlist.txt
```

### 8. snmp-info
Gathers information from SNMP-enabled devices.

```cmd
# Get system information via SNMP
nmap --script snmp-info -sU -p 161 target

# Information obtained:
- System description
- Contact information
- Location
- Uptime
- Services running
- Network interfaces
- Hardware details

# Common on:
- Network devices (routers, switches)
- Printers
- Servers
- IoT devices
```

### 9. ms-sql-info
Discovers Microsoft SQL Server information.

```cmd
# Get SQL Server details
nmap --script ms-sql-info -p 1433 target

# Information gathered:
- SQL Server version
- Instance names
- Named pipe names
- TCP port numbers
- Clustered status
- Database names

# Useful for:
- Database inventory
- Version checking for vulnerabilities
- Configuration review
```

### 10. nbstat
Discovers NetBIOS information from Windows systems.

```cmd
# Get NetBIOS name information
nmap -sU -p 137 --script nbstat target

# Information revealed:
- NetBIOS computer name
- Domain name
- MAC address
- Currently logged in users
- Server services running
- Workstation services

# Legacy but still useful:
- Older Windows networks
- Some embedded systems
- Network printers
```

## Advanced Discovery Script Combinations

### Complete Network Discovery Suite

```cmd
# Comprehensive network discovery in one command
sudo nmap --script "broadcast-* and targets-*" --script-args newtargets

# This runs:
- broadcast-ping
- broadcast-dhcp-discover
- targets-sniffer
- And adds discovered hosts to scan list

# Result: Maximum host discovery with minimal active probing
```

### Windows Network Enumeration

```cmd
# Complete Windows network information gathering
nmap --script "smb-* and nbstat" -p 137,139,445 target

# Gets:
- SMB shares and permissions
- User accounts
- System information
- Domain details
- NetBIOS information

# Perfect for:
- Windows network audits
- Active Directory reconnaissance
- Share permission reviews
```

### Web Server Deep Discovery

```cmd
# Deep web application discovery
nmap --script "http-* and ssl-*" -p 80,443,8080,8443 target

# Discovers:
- Web technologies and frameworks
- SSL/TLS configuration
- Hidden directories and files
- Server headers and information
- Web application firewalls
- Load balancers
```

## Practical Discovery Script Scenarios

### Scenario 1: Complete Internal Network Audit

```cmd
# Step 1: Passive discovery (stealthy)
sudo nmap --script targets-sniffer --script-args 'targets-sniffer.timeout=300' -oA passive-discovery

# Step 2: Active discovery of found hosts
nmap -iL passive-discovery.gnmap --script smb-os-discovery,smb-enum-shares

# Step 3: Service discovery on all hosts
nmap -iL passive-discovery.gnmap -sV --script discovery

# This approach:
- Starts stealthy
- Validates with active scanning
- Gets comprehensive information
- Minimizes network impact initially
```

### Scenario 2: External Reconnaissance

```cmd
# External target discovery workflow
# Step 1: DNS-based discovery
nmap --script dns-brute --script-args dns-brute.domain=target.com -oA dns-discovery

# Step 2: Port scanning discovered hosts
nmap -iL dns-discovery.gnmap -F

# Step 3: Service enumeration on open ports
nmap -iL dns-discovery.gnmap -sV --script http-enum,ssh-hostkey

# Benefits:
- Discovers all public-facing systems
- Maps the external attack surface
- Identifies potential entry points
```

### Scenario 3: Incident Response Discovery

```cmd
# Rapid discovery during security incident
# Step 1: Quick network sweep
sudo nmap -T4 -sn 192.168.1.0/24 -oA quick-sweep

# Step 2: Critical service discovery on found hosts
nmap -iL quick-sweep.gnmap -p 22,80,443,445,3389 --script smb-os-discovery,ssh-hostkey

# Step 3: Check for suspicious services
nmap -iL quick-sweep.gnmap -p- --script unusual-port

# Incident response focus:
- Speed is critical
- Focus on essential services
- Look for anomalies
- Document everything quickly
```

## Discovery Script Arguments and Customization

### Common Script Arguments for Discovery

#### **newtargets** - Add Discovered Hosts to Scan
```cmd
# Automatically scan discovered hosts
nmap --script dns-brute --script-args newtargets target-domain.com

# New hosts found via DNS brute-force will be automatically scanned
# Great for automated reconnaissance pipelines
```

#### **max-newtargets** - Limit Discovered Hosts
```cmd
# Prevent scanning too many new hosts
nmap --script dns-brute --script-args 'newtargets,max-newtargets=50' target

# Limits automatic scanning to first 50 discovered hosts
# Prevents overwhelming scans on large networks
```

#### **domain** - Specify Domain for Discovery
```cmd
# Set domain for various discovery scripts
nmap --script dns-brute --script-args dns-brute.domain=example.com target
nmap --script smb-enum-domains --script-args smbdomain=example.com target

# Ensures scripts look in the right places
```

#### **timeout** - Control Script Execution Time
```cmd
# Set timeouts for long-running scripts
nmap --script targets-sniffer --script-args 'targets-sniffer.timeout=600' target

# 10-minute timeout for passive sniffing
# Prevents scripts from running indefinitely
```

### Custom Wordlists and Databases

```cmd
# Use custom wordlists for discovery
nmap --script http-enum --script-args http-enum.fingerprintfile=~/custom_wordlist.txt target

# Create your own discovery databases:
# 1. Common directory names for your organization
# 2. Project-specific terms
# 3. Employee names for username discovery
# 4. Department names for share discovery
```

## Network Topology Discovery

### Using traceroute-geolocation
Maps network paths with geographic information.

```cmd
# Trace route with geographic data
nmap --script traceroute-geolocation target.com

# Shows:
- Network path to target
- Geographic locations of hops
- ISP information
- Latency measurements

# Useful for:
- Understanding network architecture
- Identifying network bottlenecks
- Geographic targeting analysis
```

### Discovering Network Relationships

```cmd
# Scripts that reveal network relationships:
nmap --script smb-enum-domains -p 445 target  # Domain relationships
nmap --script dns-zone-transfer -p 53 target   # DNS zone relationships
nmap --script ldap-rootdse -p 389 target       # LDAP directory structure

# These help understand:
- Trust relationships between domains
- DNS infrastructure hierarchy
- Directory service architecture
- Administrative boundaries
```

## Specialized Discovery Scripts

### Industrial Control System (ICS) Discovery

```cmd
# Discover SCADA/ICS systems
nmap --script modbus-discover -p 502 target
nmap --script s7-enumerate -p 102 target

# Common ICS ports discovered:
- 502: Modbus
- 102: Siemens S7
- 20000: DNP3
- 44818: EtherNet/IP

# Critical for:
- Industrial network security
- OT (Operational Technology) assessments
- Critical infrastructure protection
```

### Database Discovery Suite

```cmd
# Comprehensive database discovery
nmap --script "oracle-* or mysql-* or ms-sql-* or pgsql-*" -p 1521,3306,1433,5432 target

# Discovers:
- Database types and versions
- Instance names and configurations
- Authentication methods
- Database objects and schemas

# Essential for:
- Database security assessments
- Compliance auditing
- Migration planning
```

### Cloud Service Discovery

```cmd
# Discover cloud services and configurations
nmap --script http-aws-* -p 80,443 target    # AWS metadata
nmap --script http-azure-* -p 80,443 target   # Azure services
nmap --script http-google-* -p 80,443 target  # Google Cloud

# Finds:
- Cloud metadata services
- Configuration endpoints
- Service APIs
- Management interfaces

# Cloud security focus:
- Misconfigured cloud services
- Exposed management interfaces
- Metadata service vulnerabilities
```

## Performance Optimization for Discovery Scripts

### Managing Script Execution Time

```cmd
# Control how long discovery scripts run
nmap --script dns-brute --script-timeout 5m target  # 5 minute timeout
nmap --script http-enum --script-args http-enum.timelimit=300 target  # 300 second limit

# Balance between:
- Completeness of discovery
- Time available for scanning
- Network performance impact
```

### Parallel Discovery Execution

```cmd
# Run multiple discovery scripts in parallel
nmap --script "dns-brute and http-enum" --min-parallelism 10 target

# Increases speed but also:
- Increases network traffic
- May overwhelm target
- Could trigger rate limiting

# Best for internal networks or with permission
```

### Targeted Discovery Strategies

```cmd
# Smart discovery - focus on what matters
# Instead of running all discovery scripts:
nmap --script "discovery and safe" target

# Run specific scripts based on found services:
# If port 445 open: smb-* scripts
# If port 80 open: http-* scripts
# If port 22 open: ssh-* scripts

# More efficient and less intrusive
```

## Security and Ethical Considerations

### Responsible Discovery Scanning

```cmd
# Discovery scripts can be intrusive - use responsibly:

1. Always get permission before scanning
2. Start with least intrusive methods first:
   nmap --script targets-sniffer  # Passive first
   Then: nmap --script broadcast-*  # Broadcast next
   Finally: nmap --script dns-brute  # Active last

3. Respect network policies and rate limits
4. Document what you discovered and how
5. Report findings responsibly to system owners
```

### Legal Considerations

- Discovery scripts may access information without authentication
- Some scripts may violate terms of service
- Cloud service discovery may breach cloud provider policies
- Always check authorization and legal boundaries

### Defensive Discovery Applications

As a defender, use discovery scripts to:

1. **Know Your Network**: Regular discovery scans for asset management
2. **Find Rogue Devices**: Detect unauthorized systems
3. **Configuration Validation**: Ensure systems are properly configured
4. **Change Detection**: Monitor for unauthorized changes
5. **Compliance Auditing**: Verify security controls are in place

## Troubleshooting Discovery Script Issues

### Common Problems and Solutions

**Issue: Scripts not finding expected information**
```cmd
# Solutions:
# Increase verbosity for debugging
nmap -v --script script-name target

# Check script arguments
nmap --script-help script-name

# Verify network connectivity to target
# Check firewall rules blocking probes
# Try different timing options
nmap -T2 --script script-name target
```

**Issue: Scripts taking too long**
```cmd
# Solutions:
# Set timeouts
nmap --script script-name --script-timeout 2m target

# Use smaller wordlists or databases
nmap --script http-enum --script-args http-enum.fingerprintfile=small_list.txt

# Run fewer scripts at once
# Focus on critical discovery first
```

**Issue: Scripts causing network issues**
```cmd
# Solutions:
# Limit packet rates
nmap --script script-name --max-rate 100 target

# Use serial execution
nmap --script script-name --max-parallelism 1 target

# Schedule during off-hours
# Monitor network during execution
```

## Best Practices for Discovery Script Usage

### Planning Your Discovery Campaign

```cmd
Recommended discovery workflow:

1. Passive Discovery (stealthy)
   sudo nmap --script targets-sniffer

2. Broadcast Discovery (low impact)
   sudo nmap --script broadcast-*

3. DNS-based Discovery (external)
   nmap --script dns-brute

4. Service-specific Discovery (targeted)
   nmap --script "smb-*" -p 445
   nmap --script "http-*" -p 80,443

5. Comprehensive Discovery (if permitted)
   nmap --script discovery
```

### Documentation and Reporting

```cmd
# Document your discovery process
# Save all outputs with timestamps:
nmap --script discovery -oA discovery-scan-$(date +%Y%m%d-%H%M%S) target

# Create discovery reports including:
- Scripts used and arguments
- Hosts discovered
- Services found
- Network information gathered
- Any anomalies or concerns
- Recommendations for remediation
```

### Continuous Discovery for Security Monitoring

```cmd
# Set up regular discovery scans
# Weekly: Full discovery
nmap --script discovery -oA weekly-discovery-$(date +%Y%m%d) network-range

# Daily: Quick check for changes
nmap --script "broadcast-* and targets-sniffer" -oA daily-check-$(date +%Y%m%d)

# Compare results over time:
ndiff weekly-discovery-20240101.xml weekly-discovery-20240108.xml

# Alert on:
- New hosts appearing
- Services changing
- Configuration modifications
- Unauthorized changes
```

## Next Steps

Tomorrow (Day 13) will cover:
- Using NSE Vulnerability Scripts
- Automated vulnerability detection
- Common vulnerability checks
- Interpreting vulnerability scan results

## Reflection Questions

1. Which discovery script revealed the most surprising information?
2. How do discovery scripts change your understanding of a network?
3. What ethical considerations are most important when using discovery scripts?
4. How could you use discovery scripts to improve your organization's security?

## Need Help?

- Explore all discovery scripts: `ls /usr/share/nmap/scripts/ | grep -i discover`
- Check script documentation: `nmap --script-help script-name`
- Use verbose mode for debugging: `nmap -v --script script-name`
- Practice in isolated lab environments first
- Join Nmap community for script recommendations

**🎉 Congratulations on completing Day 12!** You've now mastered the art of automated network discovery with NSE. You can find what others miss and build comprehensive network maps automatically.

*"Discovery scripts are the eyes that see through walls, the ears that hear whispers in the network. Today you learned to use these senses to understand your digital environment completely."*