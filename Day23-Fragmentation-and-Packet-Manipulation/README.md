# Day 23 - Fragmentation and Packet Manipulation

## Day Overview
Welcome to Day 23! Today you will learn how packet fragmentation and packet manipulation work in Nmap, and how these techniques interact with real network defenses.
This is advanced material, but we will keep it beginner-friendly, ethical, and focused on safe, authorized testing environments.

This lesson builds on the previous three days:
- Day 20: Stealth and evasion techniques
- Day 21: Firewall and IDS evasion
- Day 22: Spoofing and decoy scanning

## Learning Objectives
By completing Day 23, you will be able to:
- Understand what fragmentation is and why it matters
- Use Nmap fragmentation options safely in a lab
- Identify when fragmentation can break scans or reduce accuracy
- Combine packet manipulation with conservative timing options
- Document and report your scan settings clearly

## Important Ethics Note
Fragmentation and packet manipulation must only be used with explicit permission.
Do not attempt to bypass defenses on networks you do not own or control.

## What Is Packet Fragmentation?
Packet fragmentation splits a larger packet into multiple smaller fragments.
Fragmentation can sometimes evade simple filters that only inspect complete packets.
Modern firewalls often reassemble fragments before inspection.

## Why Fragmentation Is Risky
- Many devices drop fragmented packets
- Reassembly can fail or be slow
- Results may be incomplete or misleading
- IDS/IPS may still detect fragmented scans

## What Is Packet Manipulation?
Packet manipulation means changing how Nmap sends packets to alter detection patterns.
Examples include changing packet sizes, scan delays, or TCP flags.

## Day 20 Recap: Stealth Basics
Stealth reduces scan noise but does not make scans invisible.

## Day 21 Recap: IDS/IPS Limits
Modern IDS/IPS can still detect stealth and fragmentation attempts.

## Day 22 Recap: Decoys and Spoofing
Decoys hide origin among fake IPs, but do not hide the scan itself.

## Core Nmap Fragmentation Options

### 1. Basic Fragmentation
```bash
sudo nmap -sS -f target
```
`-f` splits packets into small fragments (8 bytes of data).

### 2. Custom Fragment Size
```bash
sudo nmap -sS --mtu 16 target
```
`--mtu` sets the fragment size (must be a multiple of 8).

### 3. Combine Fragmentation with Conservative Timing
```bash
sudo nmap -sS -T2 --max-rate 50 -f target
```

## Packet Manipulation Options (Beginner-Friendly)

### 1. Scan Delay
```bash
sudo nmap -sS --scan-delay 200ms target
```
Adds a delay between probes to reduce bursts.

### 2. Host Timeout
```bash
nmap -sS --host-timeout 5m target
```
Stops scanning a host after a set time.

### 3. Min/Max Rate
```bash
nmap -sS --min-rate 10 --max-rate 100 target
```
Controls how fast packets are sent.

### 4. Data Length
```bash
sudo nmap -sS --data-length 50 target
```
Adds random data to packet payloads to change signature.

## How Fragmentation Interacts with Firewalls
- Some firewalls reassemble fragments before inspection
- Others drop fragments entirely
- Some legacy systems might allow fragments through

## When Not to Use Fragmentation
- High-latency or unstable networks
- When you need precise results quickly
- On networks that explicitly block fragments

## Practical Examples

### Example 1: Conservative Fragmented Scan
```bash
sudo nmap -sS -T2 --max-rate 30 -f -p 22,80,443 target -oA outputs/frag-basic
```

### Example 2: Custom MTU Fragmentation
```bash
sudo nmap -sS --mtu 16 -T2 target -oA outputs/frag-mtu16
```

### Example 3: Fragmentation with Random Data
```bash
sudo nmap -sS -f --data-length 50 target -oA outputs/frag-data
```

### Example 4: Fragmentation on a Small Range
```bash
sudo nmap -sS -T2 -f --randomize-hosts 192.168.1.0/29 -oA outputs/frag-range
```

### Example 5: Discovery First, Then Fragmented Scan
```bash
nmap -sn 192.168.1.0/29 -oA outputs/frag-discovery
awk '/Up$/{print $2}' outputs/frag-discovery.gnmap > outputs/frag-live.txt
sudo nmap -sS -T2 -f -iL outputs/frag-live.txt -oA outputs/frag-live-scan
```

## Combining with Automation (Day 18)
```bash
#!/usr/bin/env bash
set -euo pipefail
TARGET_FILE="${1:-targets.txt}"
RUN_DIR="outputs/$(date +%Y%m%d-%H%M%S)"
mkdir -p "$RUN_DIR"
sudo nmap -sS -T2 --max-rate 50 -f -iL "$TARGET_FILE" -oA "$RUN_DIR/fragmented"
```

## Combining with Multi-Target Scanning (Day 19)
```bash
sudo nmap -sS -T2 -f --randomize-hosts 10.0.0.0/24 -oA outputs/frag-multi
```

## Logging and Reporting
Always record:
- Time and date
- Target scope
- Fragmentation and manipulation options used
- Output file paths

## Troubleshooting

### No Responses From Hosts
```text
Fix:
- Fragmentation may be blocked
- Remove `-f` and test baseline connectivity
- Try a smaller MTU or normal scan first
```

### Scan Results Are Incomplete
```text
Fix:
- Increase timeouts
- Reduce scan delays
- Compare results with a non-fragmented scan
```

### Scan Is Too Slow
```text
Fix:
- Reduce fragmentation
- Increase timing from -T1 to -T2 or -T3
- Limit the port list
```

## Mini Glossary
- Fragmentation: Splitting packets into smaller pieces
- MTU: Maximum Transmission Unit, controls fragment size
- Data length: Adds payload data to packets

## Step-by-Step Lab: Safe Fragmentation
### Step 1: List Targets
```bash
nmap -sL 192.168.1.0/29
```
### Step 2: Host Discovery
```bash
nmap -sn 192.168.1.0/29 -oA outputs/lab-discovery
```
### Step 3: Fragmented Scan Live Hosts
```bash
awk '/Up$/{print $2}' outputs/lab-discovery.gnmap > outputs/lab-live.txt
sudo nmap -sS -T2 -f -iL outputs/lab-live.txt -oA outputs/lab-fragmented
```
### Step 4: Review Results
```bash
grep "/open/" outputs/lab-fragmented.gnmap
```

## Advanced Concepts (Theory)
### Fragment Reassembly and Evasion
Some IDS/IPS systems reassemble fragments before inspection.
If reassembly is strong, fragmentation provides little benefit.

### Evasion vs Accuracy
The more manipulation you add, the more likely accuracy drops.
Always compare results against a clean baseline scan.

### Network Path Effects
Fragmented packets can be dropped or reordered by middleboxes.
This can cause missing or misleading results.

### Legal and Operational Risk
Fragmentation can look suspicious and may violate policies.
Always get explicit approval before testing.

## Real-World Use Cases (Defensive)
- Verify firewall behavior against fragments
- Test IDS/IPS reassembly capabilities
- Validate whether security controls handle non-standard traffic

## Best Practices
- Start with a normal scan baseline
- Use fragmentation only in a controlled lab
- Keep timing conservative
- Document everything
- Compare results to verify accuracy

## Reflection Questions
1. Why can fragmentation reduce accuracy?
2. When would you avoid fragmentation completely?
3. How do Days 20-22 influence your approach to Day 23?
4. What should you document after a fragmented scan?

## Next Steps
Tomorrow (Day 24) you will learn about scanning web servers with Nmap.

**Congratulations on completing Day 23!** You now understand fragmentation and packet manipulation responsibly.
