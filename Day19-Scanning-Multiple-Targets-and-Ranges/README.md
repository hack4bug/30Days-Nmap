# Day 19 - Scanning Multiple Targets and Ranges

## Day Overview
Welcome to Day 19! Today you will learn how to scan multiple targets and ranges in a safe, beginner-friendly, and repeatable way.
This lesson builds on the previous two days:
- Day 17: Output formats and parsing (grepable and XML)
- Day 18: Automation with shell scripts
- Day 19: Multi-target and range scanning for real networks

## Learning Objectives
By completing Day 19, you will be able to:
- Scan multiple hosts in one command
- Use CIDR ranges, hyphen ranges, and target lists
- Exclude sensitive hosts and reduce scope errors
- Control scan speed and order for large ranges
- Combine multi-target scans with output parsing and automation

## Before You Start
### Prerequisites
- Nmap installed and working
- You have permission to scan the target network
- You understand basic scanning options from Days 1-18

### Quick Safety Reminder
Only scan systems you own or have explicit authorization to test.
Large scans can affect performance, so start small and go slow.

## How Multi-Target Scanning Works
Nmap accepts many target formats in a single command:
- Single IP: `192.168.1.10`
- Multiple IPs: `192.168.1.10 192.168.1.11`
- CIDR ranges: `192.168.1.0/24`
- Hyphen ranges: `192.168.1.10-50`
- DNS names: `server1.example.com`
- Target lists: `-iL targets.txt`

## Day 17 Recap: Why Output Formats Matter
When scanning many targets, you must save structured output for review and automation.
The most useful formats:
- Normal: human readable
- Grepable (`-oG`): quick parsing for scripts
- XML (`-oX`): structured data for tooling
- All formats (`-oA`): best practice for automation

Example:
```bash
nmap -sV -oA outputs/multi-scan 192.168.1.0/24
```

## Day 18 Recap: Why Automation Helps
Multi-target scans are repetitive. Automating them reduces mistakes.
A simple script can:
- Read targets from a file
- Run consistent scan options
- Store outputs in dated folders
- Parse results automatically

## Day 19 Core Skill: Multi-Target Scanning
Below are the most common ways to scan multiple targets.

### 1. Scan Multiple IPs in One Command
```bash
nmap -sV 192.168.1.10 192.168.1.11 192.168.1.12
```
Use this for a small number of known hosts.

### 2. Scan a Hyphen Range
```bash
nmap -sV 192.168.1.10-50
```
This scans 41 hosts from 10 to 50.

### 3. Scan a CIDR Range
```bash
nmap -sV 192.168.1.0/24
```
A /24 includes 256 IPs (0-255).

### 4. Scan Multiple Ranges Together
```bash
nmap -sV 192.168.1.0/24 192.168.2.0/24
```
This is useful for separate subnets.

### 5. Scan Using a Target List
Create `targets.txt` with one host per line:
```text
192.168.1.10
192.168.1.20
server1.example.com
```
Run:
```bash
nmap -sV -iL targets.txt
```

## Excluding Targets Safely
Exclude sensitive or out-of-scope systems:
```bash
nmap -sV 192.168.1.0/24 --exclude 192.168.1.5,192.168.1.10
```
Use an exclude file for larger lists:
```bash
nmap -sV 192.168.1.0/24 --excludefile exclude.txt
```

## Listing Targets Without Scanning
To verify scope before scanning:
```bash
nmap -sL 192.168.1.0/29
```
This prints the target list but does not send packets.

## Discover Live Hosts First
If you have a large range, find live hosts first:
```bash
nmap -sn 192.168.1.0/24 -oA outputs/ping-sweep
```
Then scan only live hosts:
```bash
awk '/Up$/{print $2}' outputs/ping-sweep.gnmap > outputs/alive.txt
nmap -sV -iL outputs/alive.txt -oA outputs/service-scan
```

## Ordering and Randomization
Large scans can trigger alerts if done in order.
Use randomization to reduce patterns:
```bash
nmap -sV 192.168.1.0/24 --randomize-hosts
```

## Controlling Scan Speed
Large ranges need careful speed control.
```bash
nmap -sV 192.168.1.0/24 -T3 --max-rate 100
```
Slow down if the network is sensitive.

## Combining Targets and Ports
Scan common ports across many hosts:
```bash
nmap -p 22,80,443 192.168.1.0/24
```
Fast scan for a quick check:
```bash
nmap -F 192.168.1.0/24
```

## Output Management for Multi-Target Scans
Always use `-oA` to keep results organized:
```bash
nmap -sV -oA outputs/multi-20260206 192.168.1.0/24
```
Create date-based folders for weekly scans:
```bash
RUN_DIR="outputs/$(date +%Y%m%d)"
mkdir -p "$RUN_DIR"
nmap -sV -oA "$RUN_DIR/weekly" 192.168.1.0/24
```

## Parsing Results (Day 17 Skills Applied)
Extract hosts with open ports:
```bash
grep "/open/" outputs/multi-20260206.gnmap
```
Count open ports per host:
```bash
awk '/open/{print $2}' outputs/multi-20260206.gnmap | sort | uniq -c
```
Generate a diff between two scans:
```bash
ndiff outputs/scan-old.xml outputs/scan-new.xml > outputs/diff.txt
```

## Automation Example (Day 18 Skills Applied)
A simple script that reads targets and saves output:
```bash
#!/usr/bin/env bash
set -euo pipefail
TARGET_FILE="${1:-targets.txt}"
RUN_DIR="outputs/$(date +%Y%m%d-%H%M%S)"
mkdir -p "$RUN_DIR"
nmap -sV -iL "$TARGET_FILE" -oA "$RUN_DIR/multi"
```

## Common Mistakes and How to Avoid Them
- Forgetting `-iL` when using a target file
- Scanning the wrong subnet due to a typo
- Not excluding out-of-scope IPs
- Running a full port scan on a very large range too quickly

## Mini Glossary
- CIDR: A way to define IP ranges (example: /24)
- Target list: A file with one host per line
- Exclude list: Hosts you do not want to scan
- Ping sweep: A discovery scan for live hosts

## Step-by-Step Lab: Small Office Network
This lab shows how to scan a small /29 range (8 IPs).

### Step 1: List Targets
```bash
nmap -sL 192.168.1.0/29
```

### Step 2: Find Live Hosts
```bash
nmap -sn 192.168.1.0/29 -oA outputs/lab-ping
```

### Step 3: Scan Live Hosts
```bash
awk '/Up$/{print $2}' outputs/lab-ping.gnmap > outputs/lab-alive.txt
nmap -sV -iL outputs/lab-alive.txt -oA outputs/lab-services
```

### Step 4: Review Results
```bash
grep "/open/" outputs/lab-services.gnmap
```

## Large Range Strategy
When scanning a /16 or bigger, reduce scope risk:
- Confirm authorization for each subnet
- Run discovery first
- Scan only live hosts
- Use safe timing options

## Targeting with DNS Names
You can mix IPs and DNS names:
```bash
nmap -sV server1.example.com 10.0.0.5 10.0.0.10
```
Be aware of DNS resolution delays for large lists.

## Controlling DNS Resolution
Disable DNS to speed up scanning:
```bash
nmap -n 192.168.1.0/24
```
Enable reverse DNS if needed:
```bash
nmap -R 192.168.1.0/24
```

## Selecting Port Ranges
Common options:
- `-F` for top 100 ports
- `--top-ports 1000` for top 1000
- `-p 1-1000` for a defined range
- `-p-` for all 65535 ports (slow)

## Example: Balanced Multi-Target Scan
```bash
nmap -sV -T3 --max-rate 200 --top-ports 100 192.168.1.0/24 -oA outputs/balanced
```

## Example: Quick Audit for Web Servers
```bash
nmap -p 80,443,8080,8443 -sV --script http-title 192.168.1.0/24 -oA outputs/web-audit
```

## Example: Inventory Scan with OS Detection
```bash
sudo nmap -O -sV 192.168.1.0/24 -oA outputs/inventory
```
Use only when you have permission and know it is safe.

## Handling Mixed Target Types
You can mix ranges and files:
```bash
nmap -sV 192.168.1.0/24 -iL targets.txt -oA outputs/mixed
```
Nmap will scan the union of all targets.

