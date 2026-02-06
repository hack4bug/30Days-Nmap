# Day 10 - Timing and Performance Options

## Day Overview
Welcome to Day 10! Today, we dive into one of Nmap's most powerful but often misunderstood features: **Timing and Performance Options**. This is where you learn to control the "pace" of your scans - from lightning-fast sweeps to ultra-stealthy reconnaissance. Mastering timing transforms you from a basic scanner to a strategic operator.

## Learning Objectives
By completing Day 10, you will be able to:
- Understand Nmap's timing templates and their purposes
- Customize timing parameters for different scenarios
- Balance speed, stealth, and reliability in scans
- Optimize scans for network conditions and target types
- Troubleshoot performance issues and timing problems

## Why Timing Matters

### The Scanning Balance Triad

```cmd
The Three Factors You Must Balance:

1. SPEED ← → STEALTH
   Fast scans get done quickly BUT are easily detected
   Slow scans are stealthier BUT take forever

2. RELIABILITY ← → SPEED
   Reliable scans are accurate BUT slower
   Fast scans may miss information BUT finish quickly

3. NETWORK IMPACT ← → COMPLETENESS
   Low-impact scans are polite BUT may skip data
   Comprehensive scans get everything BUT create traffic
```

### Real-World Timing Example

```cmd
Scanning a /24 network (254 hosts):

Paranoid (T0): ~21 HOURS
   - 5+ minutes between packets
   - Extremely stealthy
   - Practical for high-security targets only

Insane (T5): ~5 MINUTES
   - Maximum speed, no delays
   - Will trigger every alarm
   - Use on your own isolated lab only

Normal (T3): ~30 MINUTES
   - Balanced default
   - Moderate detection risk
   - Good for general use
```

## Nmap Timing Templates (-T0 to -T5)

### Overview of Timing Templates

```cmd
Six Timing Templates:

T0: Paranoid       - Extremely slow, maximum stealth
T1: Sneaky         - Very slow, stealthy
T2: Polite         - Slower, less intrusive
T3: Normal         - Default balanced setting
T4: Aggressive     - Faster, more aggressive
T5: Insane         - Maximum speed, minimum stealth
```

### Detailed Template Analysis

#### **T0: Paranoid**
```cmd
Characteristics:
- 5+ minutes between packets
- Serialized scanning (no parallelism)
- IDS evasion: Very difficult to detect
- Use case: Extreme stealth required

Command: nmap -T0 target
Time per port: ~300+ seconds
IDS Evasion: Excellent
Network Impact: Minimal
Reliability: High (due to retries)
```

#### **T1: Sneaky**
```cmd
Characteristics:
- 15 seconds between packets
- Limited parallelism
- IDS evasion: Hard to detect
- Use case: Stealthy reconnaissance

Command: nmap -T1 target
Time per port: ~15 seconds
IDS Evasion: Very Good
Network Impact: Very Low
Reliability: High
```

#### **T2: Polite**
```cmd
Characteristics:
- 0.4 seconds between packets
- Moderate parallelism
- IDS evasion: May avoid basic IDS
- Use case: Being "polite" to networks

Command: nmap -T2 target
Time per port: ~0.4 seconds
IDS Evasion: Good
Network Impact: Low
Reliability: High
```

#### **T3: Normal**
```cmd
Characteristics:
- Default timing (no artificial delays)
- Balanced parallelism
- IDS evasion: Basic IDS may detect
- Use case: General scanning, default

Command: nmap -T3 target  # Same as just nmap target
Time per port: No added delay
IDS Evasion: Moderate
Network Impact: Moderate
Reliability: Good
```

#### **T4: Aggressive**
```cmd
Characteristics:
- Faster timeouts
- Higher parallelism
- IDS evasion: Likely detected
- Use case: Internal networks, speed priority

Command: nmap -T4 target
Time per port: Faster timeouts
IDS Evasion: Poor
Network Impact: Moderate-High
Reliability: Good (with retries)
```

#### **T5: Insane**
```cmd
Characteristics:
- Maximum speed
- Maximum parallelism
- IDS evasion: Definitely detected
- Use case: Your own lab networks only

Command: nmap -T5 target
Time per port: Minimum possible
IDS Evasion: None
Network Impact: High
Reliability: May miss responses (too fast)
```

## Custom Timing Parameters

### When Templates Aren't Enough
Sometimes you need finer control than the six templates provide. Nmap offers granular timing parameters for these situations.

### Key Timing Parameters

#### **Host Timeout (--host-timeout)**
```cmd
# Give up on slow hosts
nmap --host-timeout 30m target

# Examples:
--host-timeout 30m    # 30 minutes per host
--host-timeout 2h     # 2 hours per host
--host-timeout 300s   # 300 seconds per host

# Use when:
- Some hosts are extremely slow
- You want to scan many hosts without getting stuck
- Network conditions vary widely
```