## Using `--exclude` with Mixed Targets
```bash
nmap -sV 192.168.1.0/24 -iL targets.txt --exclude 192.168.1.1,192.168.1.2
```

## Using `--excludefile` with Mixed Targets
```bash
nmap -sV 192.168.1.0/24 -iL targets.txt --excludefile exclude.txt
```

## Performance Tips for Beginners
- Start with `-sn` for discovery
- Use `-T2` or `-T3` to avoid network stress
- Avoid `-p-` on large ranges at first
- Use `-n` to avoid DNS delays

## Ethics and Compliance
- Always confirm authorization
- Stay inside defined scope
- Document what you scanned and when
- Share results responsibly

## Beginner-Friendly Checklists
### Pre-Scan Checklist
- I have written permission to scan
- I confirmed the target range
- I know which ports I will scan
- I created an output folder
- I will start with a discovery scan

### Post-Scan Checklist
- I saved output files
- I reviewed open ports
- I documented unusual services
- I stored results in the correct folder
- I cleaned up temporary lists

## Extended Practice Section
Below are many small drills. They repeat the same patterns so you can build muscle memory.
Use your lab environment and adjust IPs as needed.

### Drill 1: Multi-Target Discovery
Goal: Find live hosts, then scan only those hosts.
1. Run a ping sweep on your lab range.
2. Save output with `-oA` for parsing.
3. Extract live hosts from the `.gnmap` file.
4. Run a service scan on the live hosts list.
5. Save the service scan output in a dated folder.
6. Review the results for open ports.
7. Write down any unexpected services.

### Drill 2: Multi-Target Discovery
Goal: Find live hosts, then scan only those hosts.
1. Run a ping sweep on your lab range.
2. Save output with `-oA` for parsing.
3. Extract live hosts from the `.gnmap` file.
4. Run a service scan on the live hosts list.
5. Save the service scan output in a dated folder.
6. Review the results for open ports.
7. Write down any unexpected services.

### Drill 3: Multi-Target Discovery
Goal: Find live hosts, then scan only those hosts.
1. Run a ping sweep on your lab range.
2. Save output with `-oA` for parsing.
3. Extract live hosts from the `.gnmap` file.
4. Run a service scan on the live hosts list.
5. Save the service scan output in a dated folder.
6. Review the results for open ports.
7. Write down any unexpected services.

### Drill 4: Multi-Target Discovery
Goal: Find live hosts, then scan only those hosts.
1. Run a ping sweep on your lab range.
2. Save output with `-oA` for parsing.
3. Extract live hosts from the `.gnmap` file.
4. Run a service scan on the live hosts list.
5. Save the service scan output in a dated folder.
6. Review the results for open ports.
7. Write down any unexpected services.

### Drill 5: Multi-Target Discovery
Goal: Find live hosts, then scan only those hosts.
1. Run a ping sweep on your lab range.
2. Save output with `-oA` for parsing.
3. Extract live hosts from the `.gnmap` file.
4. Run a service scan on the live hosts list.
5. Save the service scan output in a dated folder.
6. Review the results for open ports.
7. Write down any unexpected services.

### Drill 6: Multi-Target Discovery
Goal: Find live hosts, then scan only those hosts.
1. Run a ping sweep on your lab range.
2. Save output with `-oA` for parsing.
3. Extract live hosts from the `.gnmap` file.
4. Run a service scan on the live hosts list.
5. Save the service scan output in a dated folder.
6. Review the results for open ports.
7. Write down any unexpected services.

### Drill 7: Multi-Target Discovery
Goal: Find live hosts, then scan only those hosts.
1. Run a ping sweep on your lab range.
2. Save output with `-oA` for parsing.
3. Extract live hosts from the `.gnmap` file.
4. Run a service scan on the live hosts list.
5. Save the service scan output in a dated folder.
6. Review the results for open ports.
7. Write down any unexpected services.

### Drill 8: Multi-Target Discovery
Goal: Find live hosts, then scan only those hosts.
1. Run a ping sweep on your lab range.
2. Save output with `-oA` for parsing.
3. Extract live hosts from the `.gnmap` file.
4. Run a service scan on the live hosts list.
5. Save the service scan output in a dated folder.
6. Review the results for open ports.
7. Write down any unexpected services.

### Drill 9: Multi-Target Discovery
Goal: Find live hosts, then scan only those hosts.
1. Run a ping sweep on your lab range.
2. Save output with `-oA` for parsing.
3. Extract live hosts from the `.gnmap` file.
4. Run a service scan on the live hosts list.
5. Save the service scan output in a dated folder.
6. Review the results for open ports.
7. Write down any unexpected services.

### Drill 10: Multi-Target Discovery
Goal: Find live hosts, then scan only those hosts.
1. Run a ping sweep on your lab range.
2. Save output with `-oA` for parsing.
3. Extract live hosts from the `.gnmap` file.
4. Run a service scan on the live hosts list.
5. Save the service scan output in a dated folder.
6. Review the results for open ports.
7. Write down any unexpected services.

### Drill 11: Multi-Target Discovery
Goal: Find live hosts, then scan only those hosts.
1. Run a ping sweep on your lab range.
2. Save output with `-oA` for parsing.
3. Extract live hosts from the `.gnmap` file.
4. Run a service scan on the live hosts list.
5. Save the service scan output in a dated folder.
6. Review the results for open ports.
7. Write down any unexpected services.

### Drill 12: Multi-Target Discovery
Goal: Find live hosts, then scan only those hosts.
1. Run a ping sweep on your lab range.
2. Save output with `-oA` for parsing.
3. Extract live hosts from the `.gnmap` file.
4. Run a service scan on the live hosts list.
5. Save the service scan output in a dated folder.
6. Review the results for open ports.
7. Write down any unexpected services.

### Drill 13: Multi-Target Discovery
Goal: Find live hosts, then scan only those hosts.
1. Run a ping sweep on your lab range.
2. Save output with `-oA` for parsing.
3. Extract live hosts from the `.gnmap` file.
4. Run a service scan on the live hosts list.
5. Save the service scan output in a dated folder.
6. Review the results for open ports.
7. Write down any unexpected services.

### Drill 14: Multi-Target Discovery
Goal: Find live hosts, then scan only those hosts.
1. Run a ping sweep on your lab range.
2. Save output with `-oA` for parsing.
3. Extract live hosts from the `.gnmap` file.
4. Run a service scan on the live hosts list.
5. Save the service scan output in a dated folder.
6. Review the results for open ports.
7. Write down any unexpected services.

### Drill 15: Multi-Target Discovery
Goal: Find live hosts, then scan only those hosts.
1. Run a ping sweep on your lab range.
2. Save output with `-oA` for parsing.
3. Extract live hosts from the `.gnmap` file.
4. Run a service scan on the live hosts list.
5. Save the service scan output in a dated folder.
6. Review the results for open ports.
7. Write down any unexpected services.

### Drill 16: Multi-Target Discovery
Goal: Find live hosts, then scan only those hosts.
1. Run a ping sweep on your lab range.
2. Save output with `-oA` for parsing.
3. Extract live hosts from the `.gnmap` file.
4. Run a service scan on the live hosts list.
5. Save the service scan output in a dated folder.
6. Review the results for open ports.
7. Write down any unexpected services.

### Drill 17: Multi-Target Discovery
Goal: Find live hosts, then scan only those hosts.
1. Run a ping sweep on your lab range.
2. Save output with `-oA` for parsing.
3. Extract live hosts from the `.gnmap` file.
4. Run a service scan on the live hosts list.
5. Save the service scan output in a dated folder.
6. Review the results for open ports.
7. Write down any unexpected services.

### Drill 18: Multi-Target Discovery
Goal: Find live hosts, then scan only those hosts.
1. Run a ping sweep on your lab range.
2. Save output with `-oA` for parsing.
3. Extract live hosts from the `.gnmap` file.
4. Run a service scan on the live hosts list.
5. Save the service scan output in a dated folder.
6. Review the results for open ports.
7. Write down any unexpected services.

### Drill 19: Multi-Target Discovery
Goal: Find live hosts, then scan only those hosts.
1. Run a ping sweep on your lab range.
2. Save output with `-oA` for parsing.
3. Extract live hosts from the `.gnmap` file.
4. Run a service scan on the live hosts list.
5. Save the service scan output in a dated folder.
6. Review the results for open ports.
7. Write down any unexpected services.

### Drill 20: Multi-Target Discovery
Goal: Find live hosts, then scan only those hosts.
1. Run a ping sweep on your lab range.
2. Save output with `-oA` for parsing.
3. Extract live hosts from the `.gnmap` file.
4. Run a service scan on the live hosts list.
5. Save the service scan output in a dated folder.
6. Review the results for open ports.
7. Write down any unexpected services.

### Drill 21: Multi-Target Discovery
Goal: Find live hosts, then scan only those hosts.
1. Run a ping sweep on your lab range.
2. Save output with `-oA` for parsing.
3. Extract live hosts from the `.gnmap` file.
4. Run a service scan on the live hosts list.
5. Save the service scan output in a dated folder.
6. Review the results for open ports.
7. Write down any unexpected services.

### Drill 22: Multi-Target Discovery
Goal: Find live hosts, then scan only those hosts.
1. Run a ping sweep on your lab range.
2. Save output with `-oA` for parsing.
3. Extract live hosts from the `.gnmap` file.
4. Run a service scan on the live hosts list.
5. Save the service scan output in a dated folder.
6. Review the results for open ports.
7. Write down any unexpected services.

### Drill 23: Multi-Target Discovery
Goal: Find live hosts, then scan only those hosts.
1. Run a ping sweep on your lab range.
2. Save output with `-oA` for parsing.
3. Extract live hosts from the `.gnmap` file.
4. Run a service scan on the live hosts list.
5. Save the service scan output in a dated folder.
6. Review the results for open ports.
7. Write down any unexpected services.

### Drill 24: Multi-Target Discovery
Goal: Find live hosts, then scan only those hosts.
1. Run a ping sweep on your lab range.
2. Save output with `-oA` for parsing.
3. Extract live hosts from the `.gnmap` file.
4. Run a service scan on the live hosts list.
5. Save the service scan output in a dated folder.
6. Review the results for open ports.
7. Write down any unexpected services.

### Drill 25: Multi-Target Discovery
Goal: Find live hosts, then scan only those hosts.
1. Run a ping sweep on your lab range.
2. Save output with `-oA` for parsing.
3. Extract live hosts from the `.gnmap` file.
4. Run a service scan on the live hosts list.
5. Save the service scan output in a dated folder.
6. Review the results for open ports.
7. Write down any unexpected services.

### Drill 26: Multi-Target Discovery
Goal: Find live hosts, then scan only those hosts.
1. Run a ping sweep on your lab range.
2. Save output with `-oA` for parsing.
3. Extract live hosts from the `.gnmap` file.
4. Run a service scan on the live hosts list.
5. Save the service scan output in a dated folder.
6. Review the results for open ports.
7. Write down any unexpected services.

### Drill 27: Multi-Target Discovery
Goal: Find live hosts, then scan only those hosts.
1. Run a ping sweep on your lab range.
2. Save output with `-oA` for parsing.
3. Extract live hosts from the `.gnmap` file.
4. Run a service scan on the live hosts list.
5. Save the service scan output in a dated folder.
6. Review the results for open ports.
7. Write down any unexpected services.

### Drill 28: Multi-Target Discovery
Goal: Find live hosts, then scan only those hosts.
1. Run a ping sweep on your lab range.
2. Save output with `-oA` for parsing.
3. Extract live hosts from the `.gnmap` file.
4. Run a service scan on the live hosts list.
5. Save the service scan output in a dated folder.
6. Review the results for open ports.
7. Write down any unexpected services.

### Drill 29: Multi-Target Discovery
Goal: Find live hosts, then scan only those hosts.
1. Run a ping sweep on your lab range.
2. Save output with `-oA` for parsing.
3. Extract live hosts from the `.gnmap` file.
4. Run a service scan on the live hosts list.
5. Save the service scan output in a dated folder.
6. Review the results for open ports.
7. Write down any unexpected services.

### Drill 30: Multi-Target Discovery
Goal: Find live hosts, then scan only those hosts.
1. Run a ping sweep on your lab range.
2. Save output with `-oA` for parsing.
3. Extract live hosts from the `.gnmap` file.
4. Run a service scan on the live hosts list.
5. Save the service scan output in a dated folder.
6. Review the results for open ports.
7. Write down any unexpected services.

### Drill 31: Multi-Target Discovery
Goal: Find live hosts, then scan only those hosts.
1. Run a ping sweep on your lab range.
2. Save output with `-oA` for parsing.
3. Extract live hosts from the `.gnmap` file.
4. Run a service scan on the live hosts list.
5. Save the service scan output in a dated folder.
6. Review the results for open ports.
7. Write down any unexpected services.

### Drill 32: Multi-Target Discovery
Goal: Find live hosts, then scan only those hosts.
1. Run a ping sweep on your lab range.
2. Save output with `-oA` for parsing.
3. Extract live hosts from the `.gnmap` file.
4. Run a service scan on the live hosts list.
5. Save the service scan output in a dated folder.
6. Review the results for open ports.
7. Write down any unexpected services.

### Drill 33: Multi-Target Discovery
Goal: Find live hosts, then scan only those hosts.
1. Run a ping sweep on your lab range.
2. Save output with `-oA` for parsing.
3. Extract live hosts from the `.gnmap` file.
4. Run a service scan on the live hosts list.
5. Save the service scan output in a dated folder.
6. Review the results for open ports.
7. Write down any unexpected services.

### Drill 34: Multi-Target Discovery
Goal: Find live hosts, then scan only those hosts.
1. Run a ping sweep on your lab range.
2. Save output with `-oA` for parsing.
3. Extract live hosts from the `.gnmap` file.
4. Run a service scan on the live hosts list.
5. Save the service scan output in a dated folder.
6. Review the results for open ports.
7. Write down any unexpected services.

### Drill 35: Multi-Target Discovery
Goal: Find live hosts, then scan only those hosts.
1. Run a ping sweep on your lab range.
2. Save output with `-oA` for parsing.
3. Extract live hosts from the `.gnmap` file.
4. Run a service scan on the live hosts list.
5. Save the service scan output in a dated folder.
6. Review the results for open ports.
7. Write down any unexpected services.

### Drill 36: Multi-Target Discovery
Goal: Find live hosts, then scan only those hosts.
1. Run a ping sweep on your lab range.
2. Save output with `-oA` for parsing.
3. Extract live hosts from the `.gnmap` file.
4. Run a service scan on the live hosts list.
5. Save the service scan output in a dated folder.
6. Review the results for open ports.
7. Write down any unexpected services.

### Drill 37: Multi-Target Discovery
Goal: Find live hosts, then scan only those hosts.
1. Run a ping sweep on your lab range.
2. Save output with `-oA` for parsing.
3. Extract live hosts from the `.gnmap` file.
4. Run a service scan on the live hosts list.
5. Save the service scan output in a dated folder.
6. Review the results for open ports.
7. Write down any unexpected services.

### Drill 38: Multi-Target Discovery
Goal: Find live hosts, then scan only those hosts.
1. Run a ping sweep on your lab range.
2. Save output with `-oA` for parsing.
3. Extract live hosts from the `.gnmap` file.
4. Run a service scan on the live hosts list.
5. Save the service scan output in a dated folder.
6. Review the results for open ports.
7. Write down any unexpected services.

### Drill 39: Multi-Target Discovery
Goal: Find live hosts, then scan only those hosts.
1. Run a ping sweep on your lab range.
2. Save output with `-oA` for parsing.
3. Extract live hosts from the `.gnmap` file.
4. Run a service scan on the live hosts list.
5. Save the service scan output in a dated folder.
6. Review the results for open ports.
7. Write down any unexpected services.

### Drill 40: Multi-Target Discovery
Goal: Find live hosts, then scan only those hosts.
1. Run a ping sweep on your lab range.
2. Save output with `-oA` for parsing.
3. Extract live hosts from the `.gnmap` file.
4. Run a service scan on the live hosts list.
5. Save the service scan output in a dated folder.
6. Review the results for open ports.
7. Write down any unexpected services.