#### **Max Retries (--max-retries)**
```cmd
# Control retransmission attempts
nmap --max-retries 0 target    # No retries (fast)
nmap --max-retries 10 target   # Many retries (reliable)

# Default: 10 retries for most scans
# UDP scans: 3 retries (slower protocol)

# Balance:
0 retries: Fast, may miss packets
10 retries: Slow, very reliable
```

#### **Initial RTT Timeout (--initial-rtt-timeout)**
```cmd
# Set initial round-trip time estimate
nmap --initial-rtt-timeout 100ms target

# What it does:
- Sets how long Nmap waits for first response
- Affects all subsequent time calculations
- Default: 1 second (1000ms)

# Use when:
- You know network latency characteristics
- Scanning local vs remote networks
- Want to optimize initial probe timing
```

#### **Max RTT Timeout (--max-rtt-timeout)**
```cmd
# Maximum time to wait for any packet
nmap --max-rtt-timeout 500ms target

# What it does:
- Caps maximum wait time for responses
- Prevents waiting forever for lost packets
- Default: 1 second (1000ms)

# Use when:
- Network is reliable and fast
- You want to speed up scans
- Dealing with congested networks
```

#### **Min RTT Timeout (--min-rtt-timeout)**
```cmd
# Minimum time to wait for responses
nmap --min-rtt-timeout 10ms target

# What it does:
- Sets floor for response waiting
- Prevents excessive speed on local networks
- Default: 100ms

# Use when:
- Scanning very fast local networks
- Need to avoid overwhelming targets
```

### Parallelism Controls

#### **Min Parallelism (--min-parallelism)**
```cmd
# Minimum number of parallel probes
nmap --min-parallelism 10 target

# What it does:
- Ensures at least N probes in parallel
- Keeps scan moving even with timeouts
- Default: Based on timing template

# Use when:
- Want to maintain minimum scan speed
- Network can handle concurrent probes
```

#### **Max Parallelism (--max-parallelism)**
```cmd
# Maximum number of parallel probes
nmap --max-parallelism 1 target    # Serial scan
nmap --max-parallelism 100 target  # Very parallel

# What it does:
- Caps maximum concurrent probes
- Controls network load
- Default: Based on timing template

# Use when:
- Need to limit network impact
- Scanning fragile devices
- Bandwidth-constrained networks
```

#### **Scan Delay (--scan-delay)**
```cmd
# Minimum time between probes
nmap --scan-delay 1s target    # 1 second between packets
nmap --scan-delay 100ms target # 100 milliseconds

# What it does:
- Adds artificial delay between packets
- Increases stealth, reduces detection
- Default: None (except in T0-T2)

# Use when:
- Need to be extra stealthy
- Avoiding rate-based detection
- Scanning monitored networks
```

#### **Max Scan Delay (--max-scan-delay)**
```cmd
# Maximum delay between probes
nmap --max-scan-delay 10s target

# What it does:
- Caps maximum delay (even if template wants more)
- Ensures scan doesn't become too slow
- Default: Based on timing template

# Use when:
- Using slow template but want time limit
- Need predictable maximum scan time
```

## Practical Timing Scenarios

### Scenario 1: Internal Network Assessment
```cmd
# Fast, comprehensive internal scan
nmap -T4 -A 192.168.1.0/24

# Rationale:
- Internal network: Less need for stealth
- Speed important for 254 hosts
- Reliable results needed
- Network impact acceptable on internal network

# Expected time: 15-30 minutes for full /24 scan
```

### Scenario 2: External Penetration Test
```cmd
# Stealthy external reconnaissance
nmap -T2 -sS -D RND:10 target.com

# Rationale:
- External target: Stealth is critical
- Decoy scanning adds to stealth
- Polite timing avoids detection
- SYN scan (stealthier than connect)

# Expected time: Hours for comprehensive scan
```

### Scenario 3: Quick Service Check
```cmd
# Fast check of specific services
nmap -T5 -F --max-retries 0 --max-rtt-timeout 100ms target

# Rationale:
- Speed is everything
- Only checking common ports (top 100)
- Accepting some packet loss
- Very short timeouts

# Expected time: Seconds to minutes
```

### Scenario 4: Fragile Device Scanning
```cmd
# Gentle scan of network devices
nmap -T1 --max-parallelism 1 --scan-delay 2s router-ip

# Rationale:
- Network devices can be fragile
- Avoid overwhelming with traffic
- Serial scanning (one probe at a time)
- Significant delays between packets

# Expected time: Very slow, but safe
```

## Performance Optimization Techniques

### Optimizing for Network Type