### Drill 41: Multi-Target Discovery
Goal: Find live hosts, then scan only those hosts.
1. Run a ping sweep on your lab range.
2. Save output with `-oA` for parsing.
3. Extract live hosts from the `.gnmap` file.
4. Run a service scan on the live hosts list.
5. Save the service scan output in a dated folder.
6. Review the results for open ports.
7. Write down any unexpected services.

### Drill 42: Multi-Target Discovery
Goal: Find live hosts, then scan only those hosts.
1. Run a ping sweep on your lab range.
2. Save output with `-oA` for parsing.
3. Extract live hosts from the `.gnmap` file.
4. Run a service scan on the live hosts list.
5. Save the service scan output in a dated folder.
6. Review the results for open ports.
7. Write down any unexpected services.

### Drill 43: Multi-Target Discovery
Goal: Find live hosts, then scan only those hosts.
1. Run a ping sweep on your lab range.
2. Save output with `-oA` for parsing.
3. Extract live hosts from the `.gnmap` file.
4. Run a service scan on the live hosts list.
5. Save the service scan output in a dated folder.
6. Review the results for open ports.
7. Write down any unexpected services.

### Drill 44: Multi-Target Discovery
Goal: Find live hosts, then scan only those hosts.
1. Run a ping sweep on your lab range.
2. Save output with `-oA` for parsing.
3. Extract live hosts from the `.gnmap` file.
4. Run a service scan on the live hosts list.
5. Save the service scan output in a dated folder.
6. Review the results for open ports.
7. Write down any unexpected services.

### Drill 45: Multi-Target Discovery
Goal: Find live hosts, then scan only those hosts.
1. Run a ping sweep on your lab range.
2. Save output with `-oA` for parsing.
3. Extract live hosts from the `.gnmap` file.
4. Run a service scan on the live hosts list.
5. Save the service scan output in a dated folder.
6. Review the results for open ports.
7. Write down any unexpected services.

### Drill 46: Multi-Target Discovery
Goal: Find live hosts, then scan only those hosts.
1. Run a ping sweep on your lab range.
2. Save output with `-oA` for parsing.
3. Extract live hosts from the `.gnmap` file.
4. Run a service scan on the live hosts list.
5. Save the service scan output in a dated folder.
6. Review the results for open ports.
7. Write down any unexpected services.

### Drill 47: Multi-Target Discovery
Goal: Find live hosts, then scan only those hosts.
1. Run a ping sweep on your lab range.
2. Save output with `-oA` for parsing.
3. Extract live hosts from the `.gnmap` file.
4. Run a service scan on the live hosts list.
5. Save the service scan output in a dated folder.
6. Review the results for open ports.
7. Write down any unexpected services.

### Drill 48: Multi-Target Discovery
Goal: Find live hosts, then scan only those hosts.
1. Run a ping sweep on your lab range.
2. Save output with `-oA` for parsing.
3. Extract live hosts from the `.gnmap` file.
4. Run a service scan on the live hosts list.
5. Save the service scan output in a dated folder.
6. Review the results for open ports.
7. Write down any unexpected services.

### Drill 49: Multi-Target Discovery
Goal: Find live hosts, then scan only those hosts.
1. Run a ping sweep on your lab range.
2. Save output with `-oA` for parsing.
3. Extract live hosts from the `.gnmap` file.
4. Run a service scan on the live hosts list.
5. Save the service scan output in a dated folder.
6. Review the results for open ports.
7. Write down any unexpected services.

### Drill 50: Multi-Target Discovery
Goal: Find live hosts, then scan only those hosts.
1. Run a ping sweep on your lab range.
2. Save output with `-oA` for parsing.
3. Extract live hosts from the `.gnmap` file.
4. Run a service scan on the live hosts list.
5. Save the service scan output in a dated folder.
6. Review the results for open ports.
7. Write down any unexpected services.

### Drill 51: Multi-Target Discovery
Goal: Find live hosts, then scan only those hosts.
1. Run a ping sweep on your lab range.
2. Save output with `-oA` for parsing.
3. Extract live hosts from the `.gnmap` file.
4. Run a service scan on the live hosts list.
5. Save the service scan output in a dated folder.
6. Review the results for open ports.
7. Write down any unexpected services.

### Drill 52: Multi-Target Discovery
Goal: Find live hosts, then scan only those hosts.
1. Run a ping sweep on your lab range.
2. Save output with `-oA` for parsing.
3. Extract live hosts from the `.gnmap` file.
4. Run a service scan on the live hosts list.
5. Save the service scan output in a dated folder.
6. Review the results for open ports.
7. Write down any unexpected services.

### Drill 53: Multi-Target Discovery
Goal: Find live hosts, then scan only those hosts.
1. Run a ping sweep on your lab range.
2. Save output with `-oA` for parsing.
3. Extract live hosts from the `.gnmap` file.
4. Run a service scan on the live hosts list.
5. Save the service scan output in a dated folder.
6. Review the results for open ports.
7. Write down any unexpected services.

### Drill 54: Multi-Target Discovery
Goal: Find live hosts, then scan only those hosts.
1. Run a ping sweep on your lab range.
2. Save output with `-oA` for parsing.
3. Extract live hosts from the `.gnmap` file.
4. Run a service scan on the live hosts list.
5. Save the service scan output in a dated folder.
6. Review the results for open ports.
7. Write down any unexpected services.

### Drill 55: Multi-Target Discovery
Goal: Find live hosts, then scan only those hosts.
1. Run a ping sweep on your lab range.
2. Save output with `-oA` for parsing.
3. Extract live hosts from the `.gnmap` file.
4. Run a service scan on the live hosts list.
5. Save the service scan output in a dated folder.
6. Review the results for open ports.
7. Write down any unexpected services.

### Drill 56: Multi-Target Discovery
Goal: Find live hosts, then scan only those hosts.
1. Run a ping sweep on your lab range.
2. Save output with `-oA` for parsing.
3. Extract live hosts from the `.gnmap` file.
4. Run a service scan on the live hosts list.
5. Save the service scan output in a dated folder.
6. Review the results for open ports.
7. Write down any unexpected services.

### Drill 57: Multi-Target Discovery
Goal: Find live hosts, then scan only those hosts.
1. Run a ping sweep on your lab range.
2. Save output with `-oA` for parsing.
3. Extract live hosts from the `.gnmap` file.
4. Run a service scan on the live hosts list.
5. Save the service scan output in a dated folder.
6. Review the results for open ports.
7. Write down any unexpected services.

### Drill 58: Multi-Target Discovery
Goal: Find live hosts, then scan only those hosts.
1. Run a ping sweep on your lab range.
2. Save output with `-oA` for parsing.
3. Extract live hosts from the `.gnmap` file.
4. Run a service scan on the live hosts list.
5. Save the service scan output in a dated folder.
6. Review the results for open ports.
7. Write down any unexpected services.

### Drill 59: Multi-Target Discovery
Goal: Find live hosts, then scan only those hosts.
1. Run a ping sweep on your lab range.
2. Save output with `-oA` for parsing.
3. Extract live hosts from the `.gnmap` file.
4. Run a service scan on the live hosts list.
5. Save the service scan output in a dated folder.
6. Review the results for open ports.
7. Write down any unexpected services.

### Drill 60: Multi-Target Discovery
Goal: Find live hosts, then scan only those hosts.
1. Run a ping sweep on your lab range.
2. Save output with `-oA` for parsing.
3. Extract live hosts from the `.gnmap` file.
4. Run a service scan on the live hosts list.
5. Save the service scan output in a dated folder.
6. Review the results for open ports.
7. Write down any unexpected services.

### Drill 61: Multi-Target Discovery
Goal: Find live hosts, then scan only those hosts.
1. Run a ping sweep on your lab range.
2. Save output with `-oA` for parsing.
3. Extract live hosts from the `.gnmap` file.
4. Run a service scan on the live hosts list.
5. Save the service scan output in a dated folder.
6. Review the results for open ports.
7. Write down any unexpected services.

### Drill 62: Multi-Target Discovery
Goal: Find live hosts, then scan only those hosts.
1. Run a ping sweep on your lab range.
2. Save output with `-oA` for parsing.
3. Extract live hosts from the `.gnmap` file.
4. Run a service scan on the live hosts list.
5. Save the service scan output in a dated folder.
6. Review the results for open ports.
7. Write down any unexpected services.

### Drill 63: Multi-Target Discovery
Goal: Find live hosts, then scan only those hosts.
1. Run a ping sweep on your lab range.
2. Save output with `-oA` for parsing.
3. Extract live hosts from the `.gnmap` file.
4. Run a service scan on the live hosts list.
5. Save the service scan output in a dated folder.
6. Review the results for open ports.
7. Write down any unexpected services.

### Drill 64: Multi-Target Discovery
Goal: Find live hosts, then scan only those hosts.
1. Run a ping sweep on your lab range.
2. Save output with `-oA` for parsing.
3. Extract live hosts from the `.gnmap` file.
4. Run a service scan on the live hosts list.
5. Save the service scan output in a dated folder.
6. Review the results for open ports.
7. Write down any unexpected services.

### Drill 65: Multi-Target Discovery
Goal: Find live hosts, then scan only those hosts.
1. Run a ping sweep on your lab range.
2. Save output with `-oA` for parsing.
3. Extract live hosts from the `.gnmap` file.
4. Run a service scan on the live hosts list.
5. Save the service scan output in a dated folder.
6. Review the results for open ports.
7. Write down any unexpected services.

### Drill 66: Multi-Target Discovery
Goal: Find live hosts, then scan only those hosts.
1. Run a ping sweep on your lab range.
2. Save output with `-oA` for parsing.
3. Extract live hosts from the `.gnmap` file.
4. Run a service scan on the live hosts list.
5. Save the service scan output in a dated folder.
6. Review the results for open ports.
7. Write down any unexpected services.

### Drill 67: Multi-Target Discovery
Goal: Find live hosts, then scan only those hosts.
1. Run a ping sweep on your lab range.
2. Save output with `-oA` for parsing.
3. Extract live hosts from the `.gnmap` file.
4. Run a service scan on the live hosts list.
5. Save the service scan output in a dated folder.
6. Review the results for open ports.
7. Write down any unexpected services.

### Drill 68: Multi-Target Discovery
Goal: Find live hosts, then scan only those hosts.
1. Run a ping sweep on your lab range.
2. Save output with `-oA` for parsing.
3. Extract live hosts from the `.gnmap` file.
4. Run a service scan on the live hosts list.
5. Save the service scan output in a dated folder.
6. Review the results for open ports.
7. Write down any unexpected services.

### Drill 69: Multi-Target Discovery
Goal: Find live hosts, then scan only those hosts.
1. Run a ping sweep on your lab range.
2. Save output with `-oA` for parsing.
3. Extract live hosts from the `.gnmap` file.
4. Run a service scan on the live hosts list.
5. Save the service scan output in a dated folder.
6. Review the results for open ports.
7. Write down any unexpected services.

### Drill 70: Multi-Target Discovery
Goal: Find live hosts, then scan only those hosts.
1. Run a ping sweep on your lab range.
2. Save output with `-oA` for parsing.
3. Extract live hosts from the `.gnmap` file.
4. Run a service scan on the live hosts list.
5. Save the service scan output in a dated folder.
6. Review the results for open ports.
7. Write down any unexpected services.

### Drill 71: Multi-Target Discovery
Goal: Find live hosts, then scan only those hosts.
1. Run a ping sweep on your lab range.
2. Save output with `-oA` for parsing.
3. Extract live hosts from the `.gnmap` file.
4. Run a service scan on the live hosts list.
5. Save the service scan output in a dated folder.
6. Review the results for open ports.
7. Write down any unexpected services.

### Drill 72: Multi-Target Discovery
Goal: Find live hosts, then scan only those hosts.
1. Run a ping sweep on your lab range.
2. Save output with `-oA` for parsing.
3. Extract live hosts from the `.gnmap` file.
4. Run a service scan on the live hosts list.
5. Save the service scan output in a dated folder.
6. Review the results for open ports.
7. Write down any unexpected services.

### Drill 73: Multi-Target Discovery
Goal: Find live hosts, then scan only those hosts.
1. Run a ping sweep on your lab range.
2. Save output with `-oA` for parsing.
3. Extract live hosts from the `.gnmap` file.
4. Run a service scan on the live hosts list.
5. Save the service scan output in a dated folder.
6. Review the results for open ports.
7. Write down any unexpected services.

### Drill 74: Multi-Target Discovery
Goal: Find live hosts, then scan only those hosts.
1. Run a ping sweep on your lab range.
2. Save output with `-oA` for parsing.
3. Extract live hosts from the `.gnmap` file.
4. Run a service scan on the live hosts list.
5. Save the service scan output in a dated folder.
6. Review the results for open ports.
7. Write down any unexpected services.

### Drill 75: Multi-Target Discovery
Goal: Find live hosts, then scan only those hosts.
1. Run a ping sweep on your lab range.
2. Save output with `-oA` for parsing.
3. Extract live hosts from the `.gnmap` file.
4. Run a service scan on the live hosts list.
5. Save the service scan output in a dated folder.
6. Review the results for open ports.
7. Write down any unexpected services.

### Drill 76: Multi-Target Discovery
Goal: Find live hosts, then scan only those hosts.
1. Run a ping sweep on your lab range.
2. Save output with `-oA` for parsing.
3. Extract live hosts from the `.gnmap` file.
4. Run a service scan on the live hosts list.
5. Save the service scan output in a dated folder.
6. Review the results for open ports.
7. Write down any unexpected services.

### Drill 77: Multi-Target Discovery
Goal: Find live hosts, then scan only those hosts.
1. Run a ping sweep on your lab range.
2. Save output with `-oA` for parsing.
3. Extract live hosts from the `.gnmap` file.
4. Run a service scan on the live hosts list.
5. Save the service scan output in a dated folder.
6. Review the results for open ports.
7. Write down any unexpected services.

### Drill 78: Multi-Target Discovery
Goal: Find live hosts, then scan only those hosts.
1. Run a ping sweep on your lab range.
2. Save output with `-oA` for parsing.
3. Extract live hosts from the `.gnmap` file.
4. Run a service scan on the live hosts list.
5. Save the service scan output in a dated folder.
6. Review the results for open ports.
7. Write down any unexpected services.

### Drill 79: Multi-Target Discovery
Goal: Find live hosts, then scan only those hosts.
1. Run a ping sweep on your lab range.
2. Save output with `-oA` for parsing.
3. Extract live hosts from the `.gnmap` file.
4. Run a service scan on the live hosts list.
5. Save the service scan output in a dated folder.
6. Review the results for open ports.
7. Write down any unexpected services.

### Drill 80: Multi-Target Discovery
Goal: Find live hosts, then scan only those hosts.
1. Run a ping sweep on your lab range.
2. Save output with `-oA` for parsing.
3. Extract live hosts from the `.gnmap` file.
4. Run a service scan on the live hosts list.
5. Save the service scan output in a dated folder.
6. Review the results for open ports.
7. Write down any unexpected services.

### Drill 81: Multi-Target Discovery
Goal: Find live hosts, then scan only those hosts.
1. Run a ping sweep on your lab range.
2. Save output with `-oA` for parsing.
3. Extract live hosts from the `.gnmap` file.
4. Run a service scan on the live hosts list.
5. Save the service scan output in a dated folder.
6. Review the results for open ports.
7. Write down any unexpected services.

### Drill 82: Multi-Target Discovery
Goal: Find live hosts, then scan only those hosts.
1. Run a ping sweep on your lab range.
2. Save output with `-oA` for parsing.
3. Extract live hosts from the `.gnmap` file.
4. Run a service scan on the live hosts list.
5. Save the service scan output in a dated folder.
6. Review the results for open ports.
7. Write down any unexpected services.

### Drill 83: Multi-Target Discovery
Goal: Find live hosts, then scan only those hosts.
1. Run a ping sweep on your lab range.
2. Save output with `-oA` for parsing.
3. Extract live hosts from the `.gnmap` file.
4. Run a service scan on the live hosts list.
5. Save the service scan output in a dated folder.
6. Review the results for open ports.
7. Write down any unexpected services.

### Drill 84: Multi-Target Discovery
Goal: Find live hosts, then scan only those hosts.
1. Run a ping sweep on your lab range.
2. Save output with `-oA` for parsing.
3. Extract live hosts from the `.gnmap` file.
4. Run a service scan on the live hosts list.
5. Save the service scan output in a dated folder.
6. Review the results for open ports.
7. Write down any unexpected services.