#### **Local Network (LAN) Optimization**
```cmd
# Local network characteristics:
- Low latency (1-10ms)
- High bandwidth
- Minimal packet loss
- Less monitoring

# Optimal settings:
nmap -T4 --max-rtt-timeout 100ms --initial-rtt-timeout 50ms target
# Fast timing, short timeouts, high parallelism
```

#### **Internet/WAN Optimization**
```cmd
# Internet characteristics:
- Higher latency (50-500ms)
- Variable bandwidth
- Packet loss possible
- More monitoring/firewalls

# Optimal settings:
nmap -T3 --max-rtt-timeout 2000ms --max-retries 2 target
# Moderate timing, longer timeouts, some retries
```

#### **High-Latency Network Optimization**
```cmd
# Satellite/VPN/International networks:
- Very high latency (500ms+)
- Bandwidth constraints
- High packet loss

# Optimal settings:
nmap -T2 --max-rtt-timeout 5000ms --max-retries 5 --min-parallelism 1 target
# Slow timing, very long timeouts, more retries, limited parallelism
```

### Scan Speed Calculation and Estimation

```cmd
Estimating Scan Time:

Factors affecting speed:
1. Number of hosts
2. Number of ports per host
3. Timing template (-T0 to -T5)
4. Network latency
5. Target responsiveness
6. Parallelism settings

Rough Estimation Formula:
Time ≈ (Hosts × Ports × Per-port-time) / Parallelism

Example: 10 hosts × 1000 ports × 0.1s ÷ 10 parallelism = 100 seconds
```

## Advanced Timing Strategies

### Adaptive Timing
Nmap automatically adjusts timing based on:
1. **Response times**: Faster responses → faster scanning
2. **Packet loss**: More loss → slower, more retries
3. **Network conditions**: Dynamically adjusts timeouts

### Custom Timing Profiles
Create reusable timing configurations:

```cmd
# Create custom timing profile in .nmaprc
echo "--max-rtt-timeout 200ms" >> ~/.nmaprc
echo "--max-retries 2" >> ~/.nmaprc
echo "--max-parallelism 50" >> ~/.nmaprc

# Now these apply to all scans
# Override with command line if needed
```

### Timing for Specific Scan Types

#### **UDP Scan Timing**
```cmd
# UDP scans are inherently slower
sudo nmap -sU -T3 --max-retries 1 target

# Why different timing?
- UDP responses are unpredictable
- ICMP rate limiting may occur
- Need to wait for possible responses
- Default UDP timeout: 5 seconds per port!
```

#### **Version Detection Timing**
```cmd
# Version detection adds significant time
nmap -sV -T4 --version-intensity 3 target

# Timing considerations:
- Service probes take time
- Some services are slow to respond
- SSL/TLS negotiation adds overhead
- Balance between completeness and speed
```

#### **OS Detection Timing**
```cmd
# OS detection requires multiple probes
sudo nmap -O -T3 --max-os-tries 2 target

# Timing considerations:
- 15+ different probe types
- Need multiple responses for accuracy
- Some probes may be blocked/filtered
- Balance accuracy vs. time
```

## Monitoring and Controlling Network Impact

### Bandwidth Control
```cmd
# Limit packet transmission rate
nmap --min-rate 100 target    # Minimum 100 packets/second
nmap --max-rate 10 target     # Maximum 10 packets/second

# Use cases:
--min-rate: Ensure scan progresses
--max-rate: Prevent bandwidth saturation
```

### Packet Rate Calculation
```cmd
Estimating packet rates:

Typical packet sizes:
- TCP SYN: 54 bytes (without options)
- TCP SYN-ACK: 60 bytes
- ICMP Echo: 74 bytes

Bandwidth calculation:
100 packets/sec × 100 bytes × 8 bits = 80,000 bps
+ Add overhead for headers, retransmissions
```

### Impact Monitoring During Scans
```cmd
# Monitor network during scan
# Terminal 1: Start scan with verbose output
sudo nmap -T4 -v target

# Terminal 2: Monitor network traffic
sudo tcpdump -i eth0 host target-ip

# Look for:
- Target response patterns
- Network congestion signs
- Unusual error messages
- Performance degradation
```

## Timing Troubleshooting

### Common Issues and Solutions

#### **Issue: Scan Too Slow**
```cmd
Symptoms:
- Scan taking hours for few hosts
- Progress bar barely moving
- High latency reported

Solutions:
# Increase timing template
nmap -T4 target

# Reduce retries
nmap --max-retries 1 target

# Shorten timeouts
nmap --max-rtt-timeout 500ms target

# Increase parallelism
nmap --min-parallelism 20 target

# Scan fewer ports
nmap -F target  # Top 100 ports only
```