### Drill 85: Multi-Target Discovery
Goal: Find live hosts, then scan only those hosts.
1. Run a ping sweep on your lab range.
2. Save output with `-oA` for parsing.
3. Extract live hosts from the `.gnmap` file.
4. Run a service scan on the live hosts list.
5. Save the service scan output in a dated folder.
6. Review the results for open ports.
7. Write down any unexpected services.

### Drill 86: Multi-Target Discovery
Goal: Find live hosts, then scan only those hosts.
1. Run a ping sweep on your lab range.
2. Save output with `-oA` for parsing.
3. Extract live hosts from the `.gnmap` file.
4. Run a service scan on the live hosts list.
5. Save the service scan output in a dated folder.
6. Review the results for open ports.
7. Write down any unexpected services.

### Drill 87: Multi-Target Discovery
Goal: Find live hosts, then scan only those hosts.
1. Run a ping sweep on your lab range.
2. Save output with `-oA` for parsing.
3. Extract live hosts from the `.gnmap` file.
4. Run a service scan on the live hosts list.
5. Save the service scan output in a dated folder.
6. Review the results for open ports.
7. Write down any unexpected services.

### Drill 88: Multi-Target Discovery
Goal: Find live hosts, then scan only those hosts.
1. Run a ping sweep on your lab range.
2. Save output with `-oA` for parsing.
3. Extract live hosts from the `.gnmap` file.
4. Run a service scan on the live hosts list.
5. Save the service scan output in a dated folder.
6. Review the results for open ports.
7. Write down any unexpected services.

### Drill 89: Multi-Target Discovery
Goal: Find live hosts, then scan only those hosts.
1. Run a ping sweep on your lab range.
2. Save output with `-oA` for parsing.
3. Extract live hosts from the `.gnmap` file.
4. Run a service scan on the live hosts list.
5. Save the service scan output in a dated folder.
6. Review the results for open ports.
7. Write down any unexpected services.

### Drill 90: Multi-Target Discovery
Goal: Find live hosts, then scan only those hosts.
1. Run a ping sweep on your lab range.
2. Save output with `-oA` for parsing.
3. Extract live hosts from the `.gnmap` file.
4. Run a service scan on the live hosts list.
5. Save the service scan output in a dated folder.
6. Review the results for open ports.
7. Write down any unexpected services.

### Drill 91: Multi-Target Discovery
Goal: Find live hosts, then scan only those hosts.
1. Run a ping sweep on your lab range.
2. Save output with `-oA` for parsing.
3. Extract live hosts from the `.gnmap` file.
4. Run a service scan on the live hosts list.
5. Save the service scan output in a dated folder.
6. Review the results for open ports.
7. Write down any unexpected services.

### Drill 92: Multi-Target Discovery
Goal: Find live hosts, then scan only those hosts.
1. Run a ping sweep on your lab range.
2. Save output with `-oA` for parsing.
3. Extract live hosts from the `.gnmap` file.
4. Run a service scan on the live hosts list.
5. Save the service scan output in a dated folder.
6. Review the results for open ports.
7. Write down any unexpected services.

### Drill 93: Multi-Target Discovery
Goal: Find live hosts, then scan only those hosts.
1. Run a ping sweep on your lab range.
2. Save output with `-oA` for parsing.
3. Extract live hosts from the `.gnmap` file.
4. Run a service scan on the live hosts list.
5. Save the service scan output in a dated folder.
6. Review the results for open ports.
7. Write down any unexpected services.

### Drill 94: Multi-Target Discovery
Goal: Find live hosts, then scan only those hosts.
1. Run a ping sweep on your lab range.
2. Save output with `-oA` for parsing.
3. Extract live hosts from the `.gnmap` file.
4. Run a service scan on the live hosts list.
5. Save the service scan output in a dated folder.
6. Review the results for open ports.
7. Write down any unexpected services.

### Drill 95: Multi-Target Discovery
Goal: Find live hosts, then scan only those hosts.
1. Run a ping sweep on your lab range.
2. Save output with `-oA` for parsing.
3. Extract live hosts from the `.gnmap` file.
4. Run a service scan on the live hosts list.
5. Save the service scan output in a dated folder.
6. Review the results for open ports.
7. Write down any unexpected services.

### Drill 96: Multi-Target Discovery
Goal: Find live hosts, then scan only those hosts.
1. Run a ping sweep on your lab range.
2. Save output with `-oA` for parsing.
3. Extract live hosts from the `.gnmap` file.
4. Run a service scan on the live hosts list.
5. Save the service scan output in a dated folder.
6. Review the results for open ports.
7. Write down any unexpected services.

### Drill 97: Multi-Target Discovery
Goal: Find live hosts, then scan only those hosts.
1. Run a ping sweep on your lab range.
2. Save output with `-oA` for parsing.
3. Extract live hosts from the `.gnmap` file.
4. Run a service scan on the live hosts list.
5. Save the service scan output in a dated folder.
6. Review the results for open ports.
7. Write down any unexpected services.

### Drill 98: Multi-Target Discovery
Goal: Find live hosts, then scan only those hosts.
1. Run a ping sweep on your lab range.
2. Save output with `-oA` for parsing.
3. Extract live hosts from the `.gnmap` file.
4. Run a service scan on the live hosts list.
5. Save the service scan output in a dated folder.
6. Review the results for open ports.
7. Write down any unexpected services.

### Drill 99: Multi-Target Discovery
Goal: Find live hosts, then scan only those hosts.
1. Run a ping sweep on your lab range.
2. Save output with `-oA` for parsing.
3. Extract live hosts from the `.gnmap` file.
4. Run a service scan on the live hosts list.
5. Save the service scan output in a dated folder.
6. Review the results for open ports.
7. Write down any unexpected services.

### Drill 100: Multi-Target Discovery
Goal: Find live hosts, then scan only those hosts.
1. Run a ping sweep on your lab range.
2. Save output with `-oA` for parsing.
3. Extract live hosts from the `.gnmap` file.
4. Run a service scan on the live hosts list.
5. Save the service scan output in a dated folder.
6. Review the results for open ports.
7. Write down any unexpected services.

```powershell

## Command Reference (One-Liners)
- Example 1: `nmap -sV 10.0.1.0/24 -oA outputs/example-1`
- Example 2: `nmap -sV 10.0.2.0/24 -oA outputs/example-2`
- Example 3: `nmap -sV 10.0.3.0/24 -oA outputs/example-3`
- Example 4: `nmap -sV 10.0.4.0/24 -oA outputs/example-4`
- Example 5: `nmap -sV 10.0.5.0/24 -oA outputs/example-5`
- Example 6: `nmap -sV 10.0.6.0/24 -oA outputs/example-6`
- Example 7: `nmap -sV 10.0.7.0/24 -oA outputs/example-7`
- Example 8: `nmap -sV 10.0.8.0/24 -oA outputs/example-8`
- Example 9: `nmap -sV 10.0.9.0/24 -oA outputs/example-9`
- Example 10: `nmap -sV 10.0.0.0/24 -oA outputs/example-10`
- Example 11: `nmap -sV 10.0.1.0/24 -oA outputs/example-11`
- Example 12: `nmap -sV 10.0.2.0/24 -oA outputs/example-12`
- Example 13: `nmap -sV 10.0.3.0/24 -oA outputs/example-13`
- Example 14: `nmap -sV 10.0.4.0/24 -oA outputs/example-14`
- Example 15: `nmap -sV 10.0.5.0/24 -oA outputs/example-15`
- Example 16: `nmap -sV 10.0.6.0/24 -oA outputs/example-16`
- Example 17: `nmap -sV 10.0.7.0/24 -oA outputs/example-17`
- Example 18: `nmap -sV 10.0.8.0/24 -oA outputs/example-18`
- Example 19: `nmap -sV 10.0.9.0/24 -oA outputs/example-19`
- Example 20: `nmap -sV 10.0.0.0/24 -oA outputs/example-20`
- Example 21: `nmap -sV 10.0.1.0/24 -oA outputs/example-21`
- Example 22: `nmap -sV 10.0.2.0/24 -oA outputs/example-22`
- Example 23: `nmap -sV 10.0.3.0/24 -oA outputs/example-23`
- Example 24: `nmap -sV 10.0.4.0/24 -oA outputs/example-24`
- Example 25: `nmap -sV 10.0.5.0/24 -oA outputs/example-25`
- Example 26: `nmap -sV 10.0.6.0/24 -oA outputs/example-26`
- Example 27: `nmap -sV 10.0.7.0/24 -oA outputs/example-27`
- Example 28: `nmap -sV 10.0.8.0/24 -oA outputs/example-28`
- Example 29: `nmap -sV 10.0.9.0/24 -oA outputs/example-29`
- Example 30: `nmap -sV 10.0.0.0/24 -oA outputs/example-30`
- Example 31: `nmap -sV 10.0.1.0/24 -oA outputs/example-31`
- Example 32: `nmap -sV 10.0.2.0/24 -oA outputs/example-32`
- Example 33: `nmap -sV 10.0.3.0/24 -oA outputs/example-33`
- Example 34: `nmap -sV 10.0.4.0/24 -oA outputs/example-34`
- Example 35: `nmap -sV 10.0.5.0/24 -oA outputs/example-35`
- Example 36: `nmap -sV 10.0.6.0/24 -oA outputs/example-36`
- Example 37: `nmap -sV 10.0.7.0/24 -oA outputs/example-37`
- Example 38: `nmap -sV 10.0.8.0/24 -oA outputs/example-38`
- Example 39: `nmap -sV 10.0.9.0/24 -oA outputs/example-39`
- Example 40: `nmap -sV 10.0.0.0/24 -oA outputs/example-40`
- Example 41: `nmap -sV 10.0.1.0/24 -oA outputs/example-41`
- Example 42: `nmap -sV 10.0.2.0/24 -oA outputs/example-42`
- Example 43: `nmap -sV 10.0.3.0/24 -oA outputs/example-43`
- Example 44: `nmap -sV 10.0.4.0/24 -oA outputs/example-44`
- Example 45: `nmap -sV 10.0.5.0/24 -oA outputs/example-45`
- Example 46: `nmap -sV 10.0.6.0/24 -oA outputs/example-46`
- Example 47: `nmap -sV 10.0.7.0/24 -oA outputs/example-47`
- Example 48: `nmap -sV 10.0.8.0/24 -oA outputs/example-48`
- Example 49: `nmap -sV 10.0.9.0/24 -oA outputs/example-49`
- Example 50: `nmap -sV 10.0.0.0/24 -oA outputs/example-50`
- Example 51: `nmap -sV 10.0.1.0/24 -oA outputs/example-51`
- Example 52: `nmap -sV 10.0.2.0/24 -oA outputs/example-52`
- Example 53: `nmap -sV 10.0.3.0/24 -oA outputs/example-53`
- Example 54: `nmap -sV 10.0.4.0/24 -oA outputs/example-54`
- Example 55: `nmap -sV 10.0.5.0/24 -oA outputs/example-55`
- Example 56: `nmap -sV 10.0.6.0/24 -oA outputs/example-56`
- Example 57: `nmap -sV 10.0.7.0/24 -oA outputs/example-57`
- Example 58: `nmap -sV 10.0.8.0/24 -oA outputs/example-58`
- Example 59: `nmap -sV 10.0.9.0/24 -oA outputs/example-59`
- Example 60: `nmap -sV 10.0.0.0/24 -oA outputs/example-60`
- Example 61: `nmap -sV 10.0.1.0/24 -oA outputs/example-61`
- Example 62: `nmap -sV 10.0.2.0/24 -oA outputs/example-62`
- Example 63: `nmap -sV 10.0.3.0/24 -oA outputs/example-63`
- Example 64: `nmap -sV 10.0.4.0/24 -oA outputs/example-64`
- Example 65: `nmap -sV 10.0.5.0/24 -oA outputs/example-65`
- Example 66: `nmap -sV 10.0.6.0/24 -oA outputs/example-66`
- Example 67: `nmap -sV 10.0.7.0/24 -oA outputs/example-67`
- Example 68: `nmap -sV 10.0.8.0/24 -oA outputs/example-68`
- Example 69: `nmap -sV 10.0.9.0/24 -oA outputs/example-69`
- Example 70: `nmap -sV 10.0.0.0/24 -oA outputs/example-70`
- Example 71: `nmap -sV 10.0.1.0/24 -oA outputs/example-71`
- Example 72: `nmap -sV 10.0.2.0/24 -oA outputs/example-72`
- Example 73: `nmap -sV 10.0.3.0/24 -oA outputs/example-73`
- Example 74: `nmap -sV 10.0.4.0/24 -oA outputs/example-74`
- Example 75: `nmap -sV 10.0.5.0/24 -oA outputs/example-75`
- Example 76: `nmap -sV 10.0.6.0/24 -oA outputs/example-76`
- Example 77: `nmap -sV 10.0.7.0/24 -oA outputs/example-77`
- Example 78: `nmap -sV 10.0.8.0/24 -oA outputs/example-78`
- Example 79: `nmap -sV 10.0.9.0/24 -oA outputs/example-79`
- Example 80: `nmap -sV 10.0.0.0/24 -oA outputs/example-80`
- Example 81: `nmap -sV 10.0.1.0/24 -oA outputs/example-81`
- Example 82: `nmap -sV 10.0.2.0/24 -oA outputs/example-82`
- Example 83: `nmap -sV 10.0.3.0/24 -oA outputs/example-83`
- Example 84: `nmap -sV 10.0.4.0/24 -oA outputs/example-84`
- Example 85: `nmap -sV 10.0.5.0/24 -oA outputs/example-85`
- Example 86: `nmap -sV 10.0.6.0/24 -oA outputs/example-86`
- Example 87: `nmap -sV 10.0.7.0/24 -oA outputs/example-87`
- Example 88: `nmap -sV 10.0.8.0/24 -oA outputs/example-88`
- Example 89: `nmap -sV 10.0.9.0/24 -oA outputs/example-89`
- Example 90: `nmap -sV 10.0.0.0/24 -oA outputs/example-90`
- Example 91: `nmap -sV 10.0.1.0/24 -oA outputs/example-91`
- Example 92: `nmap -sV 10.0.2.0/24 -oA outputs/example-92`
- Example 93: `nmap -sV 10.0.3.0/24 -oA outputs/example-93`
- Example 94: `nmap -sV 10.0.4.0/24 -oA outputs/example-94`
- Example 95: `nmap -sV 10.0.5.0/24 -oA outputs/example-95`
- Example 96: `nmap -sV 10.0.6.0/24 -oA outputs/example-96`
- Example 97: `nmap -sV 10.0.7.0/24 -oA outputs/example-97`
- Example 98: `nmap -sV 10.0.8.0/24 -oA outputs/example-98`
- Example 99: `nmap -sV 10.0.9.0/24 -oA outputs/example-99`
- Example 100: `nmap -sV 10.0.0.0/24 -oA outputs/example-100`
- Example 101: `nmap -sV 10.0.1.0/24 -oA outputs/example-101`
- Example 102: `nmap -sV 10.0.2.0/24 -oA outputs/example-102`
- Example 103: `nmap -sV 10.0.3.0/24 -oA outputs/example-103`
- Example 104: `nmap -sV 10.0.4.0/24 -oA outputs/example-104`
- Example 105: `nmap -sV 10.0.5.0/24 -oA outputs/example-105`
- Example 106: `nmap -sV 10.0.6.0/24 -oA outputs/example-106`
- Example 107: `nmap -sV 10.0.7.0/24 -oA outputs/example-107`
- Example 108: `nmap -sV 10.0.8.0/24 -oA outputs/example-108`
- Example 109: `nmap -sV 10.0.9.0/24 -oA outputs/example-109`
- Example 110: `nmap -sV 10.0.0.0/24 -oA outputs/example-110`
- Example 111: `nmap -sV 10.0.1.0/24 -oA outputs/example-111`
- Example 112: `nmap -sV 10.0.2.0/24 -oA outputs/example-112`
- Example 113: `nmap -sV 10.0.3.0/24 -oA outputs/example-113`
- Example 114: `nmap -sV 10.0.4.0/24 -oA outputs/example-114`
- Example 115: `nmap -sV 10.0.5.0/24 -oA outputs/example-115`
- Example 116: `nmap -sV 10.0.6.0/24 -oA outputs/example-116`
- Example 117: `nmap -sV 10.0.7.0/24 -oA outputs/example-117`
- Example 118: `nmap -sV 10.0.8.0/24 -oA outputs/example-118`
- Example 119: `nmap -sV 10.0.9.0/24 -oA outputs/example-119`
- Example 120: `nmap -sV 10.0.0.0/24 -oA outputs/example-120`
- Example 121: `nmap -sV 10.0.1.0/24 -oA outputs/example-121`
- Example 122: `nmap -sV 10.0.2.0/24 -oA outputs/example-122`
- Example 123: `nmap -sV 10.0.3.0/24 -oA outputs/example-123`
- Example 124: `nmap -sV 10.0.4.0/24 -oA outputs/example-124`
- Example 125: `nmap -sV 10.0.5.0/24 -oA outputs/example-125`
- Example 126: `nmap -sV 10.0.6.0/24 -oA outputs/example-126`
- Example 127: `nmap -sV 10.0.7.0/24 -oA outputs/example-127`
- Example 128: `nmap -sV 10.0.8.0/24 -oA outputs/example-128`
- Example 129: `nmap -sV 10.0.9.0/24 -oA outputs/example-129`
- Example 130: `nmap -sV 10.0.0.0/24 -oA outputs/example-130`
- Example 131: `nmap -sV 10.0.1.0/24 -oA outputs/example-131`
- Example 132: `nmap -sV 10.0.2.0/24 -oA outputs/example-132`
- Example 133: `nmap -sV 10.0.3.0/24 -oA outputs/example-133`
- Example 134: `nmap -sV 10.0.4.0/24 -oA outputs/example-134`
- Example 135: `nmap -sV 10.0.5.0/24 -oA outputs/example-135`
- Example 136: `nmap -sV 10.0.6.0/24 -oA outputs/example-136`
- Example 137: `nmap -sV 10.0.7.0/24 -oA outputs/example-137`
- Example 138: `nmap -sV 10.0.8.0/24 -oA outputs/example-138`
- Example 139: `nmap -sV 10.0.9.0/24 -oA outputs/example-139`
- Example 140: `nmap -sV 10.0.0.0/24 -oA outputs/example-140`
- Example 141: `nmap -sV 10.0.1.0/24 -oA outputs/example-141`
- Example 142: `nmap -sV 10.0.2.0/24 -oA outputs/example-142`
- Example 143: `nmap -sV 10.0.3.0/24 -oA outputs/example-143`
- Example 144: `nmap -sV 10.0.4.0/24 -oA outputs/example-144`
- Example 145: `nmap -sV 10.0.5.0/24 -oA outputs/example-145`
- Example 146: `nmap -sV 10.0.6.0/24 -oA outputs/example-146`
- Example 147: `nmap -sV 10.0.7.0/24 -oA outputs/example-147`
- Example 148: `nmap -sV 10.0.8.0/24 -oA outputs/example-148`
- Example 149: `nmap -sV 10.0.9.0/24 -oA outputs/example-149`
- Example 150: `nmap -sV 10.0.0.0/24 -oA outputs/example-150`
- Example 151: `nmap -sV 10.0.1.0/24 -oA outputs/example-151`
- Example 152: `nmap -sV 10.0.2.0/24 -oA outputs/example-152`
- Example 153: `nmap -sV 10.0.3.0/24 -oA outputs/example-153`
- Example 154: `nmap -sV 10.0.4.0/24 -oA outputs/example-154`
- Example 155: `nmap -sV 10.0.5.0/24 -oA outputs/example-155`
- Example 156: `nmap -sV 10.0.6.0/24 -oA outputs/example-156`
- Example 157: `nmap -sV 10.0.7.0/24 -oA outputs/example-157`
- Example 158: `nmap -sV 10.0.8.0/24 -oA outputs/example-158`
- Example 159: `nmap -sV 10.0.9.0/24 -oA outputs/example-159`
- Example 160: `nmap -sV 10.0.0.0/24 -oA outputs/example-160`
- Example 161: `nmap -sV 10.0.1.0/24 -oA outputs/example-161`
- Example 162: `nmap -sV 10.0.2.0/24 -oA outputs/example-162`
- Example 163: `nmap -sV 10.0.3.0/24 -oA outputs/example-163`
- Example 164: `nmap -sV 10.0.4.0/24 -oA outputs/example-164`
- Example 165: `nmap -sV 10.0.5.0/24 -oA outputs/example-165`
- Example 166: `nmap -sV 10.0.6.0/24 -oA outputs/example-166`
- Example 167: `nmap -sV 10.0.7.0/24 -oA outputs/example-167`
- Example 168: `nmap -sV 10.0.8.0/24 -oA outputs/example-168`
- Example 169: `nmap -sV 10.0.9.0/24 -oA outputs/example-169`
- Example 170: `nmap -sV 10.0.0.0/24 -oA outputs/example-170`
- Example 171: `nmap -sV 10.0.1.0/24 -oA outputs/example-171`
- Example 172: `nmap -sV 10.0.2.0/24 -oA outputs/example-172`
- Example 173: `nmap -sV 10.0.3.0/24 -oA outputs/example-173`
- Example 174: `nmap -sV 10.0.4.0/24 -oA outputs/example-174`
- Example 175: `nmap -sV 10.0.5.0/24 -oA outputs/example-175`
- Example 176: `nmap -sV 10.0.6.0/24 -oA outputs/example-176`
- Example 177: `nmap -sV 10.0.7.0/24 -oA outputs/example-177`
- Example 178: `nmap -sV 10.0.8.0/24 -oA outputs/example-178`
- Example 179: `nmap -sV 10.0.9.0/24 -oA outputs/example-179`
- Example 180: `nmap -sV 10.0.0.0/24 -oA outputs/example-180`
- Example 181: `nmap -sV 10.0.1.0/24 -oA outputs/example-181`
- Example 182: `nmap -sV 10.0.2.0/24 -oA outputs/example-182`
- Example 183: `nmap -sV 10.0.3.0/24 -oA outputs/example-183`
- Example 184: `nmap -sV 10.0.4.0/24 -oA outputs/example-184`
- Example 185: `nmap -sV 10.0.5.0/24 -oA outputs/example-185`
- Example 186: `nmap -sV 10.0.6.0/24 -oA outputs/example-186`
- Example 187: `nmap -sV 10.0.7.0/24 -oA outputs/example-187`
- Example 188: `nmap -sV 10.0.8.0/24 -oA outputs/example-188`
- Example 189: `nmap -sV 10.0.9.0/24 -oA outputs/example-189`
- Example 190: `nmap -sV 10.0.0.0/24 -oA outputs/example-190`
- Example 191: `nmap -sV 10.0.1.0/24 -oA outputs/example-191`
- Example 192: `nmap -sV 10.0.2.0/24 -oA outputs/example-192`
- Example 193: `nmap -sV 10.0.3.0/24 -oA outputs/example-193`
- Example 194: `nmap -sV 10.0.4.0/24 -oA outputs/example-194`
- Example 195: `nmap -sV 10.0.5.0/24 -oA outputs/example-195`
- Example 196: `nmap -sV 10.0.6.0/24 -oA outputs/example-196`
- Example 197: `nmap -sV 10.0.7.0/24 -oA outputs/example-197`
- Example 198: `nmap -sV 10.0.8.0/24 -oA outputs/example-198`
- Example 199: `nmap -sV 10.0.9.0/24 -oA outputs/example-199`
- Example 200: `nmap -sV 10.0.0.0/24 -oA outputs/example-200`
```
## Reflection Questions
1. Which targeting method is easiest for you to remember?
2. When would you use a target file instead of a CIDR range?
3. How does Day 17 output parsing help with Day 19 scanning?
4. How does Day 18 automation reduce mistakes in Day 19 scans?
5. What limits would you set before scanning a /16 network?

## Next Steps
Tomorrow (Day 20) you will learn about stealth and evasion techniques.

**Congratulations on completing Day 19!** You can now scan multiple targets and ranges safely and consistently.