#### **Issue: Scan Too Fast (Missing Results)**
```cmd
Symptoms:
- Scan completes suspiciously fast
- Missing expected open ports
- High packet loss reported

Solutions:
# Decrease timing template
nmap -T2 target

# Increase retries
nmap --max-retries 5 target

# Lengthen timeouts
nmap --max-rtt-timeout 2000ms target

# Decrease parallelism
nmap --max-parallelism 5 target
```

#### **Issue: Unstable/Inconsistent Results**
```cmd
Symptoms:
- Different scans show different results
- Intermittent port detection
- Timeout errors

Solutions:
# Use more conservative timing
nmap -T3 target

# Ensure stable network connection
# Check for network issues
# Verify target is not overloaded

# Add scan delays
nmap --scan-delay 100ms target
```

## Performance Testing and Benchmarking

### Creating Timing Baselines
```cmd
# Test different timing settings on known target
# Record results for comparison

Test 1: nmap -T0 target --stats-every 10s
Test 2: nmap -T1 target --stats-every 10s
Test 3: nmap -T2 target --stats-every 10s
Test 4: nmap -T3 target --stats-every 10s
Test 5: nmap -T4 target --stats-every 10s
Test 6: nmap -T5 target --stats-every 10s

# Compare:
- Total scan time
- Packets sent/received
- Open ports detected
- Any missed information
```

### Using --stats-every for Monitoring
```cmd
# Get periodic progress reports
nmap --stats-every 10s target

# Output shows:
Time elapsed
Percentage complete
Hosts scanned
Open ports found
Packets sent/received

# Helps identify:
- Scan progression rate
- Performance bottlenecks
- When to adjust timing
```

### Network Impact Testing
```cmd
# Test scan impact before full deployment
# Small test first:
nmap -T4 --max-rate 100 target-test

# Monitor during test:
- Target system performance
- Network device load
- Bandwidth utilization
- Log/alert generation

# Adjust based on results before full scan
```

## Best Practices by Use Case

### Security Audits
```cmd
# Comprehensive but reasonable
nmap -T3 -sS -sV -O --script=default target

# Why T3?
- Balanced speed vs. stealth
- Reliable results
- Moderate network impact
- Good for scheduled audits
```

### Incident Response
```cmd
# Fast but thorough
nmap -T4 -A --max-retries 2 target

# Why T4?
- Speed is critical
- Need comprehensive information
- Internal network usually
- Accept some detection risk
```

### Continuous Monitoring
```cmd
# Lightweight and fast
nmap -T5 -F --max-retries 0 local-subnet

# Why T5?
- Scanning own network frequently
- Speed enables frequent scans
- Minimal detail needed for change detection
- Accept packet loss for speed
```

### External Reconnaissance
```cmd
# Stealthy and thorough
nmap -T2 -sS -sV --scan-delay 500ms external-target

# Why T2 with delay?
- Maximum stealth needed
- Willing to trade time for stealth
- Avoid triggering alarms
- Still get version information
```

## Timing Decision Framework

### Questions to Ask Before Setting Timing

1. **What is my objective?**
   - Discovery? Audit? Monitoring? Response?

2. **What is the network environment?**
   - Internal? External? Monitored? Sensitive?

3. **What are the constraints?**
   - Time available? Bandwidth limits? Stealth required?

4. **What are the consequences of detection?**
   - None? Minor? Serious? Critical?

5. **What level of reliability is needed?**
   - Can I miss some information?
   - Do I need 100% accuracy?

### Decision Tree Example
```cmd
Start → What's the environment?
├─ Internal Lab → Use T4 or T5
├─ Internal Corporate → Use T3
├─ External Monitored → Use T2
└─ External High-Security → Use T0 or T1

Then → Any special requirements?
├─ Fragile devices → Add --max-parallelism 1
├─ Bandwidth limits → Add --max-rate
├─ Time constraints → Adjust template up
└─ Stealth critical → Add --scan-delay
```

## Next Steps

Tomorrow (Day 11) will cover:
- Nmap Scripting Engine (NSE) Basics
- Understanding NSE architecture
- Running basic scripts
- Script categories and purposes

## Reflection Questions

1. Which timing template worked best for your network environment?
2. What trade-offs did you notice between speed and reliability?
3. How would you adjust timing for scanning a production web server vs a test server?
4. What timing considerations are most important for ethical scanning?

## Need Help?

- Use `nmap -h` to see all timing options
- Experiment in lab environments first
- Start conservative (slower) and speed up as you learn
- Monitor scan progress with `-v` or `--stats-every`
- Document what timing works for different scenarios

**🎉 Congratulations on completing Day 10!** You've now mastered the art and science of scan timing - knowing not just what to scan, but how to scan it effectively. This knowledge separates beginners from experts in network reconnaissance.

*"The master fisherman knows not just where to cast the net, but how fast to pull it in. Timing is everything in the catch."*