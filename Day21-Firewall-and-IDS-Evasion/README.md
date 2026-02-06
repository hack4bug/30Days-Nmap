# Day 21 - Firewall and IDS Evasion

## Day Overview
Welcome to Day 21! Today you will learn how firewalls and IDS/IPS systems detect scans and how to reduce noise in authorized testing environments.
This topic is sensitive, so we keep it ethical, beginner-friendly, and focused on safe, permitted use.

This lesson builds on the last three days:
- Day 18: Automation with shell scripts
- Day 19: Scanning multiple targets and ranges
- Day 20: Stealth and evasion techniques

## Learning Objectives
By completing Day 21, you will be able to:
- Understand how firewalls and IDS/IPS detect scans
- Use safer scan options that reduce alerts
- Adjust timing, rate, and scope for controlled testing
- Combine evasion options with proper logging
- Know the limits of evasion techniques

## Important Ethics Note
Firewall and IDS evasion must only be used with explicit permission.
Do not attempt to hide scanning activity on networks you do not own or control.

## What Are Firewalls and IDS/IPS?
- Firewalls enforce traffic rules based on ports, protocols, and IPs
- IDS (Intrusion Detection Systems) detect suspicious traffic
- IPS (Intrusion Prevention Systems) can block traffic

## How Detection Usually Happens
Detection typically occurs when scans are:
- Fast and noisy
- Probing many ports
- Sending unusual packet patterns
- Repeated across many hosts in order

## Day 18 Recap: Automation and Safety
Automate scanning to keep options consistent and logged.
Use scripts to ensure you always save outputs and scope rules.

## Day 19 Recap: Scope Control
Reduce scope to reduce alerts. Use target lists and exclude files.

## Day 20 Recap: Stealth Basics
Stealth options reduce noise but do not make scans invisible.

## Core Evasion Concepts (Beginner-Friendly)

### 1. Slow Down the Scan
```bash
nmap -sS -T2 --max-rate 50 target
```
Slower scans reduce IDS/IPS triggers.

### 2. Limit Ports to What You Need
```bash
nmap -sS -p 22,80,443 target
```
Fewer probes = fewer alerts.

### 3. Randomize Host Order
```bash
nmap -sS --randomize-hosts 192.168.1.0/24
```
Avoid obvious sequential patterns.

### 4. Use Host Discovery First
```bash
nmap -sn 192.168.1.0/24 -oA outputs/ping-sweep
```
Then scan only live hosts.

### 5. Disable DNS Resolution
```bash
nmap -n -sS target
```
Reduces extra DNS traffic and logs.

## Common Firewall Behavior
- Block all ports except allowed services
- Drop packets silently (no response)
- Rate-limit or throttle traffic
- Trigger alerts on repeated probes

## IDS/IPS Behavior Patterns
- Alerts on port scans across many ports
- Detects unusual TCP flags or malformed packets
- Correlates activity across time
- Blocks or blacklists IPs after detection

## Evasion Options (Advanced, Use with Care)
These options exist in Nmap, but should only be used in labs or permitted tests.

### 1. Fragmentation
```bash
sudo nmap -sS -f target
```
May bypass basic filters, but often blocked by modern systems.

### 2. Decoys
```bash
sudo nmap -sS -D RND:5 target
```
Adds fake source IPs to hide your real one among decoys.

### 3. Source Port Manipulation
```bash
sudo nmap -sS --source-port 53 target
```
Some old filters allow traffic from trusted ports.

### 4. Idle Scan
```bash
sudo nmap -sI zombie-host target
```
Very advanced. Only for labs and authorized exercises.

## Evasion Limitations
- Modern IDS/IPS can still detect stealth scans
- Firewalls can drop or rate-limit suspicious traffic
- Evasion can reduce accuracy of results
- Logs are often still generated

## Safe Evasion Strategy for Beginners
- Use slow timing
- Limit port lists
- Randomize hosts
- Keep scope small
- Log everything

## Practical Evasion Examples

### Example 1: Conservative Scan
```bash
sudo nmap -sS -T2 --max-rate 30 -p 22,80,443 target -oA outputs/conservative
```

### Example 2: Conservative Range Scan
```bash
sudo nmap -sS -T2 --randomize-hosts 10.0.0.0/24 -oA outputs/conservative-range
```

### Example 3: Discovery First, Then Scan
```bash
nmap -sn 10.0.0.0/24 -oA outputs/discovery
awk '/Up$/{print $2}' outputs/discovery.gnmap > outputs/live.txt
sudo nmap -sS -T2 -iL outputs/live.txt -oA outputs/live-scan
```

## Combining with Automation (Day 18)
```bash
#!/usr/bin/env bash
set -euo pipefail
TARGET_FILE="${1:-targets.txt}"
RUN_DIR="outputs/$(date +%Y%m%d-%H%M%S)"
mkdir -p "$RUN_DIR"
sudo nmap -sS -T2 --max-rate 50 --randomize-hosts -iL "$TARGET_FILE" -oA "$RUN_DIR/evasion"
```

## Combining with Multi-Target Scanning (Day 19)
```bash
sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 192.168.1.0/24 -oA outputs/evasion-range
```

## Logging and Reporting
Always record:
- Time and date
- Target scope
- Options used
- Output file paths

## Beginner-Friendly Checklists
### Pre-Scan Checklist
- I have written permission
- I confirmed scope and exclusions
- I selected safe timing options
- I prepared output folders
- I documented the scan plan

### Post-Scan Checklist
- I saved all outputs
- I reviewed results for open ports
- I documented alerts or anomalies
- I stored scan logs safely

## Troubleshooting

### No Responses From Hosts
```text
Fix:
- Firewalls may be blocking probes
- Try host discovery with -Pn
- Confirm connectivity and routing
```

### Too Many Alerts
```text
Fix:
- Reduce timing to -T2 or -T1
- Limit the port list
- Randomize hosts
```

### Incomplete Results
```text
Fix:
- Reduce scan delays
- Increase timeouts
- Verify packet loss or filtering
```

## Mini Glossary
- Firewall: Filters traffic based on rules
- IDS: Detects suspicious traffic
- IPS: Detects and blocks suspicious traffic
- Evasion: Attempts to reduce detection

## Step-by-Step Lab: Conservative Evasion
### Step 1: List Targets
```bash
nmap -sL 192.168.1.0/29
```
### Step 2: Host Discovery
```bash
nmap -sn 192.168.1.0/29 -oA outputs/lab-discovery
```
### Step 3: Stealth Scan Live Hosts
```bash
awk '/Up$/{print $2}' outputs/lab-discovery.gnmap > outputs/lab-live.txt
sudo nmap -sS -T2 -iL outputs/lab-live.txt -oA outputs/lab-evasion
```
### Step 4: Review Results
```bash
grep "/open/" outputs/lab-evasion.gnmap
```

## Large Network Strategy
- Run discovery first
- Scan only live hosts
- Use conservative timing
- Document all actions

## Example: Conservative Multi-Target Scan
```bash
sudo nmap -sS -T2 --max-rate 30 --randomize-hosts 10.0.0.0/24 -oA outputs/large-conservative
```

## Example: Focused Web Scan
```bash
sudo nmap -sS -T2 -p 80,443,8080,8443 --max-rate 40 10.0.0.0/24 -oA outputs/web-focus
```

## Advanced Options Summary (Do Not Use Blindly)
- `-f` for fragmentation
- `-D` for decoys
- `--source-port` for source port manipulation
- `-sI` for idle scan

## Extended Practice Section
Below are repeated drills to build comfort with conservative evasion approaches.
Use a lab environment and adjust IPs as needed.

### Drill 1: Conservative Evasion
Goal: Reduce detection while scanning authorized targets.
1. Choose a small range or target list.
2. Run discovery to find live hosts.
3. Use `-sS` with `-T2` and a short port list.
4. Add `--max-rate` to control traffic.
5. Save output with `-oA`.
6. Review `.gnmap` for open ports.
7. Document your options and results.

### Drill 2: Conservative Evasion
Goal: Reduce detection while scanning authorized targets.
1. Choose a small range or target list.
2. Run discovery to find live hosts.
3. Use `-sS` with `-T2` and a short port list.
4. Add `--max-rate` to control traffic.
5. Save output with `-oA`.
6. Review `.gnmap` for open ports.
7. Document your options and results.

### Drill 3: Conservative Evasion
Goal: Reduce detection while scanning authorized targets.
1. Choose a small range or target list.
2. Run discovery to find live hosts.
3. Use `-sS` with `-T2` and a short port list.
4. Add `--max-rate` to control traffic.
5. Save output with `-oA`.
6. Review `.gnmap` for open ports.
7. Document your options and results.

### Drill 4: Conservative Evasion
Goal: Reduce detection while scanning authorized targets.
1. Choose a small range or target list.
2. Run discovery to find live hosts.
3. Use `-sS` with `-T2` and a short port list.
4. Add `--max-rate` to control traffic.
5. Save output with `-oA`.
6. Review `.gnmap` for open ports.
7. Document your options and results.

### Drill 5: Conservative Evasion
Goal: Reduce detection while scanning authorized targets.
1. Choose a small range or target list.
2. Run discovery to find live hosts.
3. Use `-sS` with `-T2` and a short port list.
4. Add `--max-rate` to control traffic.
5. Save output with `-oA`.
6. Review `.gnmap` for open ports.
7. Document your options and results.

### Drill 6: Conservative Evasion
Goal: Reduce detection while scanning authorized targets.
1. Choose a small range or target list.
2. Run discovery to find live hosts.
3. Use `-sS` with `-T2` and a short port list.
4. Add `--max-rate` to control traffic.
5. Save output with `-oA`.
6. Review `.gnmap` for open ports.
7. Document your options and results.

### Drill 7: Conservative Evasion
Goal: Reduce detection while scanning authorized targets.
1. Choose a small range or target list.
2. Run discovery to find live hosts.
3. Use `-sS` with `-T2` and a short port list.
4. Add `--max-rate` to control traffic.
5. Save output with `-oA`.
6. Review `.gnmap` for open ports.
7. Document your options and results.

### Drill 8: Conservative Evasion
Goal: Reduce detection while scanning authorized targets.
1. Choose a small range or target list.
2. Run discovery to find live hosts.
3. Use `-sS` with `-T2` and a short port list.
4. Add `--max-rate` to control traffic.
5. Save output with `-oA`.
6. Review `.gnmap` for open ports.
7. Document your options and results.

### Drill 9: Conservative Evasion
Goal: Reduce detection while scanning authorized targets.
1. Choose a small range or target list.
2. Run discovery to find live hosts.
3. Use `-sS` with `-T2` and a short port list.
4. Add `--max-rate` to control traffic.
5. Save output with `-oA`.
6. Review `.gnmap` for open ports.
7. Document your options and results.

### Drill 10: Conservative Evasion
Goal: Reduce detection while scanning authorized targets.
1. Choose a small range or target list.
2. Run discovery to find live hosts.
3. Use `-sS` with `-T2` and a short port list.
4. Add `--max-rate` to control traffic.
5. Save output with `-oA`.
6. Review `.gnmap` for open ports.
7. Document your options and results.

### Drill 11: Conservative Evasion
Goal: Reduce detection while scanning authorized targets.
1. Choose a small range or target list.
2. Run discovery to find live hosts.
3. Use `-sS` with `-T2` and a short port list.
4. Add `--max-rate` to control traffic.
5. Save output with `-oA`.
6. Review `.gnmap` for open ports.
7. Document your options and results.

### Drill 12: Conservative Evasion
Goal: Reduce detection while scanning authorized targets.
1. Choose a small range or target list.
2. Run discovery to find live hosts.
3. Use `-sS` with `-T2` and a short port list.
4. Add `--max-rate` to control traffic.
5. Save output with `-oA`.
6. Review `.gnmap` for open ports.
7. Document your options and results.

### Drill 13: Conservative Evasion
Goal: Reduce detection while scanning authorized targets.
1. Choose a small range or target list.
2. Run discovery to find live hosts.
3. Use `-sS` with `-T2` and a short port list.
4. Add `--max-rate` to control traffic.
5. Save output with `-oA`.
6. Review `.gnmap` for open ports.
7. Document your options and results.

### Drill 14: Conservative Evasion
Goal: Reduce detection while scanning authorized targets.
1. Choose a small range or target list.
2. Run discovery to find live hosts.
3. Use `-sS` with `-T2` and a short port list.
4. Add `--max-rate` to control traffic.
5. Save output with `-oA`.
6. Review `.gnmap` for open ports.
7. Document your options and results.

### Drill 15: Conservative Evasion
Goal: Reduce detection while scanning authorized targets.
1. Choose a small range or target list.
2. Run discovery to find live hosts.
3. Use `-sS` with `-T2` and a short port list.
4. Add `--max-rate` to control traffic.
5. Save output with `-oA`.
6. Review `.gnmap` for open ports.
7. Document your options and results.

### Drill 16: Conservative Evasion
Goal: Reduce detection while scanning authorized targets.
1. Choose a small range or target list.
2. Run discovery to find live hosts.
3. Use `-sS` with `-T2` and a short port list.
4. Add `--max-rate` to control traffic.
5. Save output with `-oA`.
6. Review `.gnmap` for open ports.
7. Document your options and results.

### Drill 17: Conservative Evasion
Goal: Reduce detection while scanning authorized targets.
1. Choose a small range or target list.
2. Run discovery to find live hosts.
3. Use `-sS` with `-T2` and a short port list.
4. Add `--max-rate` to control traffic.
5. Save output with `-oA`.
6. Review `.gnmap` for open ports.
7. Document your options and results.

### Drill 18: Conservative Evasion
Goal: Reduce detection while scanning authorized targets.
1. Choose a small range or target list.
2. Run discovery to find live hosts.
3. Use `-sS` with `-T2` and a short port list.
4. Add `--max-rate` to control traffic.
5. Save output with `-oA`.
6. Review `.gnmap` for open ports.
7. Document your options and results.

### Drill 19: Conservative Evasion
Goal: Reduce detection while scanning authorized targets.
1. Choose a small range or target list.
2. Run discovery to find live hosts.
3. Use `-sS` with `-T2` and a short port list.
4. Add `--max-rate` to control traffic.
5. Save output with `-oA`.
6. Review `.gnmap` for open ports.
7. Document your options and results.

### Drill 20: Conservative Evasion
Goal: Reduce detection while scanning authorized targets.
1. Choose a small range or target list.
2. Run discovery to find live hosts.
3. Use `-sS` with `-T2` and a short port list.
4. Add `--max-rate` to control traffic.
5. Save output with `-oA`.
6. Review `.gnmap` for open ports.
7. Document your options and results.

### Drill 21: Conservative Evasion
Goal: Reduce detection while scanning authorized targets.
1. Choose a small range or target list.
2. Run discovery to find live hosts.
3. Use `-sS` with `-T2` and a short port list.
4. Add `--max-rate` to control traffic.
5. Save output with `-oA`.
6. Review `.gnmap` for open ports.
7. Document your options and results.

### Drill 22: Conservative Evasion
Goal: Reduce detection while scanning authorized targets.
1. Choose a small range or target list.
2. Run discovery to find live hosts.
3. Use `-sS` with `-T2` and a short port list.
4. Add `--max-rate` to control traffic.
5. Save output with `-oA`.
6. Review `.gnmap` for open ports.
7. Document your options and results.

### Drill 23: Conservative Evasion
Goal: Reduce detection while scanning authorized targets.
1. Choose a small range or target list.
2. Run discovery to find live hosts.
3. Use `-sS` with `-T2` and a short port list.
4. Add `--max-rate` to control traffic.
5. Save output with `-oA`.
6. Review `.gnmap` for open ports.
7. Document your options and results.

### Drill 24: Conservative Evasion
Goal: Reduce detection while scanning authorized targets.
1. Choose a small range or target list.
2. Run discovery to find live hosts.
3. Use `-sS` with `-T2` and a short port list.
4. Add `--max-rate` to control traffic.
5. Save output with `-oA`.
6. Review `.gnmap` for open ports.
7. Document your options and results.

### Drill 25: Conservative Evasion
Goal: Reduce detection while scanning authorized targets.
1. Choose a small range or target list.
2. Run discovery to find live hosts.
3. Use `-sS` with `-T2` and a short port list.
4. Add `--max-rate` to control traffic.
5. Save output with `-oA`.
6. Review `.gnmap` for open ports.
7. Document your options and results.

### Drill 26: Conservative Evasion
Goal: Reduce detection while scanning authorized targets.
1. Choose a small range or target list.
2. Run discovery to find live hosts.
3. Use `-sS` with `-T2` and a short port list.
4. Add `--max-rate` to control traffic.
5. Save output with `-oA`.
6. Review `.gnmap` for open ports.
7. Document your options and results.

### Drill 27: Conservative Evasion
Goal: Reduce detection while scanning authorized targets.
1. Choose a small range or target list.
2. Run discovery to find live hosts.
3. Use `-sS` with `-T2` and a short port list.
4. Add `--max-rate` to control traffic.
5. Save output with `-oA`.
6. Review `.gnmap` for open ports.
7. Document your options and results.

### Drill 28: Conservative Evasion
Goal: Reduce detection while scanning authorized targets.
1. Choose a small range or target list.
2. Run discovery to find live hosts.
3. Use `-sS` with `-T2` and a short port list.
4. Add `--max-rate` to control traffic.
5. Save output with `-oA`.
6. Review `.gnmap` for open ports.
7. Document your options and results.

### Drill 29: Conservative Evasion
Goal: Reduce detection while scanning authorized targets.
1. Choose a small range or target list.
2. Run discovery to find live hosts.
3. Use `-sS` with `-T2` and a short port list.
4. Add `--max-rate` to control traffic.
5. Save output with `-oA`.
6. Review `.gnmap` for open ports.
7. Document your options and results.

### Drill 30: Conservative Evasion
Goal: Reduce detection while scanning authorized targets.
1. Choose a small range or target list.
2. Run discovery to find live hosts.
3. Use `-sS` with `-T2` and a short port list.
4. Add `--max-rate` to control traffic.
5. Save output with `-oA`.
6. Review `.gnmap` for open ports.
7. Document your options and results.

### Drill 31: Conservative Evasion
Goal: Reduce detection while scanning authorized targets.
1. Choose a small range or target list.
2. Run discovery to find live hosts.
3. Use `-sS` with `-T2` and a short port list.
4. Add `--max-rate` to control traffic.
5. Save output with `-oA`.
6. Review `.gnmap` for open ports.
7. Document your options and results.

### Drill 32: Conservative Evasion
Goal: Reduce detection while scanning authorized targets.
1. Choose a small range or target list.
2. Run discovery to find live hosts.
3. Use `-sS` with `-T2` and a short port list.
4. Add `--max-rate` to control traffic.
5. Save output with `-oA`.
6. Review `.gnmap` for open ports.
7. Document your options and results.

### Drill 33: Conservative Evasion
Goal: Reduce detection while scanning authorized targets.
1. Choose a small range or target list.
2. Run discovery to find live hosts.
3. Use `-sS` with `-T2` and a short port list.
4. Add `--max-rate` to control traffic.
5. Save output with `-oA`.
6. Review `.gnmap` for open ports.
7. Document your options and results.

### Drill 34: Conservative Evasion
Goal: Reduce detection while scanning authorized targets.
1. Choose a small range or target list.
2. Run discovery to find live hosts.
3. Use `-sS` with `-T2` and a short port list.
4. Add `--max-rate` to control traffic.
5. Save output with `-oA`.
6. Review `.gnmap` for open ports.
7. Document your options and results.

### Drill 35: Conservative Evasion
Goal: Reduce detection while scanning authorized targets.
1. Choose a small range or target list.
2. Run discovery to find live hosts.
3. Use `-sS` with `-T2` and a short port list.
4. Add `--max-rate` to control traffic.
5. Save output with `-oA`.
6. Review `.gnmap` for open ports.
7. Document your options and results.

### Drill 36: Conservative Evasion
Goal: Reduce detection while scanning authorized targets.
1. Choose a small range or target list.
2. Run discovery to find live hosts.
3. Use `-sS` with `-T2` and a short port list.
4. Add `--max-rate` to control traffic.
5. Save output with `-oA`.
6. Review `.gnmap` for open ports.
7. Document your options and results.

### Drill 37: Conservative Evasion
Goal: Reduce detection while scanning authorized targets.
1. Choose a small range or target list.
2. Run discovery to find live hosts.
3. Use `-sS` with `-T2` and a short port list.
4. Add `--max-rate` to control traffic.
5. Save output with `-oA`.
6. Review `.gnmap` for open ports.
7. Document your options and results.

### Drill 38: Conservative Evasion
Goal: Reduce detection while scanning authorized targets.
1. Choose a small range or target list.
2. Run discovery to find live hosts.
3. Use `-sS` with `-T2` and a short port list.
4. Add `--max-rate` to control traffic.
5. Save output with `-oA`.
6. Review `.gnmap` for open ports.
7. Document your options and results.

### Drill 39: Conservative Evasion
Goal: Reduce detection while scanning authorized targets.
1. Choose a small range or target list.
2. Run discovery to find live hosts.
3. Use `-sS` with `-T2` and a short port list.
4. Add `--max-rate` to control traffic.
5. Save output with `-oA`.
6. Review `.gnmap` for open ports.
7. Document your options and results.

### Drill 40: Conservative Evasion
Goal: Reduce detection while scanning authorized targets.
1. Choose a small range or target list.
2. Run discovery to find live hosts.
3. Use `-sS` with `-T2` and a short port list.
4. Add `--max-rate` to control traffic.
5. Save output with `-oA`.
6. Review `.gnmap` for open ports.
7. Document your options and results.

### Drill 41: Conservative Evasion
Goal: Reduce detection while scanning authorized targets.
1. Choose a small range or target list.
2. Run discovery to find live hosts.
3. Use `-sS` with `-T2` and a short port list.
4. Add `--max-rate` to control traffic.
5. Save output with `-oA`.
6. Review `.gnmap` for open ports.
7. Document your options and results.

### Drill 42: Conservative Evasion
Goal: Reduce detection while scanning authorized targets.
1. Choose a small range or target list.
2. Run discovery to find live hosts.
3. Use `-sS` with `-T2` and a short port list.
4. Add `--max-rate` to control traffic.
5. Save output with `-oA`.
6. Review `.gnmap` for open ports.
7. Document your options and results.

### Drill 43: Conservative Evasion
Goal: Reduce detection while scanning authorized targets.
1. Choose a small range or target list.
2. Run discovery to find live hosts.
3. Use `-sS` with `-T2` and a short port list.
4. Add `--max-rate` to control traffic.
5. Save output with `-oA`.
6. Review `.gnmap` for open ports.
7. Document your options and results.

### Drill 44: Conservative Evasion
Goal: Reduce detection while scanning authorized targets.
1. Choose a small range or target list.
2. Run discovery to find live hosts.
3. Use `-sS` with `-T2` and a short port list.
4. Add `--max-rate` to control traffic.
5. Save output with `-oA`.
6. Review `.gnmap` for open ports.
7. Document your options and results.

### Drill 45: Conservative Evasion
Goal: Reduce detection while scanning authorized targets.
1. Choose a small range or target list.
2. Run discovery to find live hosts.
3. Use `-sS` with `-T2` and a short port list.
4. Add `--max-rate` to control traffic.
5. Save output with `-oA`.
6. Review `.gnmap` for open ports.
7. Document your options and results.

### Drill 46: Conservative Evasion
Goal: Reduce detection while scanning authorized targets.
1. Choose a small range or target list.
2. Run discovery to find live hosts.
3. Use `-sS` with `-T2` and a short port list.
4. Add `--max-rate` to control traffic.
5. Save output with `-oA`.
6. Review `.gnmap` for open ports.
7. Document your options and results.

### Drill 47: Conservative Evasion
Goal: Reduce detection while scanning authorized targets.
1. Choose a small range or target list.
2. Run discovery to find live hosts.
3. Use `-sS` with `-T2` and a short port list.
4. Add `--max-rate` to control traffic.
5. Save output with `-oA`.
6. Review `.gnmap` for open ports.
7. Document your options and results.

### Drill 48: Conservative Evasion
Goal: Reduce detection while scanning authorized targets.
1. Choose a small range or target list.
2. Run discovery to find live hosts.
3. Use `-sS` with `-T2` and a short port list.
4. Add `--max-rate` to control traffic.
5. Save output with `-oA`.
6. Review `.gnmap` for open ports.
7. Document your options and results.

### Drill 49: Conservative Evasion
Goal: Reduce detection while scanning authorized targets.
1. Choose a small range or target list.
2. Run discovery to find live hosts.
3. Use `-sS` with `-T2` and a short port list.
4. Add `--max-rate` to control traffic.
5. Save output with `-oA`.
6. Review `.gnmap` for open ports.
7. Document your options and results.

### Drill 50: Conservative Evasion
Goal: Reduce detection while scanning authorized targets.
1. Choose a small range or target list.
2. Run discovery to find live hosts.
3. Use `-sS` with `-T2` and a short port list.
4. Add `--max-rate` to control traffic.
5. Save output with `-oA`.
6. Review `.gnmap` for open ports.
7. Document your options and results.

### Drill 51: Conservative Evasion
Goal: Reduce detection while scanning authorized targets.
1. Choose a small range or target list.
2. Run discovery to find live hosts.
3. Use `-sS` with `-T2` and a short port list.
4. Add `--max-rate` to control traffic.
5. Save output with `-oA`.
6. Review `.gnmap` for open ports.
7. Document your options and results.

### Drill 52: Conservative Evasion
Goal: Reduce detection while scanning authorized targets.
1. Choose a small range or target list.
2. Run discovery to find live hosts.
3. Use `-sS` with `-T2` and a short port list.
4. Add `--max-rate` to control traffic.
5. Save output with `-oA`.
6. Review `.gnmap` for open ports.
7. Document your options and results.

### Drill 53: Conservative Evasion
Goal: Reduce detection while scanning authorized targets.
1. Choose a small range or target list.
2. Run discovery to find live hosts.
3. Use `-sS` with `-T2` and a short port list.
4. Add `--max-rate` to control traffic.
5. Save output with `-oA`.
6. Review `.gnmap` for open ports.
7. Document your options and results.

### Drill 54: Conservative Evasion
Goal: Reduce detection while scanning authorized targets.
1. Choose a small range or target list.
2. Run discovery to find live hosts.
3. Use `-sS` with `-T2` and a short port list.
4. Add `--max-rate` to control traffic.
5. Save output with `-oA`.
6. Review `.gnmap` for open ports.
7. Document your options and results.

### Drill 55: Conservative Evasion
Goal: Reduce detection while scanning authorized targets.
1. Choose a small range or target list.
2. Run discovery to find live hosts.
3. Use `-sS` with `-T2` and a short port list.
4. Add `--max-rate` to control traffic.
5. Save output with `-oA`.
6. Review `.gnmap` for open ports.
7. Document your options and results.

### Drill 56: Conservative Evasion
Goal: Reduce detection while scanning authorized targets.
1. Choose a small range or target list.
2. Run discovery to find live hosts.
3. Use `-sS` with `-T2` and a short port list.
4. Add `--max-rate` to control traffic.
5. Save output with `-oA`.
6. Review `.gnmap` for open ports.
7. Document your options and results.

### Drill 57: Conservative Evasion
Goal: Reduce detection while scanning authorized targets.
1. Choose a small range or target list.
2. Run discovery to find live hosts.
3. Use `-sS` with `-T2` and a short port list.
4. Add `--max-rate` to control traffic.
5. Save output with `-oA`.
6. Review `.gnmap` for open ports.
7. Document your options and results.

### Drill 58: Conservative Evasion
Goal: Reduce detection while scanning authorized targets.
1. Choose a small range or target list.
2. Run discovery to find live hosts.
3. Use `-sS` with `-T2` and a short port list.
4. Add `--max-rate` to control traffic.
5. Save output with `-oA`.
6. Review `.gnmap` for open ports.
7. Document your options and results.

### Drill 59: Conservative Evasion
Goal: Reduce detection while scanning authorized targets.
1. Choose a small range or target list.
2. Run discovery to find live hosts.
3. Use `-sS` with `-T2` and a short port list.
4. Add `--max-rate` to control traffic.
5. Save output with `-oA`.
6. Review `.gnmap` for open ports.
7. Document your options and results.

### Drill 60: Conservative Evasion
Goal: Reduce detection while scanning authorized targets.
1. Choose a small range or target list.
2. Run discovery to find live hosts.
3. Use `-sS` with `-T2` and a short port list.
4. Add `--max-rate` to control traffic.
5. Save output with `-oA`.
6. Review `.gnmap` for open ports.
7. Document your options and results.

### Drill 61: Conservative Evasion
Goal: Reduce detection while scanning authorized targets.
1. Choose a small range or target list.
2. Run discovery to find live hosts.
3. Use `-sS` with `-T2` and a short port list.
4. Add `--max-rate` to control traffic.
5. Save output with `-oA`.
6. Review `.gnmap` for open ports.
7. Document your options and results.

### Drill 62: Conservative Evasion
Goal: Reduce detection while scanning authorized targets.
1. Choose a small range or target list.
2. Run discovery to find live hosts.
3. Use `-sS` with `-T2` and a short port list.
4. Add `--max-rate` to control traffic.
5. Save output with `-oA`.
6. Review `.gnmap` for open ports.
7. Document your options and results.

### Drill 63: Conservative Evasion
Goal: Reduce detection while scanning authorized targets.
1. Choose a small range or target list.
2. Run discovery to find live hosts.
3. Use `-sS` with `-T2` and a short port list.
4. Add `--max-rate` to control traffic.
5. Save output with `-oA`.
6. Review `.gnmap` for open ports.
7. Document your options and results.

### Drill 64: Conservative Evasion
Goal: Reduce detection while scanning authorized targets.
1. Choose a small range or target list.
2. Run discovery to find live hosts.
3. Use `-sS` with `-T2` and a short port list.
4. Add `--max-rate` to control traffic.
5. Save output with `-oA`.
6. Review `.gnmap` for open ports.
7. Document your options and results.

### Drill 65: Conservative Evasion
Goal: Reduce detection while scanning authorized targets.
1. Choose a small range or target list.
2. Run discovery to find live hosts.
3. Use `-sS` with `-T2` and a short port list.
4. Add `--max-rate` to control traffic.
5. Save output with `-oA`.
6. Review `.gnmap` for open ports.
7. Document your options and results.

### Drill 66: Conservative Evasion
Goal: Reduce detection while scanning authorized targets.
1. Choose a small range or target list.
2. Run discovery to find live hosts.
3. Use `-sS` with `-T2` and a short port list.
4. Add `--max-rate` to control traffic.
5. Save output with `-oA`.
6. Review `.gnmap` for open ports.
7. Document your options and results.

### Drill 67: Conservative Evasion
Goal: Reduce detection while scanning authorized targets.
1. Choose a small range or target list.
2. Run discovery to find live hosts.
3. Use `-sS` with `-T2` and a short port list.
4. Add `--max-rate` to control traffic.
5. Save output with `-oA`.
6. Review `.gnmap` for open ports.
7. Document your options and results.

### Drill 68: Conservative Evasion
Goal: Reduce detection while scanning authorized targets.
1. Choose a small range or target list.
2. Run discovery to find live hosts.
3. Use `-sS` with `-T2` and a short port list.
4. Add `--max-rate` to control traffic.
5. Save output with `-oA`.
6. Review `.gnmap` for open ports.
7. Document your options and results.

### Drill 69: Conservative Evasion
Goal: Reduce detection while scanning authorized targets.
1. Choose a small range or target list.
2. Run discovery to find live hosts.
3. Use `-sS` with `-T2` and a short port list.
4. Add `--max-rate` to control traffic.
5. Save output with `-oA`.
6. Review `.gnmap` for open ports.
7. Document your options and results.

### Drill 70: Conservative Evasion
Goal: Reduce detection while scanning authorized targets.
1. Choose a small range or target list.
2. Run discovery to find live hosts.
3. Use `-sS` with `-T2` and a short port list.
4. Add `--max-rate` to control traffic.
5. Save output with `-oA`.
6. Review `.gnmap` for open ports.
7. Document your options and results.

### Drill 71: Conservative Evasion
Goal: Reduce detection while scanning authorized targets.
1. Choose a small range or target list.
2. Run discovery to find live hosts.
3. Use `-sS` with `-T2` and a short port list.
4. Add `--max-rate` to control traffic.
5. Save output with `-oA`.
6. Review `.gnmap` for open ports.
7. Document your options and results.

### Drill 72: Conservative Evasion
Goal: Reduce detection while scanning authorized targets.
1. Choose a small range or target list.
2. Run discovery to find live hosts.
3. Use `-sS` with `-T2` and a short port list.
4. Add `--max-rate` to control traffic.
5. Save output with `-oA`.
6. Review `.gnmap` for open ports.
7. Document your options and results.

### Drill 73: Conservative Evasion
Goal: Reduce detection while scanning authorized targets.
1. Choose a small range or target list.
2. Run discovery to find live hosts.
3. Use `-sS` with `-T2` and a short port list.
4. Add `--max-rate` to control traffic.
5. Save output with `-oA`.
6. Review `.gnmap` for open ports.
7. Document your options and results.

### Drill 74: Conservative Evasion
Goal: Reduce detection while scanning authorized targets.
1. Choose a small range or target list.
2. Run discovery to find live hosts.
3. Use `-sS` with `-T2` and a short port list.
4. Add `--max-rate` to control traffic.
5. Save output with `-oA`.
6. Review `.gnmap` for open ports.
7. Document your options and results.

### Drill 75: Conservative Evasion
Goal: Reduce detection while scanning authorized targets.
1. Choose a small range or target list.
2. Run discovery to find live hosts.
3. Use `-sS` with `-T2` and a short port list.
4. Add `--max-rate` to control traffic.
5. Save output with `-oA`.
6. Review `.gnmap` for open ports.
7. Document your options and results.

### Drill 76: Conservative Evasion
Goal: Reduce detection while scanning authorized targets.
1. Choose a small range or target list.
2. Run discovery to find live hosts.
3. Use `-sS` with `-T2` and a short port list.
4. Add `--max-rate` to control traffic.
5. Save output with `-oA`.
6. Review `.gnmap` for open ports.
7. Document your options and results.

### Drill 77: Conservative Evasion
Goal: Reduce detection while scanning authorized targets.
1. Choose a small range or target list.
2. Run discovery to find live hosts.
3. Use `-sS` with `-T2` and a short port list.
4. Add `--max-rate` to control traffic.
5. Save output with `-oA`.
6. Review `.gnmap` for open ports.
7. Document your options and results.

### Drill 78: Conservative Evasion
Goal: Reduce detection while scanning authorized targets.
1. Choose a small range or target list.
2. Run discovery to find live hosts.
3. Use `-sS` with `-T2` and a short port list.
4. Add `--max-rate` to control traffic.
5. Save output with `-oA`.
6. Review `.gnmap` for open ports.
7. Document your options and results.

### Drill 79: Conservative Evasion
Goal: Reduce detection while scanning authorized targets.
1. Choose a small range or target list.
2. Run discovery to find live hosts.
3. Use `-sS` with `-T2` and a short port list.
4. Add `--max-rate` to control traffic.
5. Save output with `-oA`.
6. Review `.gnmap` for open ports.
7. Document your options and results.

### Drill 80: Conservative Evasion
Goal: Reduce detection while scanning authorized targets.
1. Choose a small range or target list.
2. Run discovery to find live hosts.
3. Use `-sS` with `-T2` and a short port list.
4. Add `--max-rate` to control traffic.
5. Save output with `-oA`.
6. Review `.gnmap` for open ports.
7. Document your options and results.

### Drill 81: Conservative Evasion
Goal: Reduce detection while scanning authorized targets.
1. Choose a small range or target list.
2. Run discovery to find live hosts.
3. Use `-sS` with `-T2` and a short port list.
4. Add `--max-rate` to control traffic.
5. Save output with `-oA`.
6. Review `.gnmap` for open ports.
7. Document your options and results.

### Drill 82: Conservative Evasion
Goal: Reduce detection while scanning authorized targets.
1. Choose a small range or target list.
2. Run discovery to find live hosts.
3. Use `-sS` with `-T2` and a short port list.
4. Add `--max-rate` to control traffic.
5. Save output with `-oA`.
6. Review `.gnmap` for open ports.
7. Document your options and results.

### Drill 83: Conservative Evasion
Goal: Reduce detection while scanning authorized targets.
1. Choose a small range or target list.
2. Run discovery to find live hosts.
3. Use `-sS` with `-T2` and a short port list.
4. Add `--max-rate` to control traffic.
5. Save output with `-oA`.
6. Review `.gnmap` for open ports.
7. Document your options and results.

### Drill 84: Conservative Evasion
Goal: Reduce detection while scanning authorized targets.
1. Choose a small range or target list.
2. Run discovery to find live hosts.
3. Use `-sS` with `-T2` and a short port list.
4. Add `--max-rate` to control traffic.
5. Save output with `-oA`.
6. Review `.gnmap` for open ports.
7. Document your options and results.

### Drill 85: Conservative Evasion
Goal: Reduce detection while scanning authorized targets.
1. Choose a small range or target list.
2. Run discovery to find live hosts.
3. Use `-sS` with `-T2` and a short port list.
4. Add `--max-rate` to control traffic.
5. Save output with `-oA`.
6. Review `.gnmap` for open ports.
7. Document your options and results.

### Drill 86: Conservative Evasion
Goal: Reduce detection while scanning authorized targets.
1. Choose a small range or target list.
2. Run discovery to find live hosts.
3. Use `-sS` with `-T2` and a short port list.
4. Add `--max-rate` to control traffic.
5. Save output with `-oA`.
6. Review `.gnmap` for open ports.
7. Document your options and results.

### Drill 87: Conservative Evasion
Goal: Reduce detection while scanning authorized targets.
1. Choose a small range or target list.
2. Run discovery to find live hosts.
3. Use `-sS` with `-T2` and a short port list.
4. Add `--max-rate` to control traffic.
5. Save output with `-oA`.
6. Review `.gnmap` for open ports.
7. Document your options and results.

### Drill 88: Conservative Evasion
Goal: Reduce detection while scanning authorized targets.
1. Choose a small range or target list.
2. Run discovery to find live hosts.
3. Use `-sS` with `-T2` and a short port list.
4. Add `--max-rate` to control traffic.
5. Save output with `-oA`.
6. Review `.gnmap` for open ports.
7. Document your options and results.

### Drill 89: Conservative Evasion
Goal: Reduce detection while scanning authorized targets.
1. Choose a small range or target list.
2. Run discovery to find live hosts.
3. Use `-sS` with `-T2` and a short port list.
4. Add `--max-rate` to control traffic.
5. Save output with `-oA`.
6. Review `.gnmap` for open ports.
7. Document your options and results.

### Drill 90: Conservative Evasion
Goal: Reduce detection while scanning authorized targets.
1. Choose a small range or target list.
2. Run discovery to find live hosts.
3. Use `-sS` with `-T2` and a short port list.
4. Add `--max-rate` to control traffic.
5. Save output with `-oA`.
6. Review `.gnmap` for open ports.
7. Document your options and results.

### Drill 91: Conservative Evasion
Goal: Reduce detection while scanning authorized targets.
1. Choose a small range or target list.
2. Run discovery to find live hosts.
3. Use `-sS` with `-T2` and a short port list.
4. Add `--max-rate` to control traffic.
5. Save output with `-oA`.
6. Review `.gnmap` for open ports.
7. Document your options and results.

### Drill 92: Conservative Evasion
Goal: Reduce detection while scanning authorized targets.
1. Choose a small range or target list.
2. Run discovery to find live hosts.
3. Use `-sS` with `-T2` and a short port list.
4. Add `--max-rate` to control traffic.
5. Save output with `-oA`.
6. Review `.gnmap` for open ports.
7. Document your options and results.

### Drill 93: Conservative Evasion
Goal: Reduce detection while scanning authorized targets.
1. Choose a small range or target list.
2. Run discovery to find live hosts.
3. Use `-sS` with `-T2` and a short port list.
4. Add `--max-rate` to control traffic.
5. Save output with `-oA`.
6. Review `.gnmap` for open ports.
7. Document your options and results.

### Drill 94: Conservative Evasion
Goal: Reduce detection while scanning authorized targets.
1. Choose a small range or target list.
2. Run discovery to find live hosts.
3. Use `-sS` with `-T2` and a short port list.
4. Add `--max-rate` to control traffic.
5. Save output with `-oA`.
6. Review `.gnmap` for open ports.
7. Document your options and results.

### Drill 95: Conservative Evasion
Goal: Reduce detection while scanning authorized targets.
1. Choose a small range or target list.
2. Run discovery to find live hosts.
3. Use `-sS` with `-T2` and a short port list.
4. Add `--max-rate` to control traffic.
5. Save output with `-oA`.
6. Review `.gnmap` for open ports.
7. Document your options and results.

### Drill 96: Conservative Evasion
Goal: Reduce detection while scanning authorized targets.
1. Choose a small range or target list.
2. Run discovery to find live hosts.
3. Use `-sS` with `-T2` and a short port list.
4. Add `--max-rate` to control traffic.
5. Save output with `-oA`.
6. Review `.gnmap` for open ports.
7. Document your options and results.

### Drill 97: Conservative Evasion
Goal: Reduce detection while scanning authorized targets.
1. Choose a small range or target list.
2. Run discovery to find live hosts.
3. Use `-sS` with `-T2` and a short port list.
4. Add `--max-rate` to control traffic.
5. Save output with `-oA`.
6. Review `.gnmap` for open ports.
7. Document your options and results.

### Drill 98: Conservative Evasion
Goal: Reduce detection while scanning authorized targets.
1. Choose a small range or target list.
2. Run discovery to find live hosts.
3. Use `-sS` with `-T2` and a short port list.
4. Add `--max-rate` to control traffic.
5. Save output with `-oA`.
6. Review `.gnmap` for open ports.
7. Document your options and results.

### Drill 99: Conservative Evasion
Goal: Reduce detection while scanning authorized targets.
1. Choose a small range or target list.
2. Run discovery to find live hosts.
3. Use `-sS` with `-T2` and a short port list.
4. Add `--max-rate` to control traffic.
5. Save output with `-oA`.
6. Review `.gnmap` for open ports.
7. Document your options and results.

### Drill 100: Conservative Evasion
Goal: Reduce detection while scanning authorized targets.
1. Choose a small range or target list.
2. Run discovery to find live hosts.
3. Use `-sS` with `-T2` and a short port list.
4. Add `--max-rate` to control traffic.
5. Save output with `-oA`.
6. Review `.gnmap` for open ports.
7. Document your options and results.

```powershell
## Command Reference (One-Liners)
- Example 1: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.1.0/24 -oA outputs/evasion-1`
- Example 2: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.2.0/24 -oA outputs/evasion-2`
- Example 3: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.3.0/24 -oA outputs/evasion-3`
- Example 4: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.4.0/24 -oA outputs/evasion-4`
- Example 5: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.5.0/24 -oA outputs/evasion-5`
- Example 6: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.6.0/24 -oA outputs/evasion-6`
- Example 7: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.7.0/24 -oA outputs/evasion-7`
- Example 8: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.8.0/24 -oA outputs/evasion-8`
- Example 9: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.9.0/24 -oA outputs/evasion-9`
- Example 10: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.0.0/24 -oA outputs/evasion-10`
- Example 11: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.1.0/24 -oA outputs/evasion-11`
- Example 12: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.2.0/24 -oA outputs/evasion-12`
- Example 13: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.3.0/24 -oA outputs/evasion-13`
- Example 14: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.4.0/24 -oA outputs/evasion-14`
- Example 15: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.5.0/24 -oA outputs/evasion-15`
- Example 16: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.6.0/24 -oA outputs/evasion-16`
- Example 17: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.7.0/24 -oA outputs/evasion-17`
- Example 18: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.8.0/24 -oA outputs/evasion-18`
- Example 19: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.9.0/24 -oA outputs/evasion-19`
- Example 20: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.0.0/24 -oA outputs/evasion-20`
- Example 21: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.1.0/24 -oA outputs/evasion-21`
- Example 22: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.2.0/24 -oA outputs/evasion-22`
- Example 23: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.3.0/24 -oA outputs/evasion-23`
- Example 24: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.4.0/24 -oA outputs/evasion-24`
- Example 25: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.5.0/24 -oA outputs/evasion-25`
- Example 26: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.6.0/24 -oA outputs/evasion-26`
- Example 27: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.7.0/24 -oA outputs/evasion-27`
- Example 28: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.8.0/24 -oA outputs/evasion-28`
- Example 29: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.9.0/24 -oA outputs/evasion-29`
- Example 30: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.0.0/24 -oA outputs/evasion-30`
- Example 31: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.1.0/24 -oA outputs/evasion-31`
- Example 32: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.2.0/24 -oA outputs/evasion-32`
- Example 33: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.3.0/24 -oA outputs/evasion-33`
- Example 34: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.4.0/24 -oA outputs/evasion-34`
- Example 35: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.5.0/24 -oA outputs/evasion-35`
- Example 36: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.6.0/24 -oA outputs/evasion-36`
- Example 37: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.7.0/24 -oA outputs/evasion-37`
- Example 38: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.8.0/24 -oA outputs/evasion-38`
- Example 39: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.9.0/24 -oA outputs/evasion-39`
- Example 40: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.0.0/24 -oA outputs/evasion-40`
- Example 41: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.1.0/24 -oA outputs/evasion-41`
- Example 42: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.2.0/24 -oA outputs/evasion-42`
- Example 43: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.3.0/24 -oA outputs/evasion-43`
- Example 44: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.4.0/24 -oA outputs/evasion-44`
- Example 45: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.5.0/24 -oA outputs/evasion-45`
- Example 46: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.6.0/24 -oA outputs/evasion-46`
- Example 47: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.7.0/24 -oA outputs/evasion-47`
- Example 48: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.8.0/24 -oA outputs/evasion-48`
- Example 49: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.9.0/24 -oA outputs/evasion-49`
- Example 50: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.0.0/24 -oA outputs/evasion-50`
- Example 51: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.1.0/24 -oA outputs/evasion-51`
- Example 52: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.2.0/24 -oA outputs/evasion-52`
- Example 53: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.3.0/24 -oA outputs/evasion-53`
- Example 54: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.4.0/24 -oA outputs/evasion-54`
- Example 55: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.5.0/24 -oA outputs/evasion-55`
- Example 56: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.6.0/24 -oA outputs/evasion-56`
- Example 57: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.7.0/24 -oA outputs/evasion-57`
- Example 58: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.8.0/24 -oA outputs/evasion-58`
- Example 59: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.9.0/24 -oA outputs/evasion-59`
- Example 60: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.0.0/24 -oA outputs/evasion-60`
- Example 61: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.1.0/24 -oA outputs/evasion-61`
- Example 62: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.2.0/24 -oA outputs/evasion-62`
- Example 63: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.3.0/24 -oA outputs/evasion-63`
- Example 64: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.4.0/24 -oA outputs/evasion-64`
- Example 65: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.5.0/24 -oA outputs/evasion-65`
- Example 66: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.6.0/24 -oA outputs/evasion-66`
- Example 67: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.7.0/24 -oA outputs/evasion-67`
- Example 68: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.8.0/24 -oA outputs/evasion-68`
- Example 69: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.9.0/24 -oA outputs/evasion-69`
- Example 70: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.0.0/24 -oA outputs/evasion-70`
- Example 71: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.1.0/24 -oA outputs/evasion-71`
- Example 72: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.2.0/24 -oA outputs/evasion-72`
- Example 73: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.3.0/24 -oA outputs/evasion-73`
- Example 74: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.4.0/24 -oA outputs/evasion-74`
- Example 75: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.5.0/24 -oA outputs/evasion-75`
- Example 76: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.6.0/24 -oA outputs/evasion-76`
- Example 77: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.7.0/24 -oA outputs/evasion-77`
- Example 78: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.8.0/24 -oA outputs/evasion-78`
- Example 79: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.9.0/24 -oA outputs/evasion-79`
- Example 80: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.0.0/24 -oA outputs/evasion-80`
- Example 81: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.1.0/24 -oA outputs/evasion-81`
- Example 82: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.2.0/24 -oA outputs/evasion-82`
- Example 83: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.3.0/24 -oA outputs/evasion-83`
- Example 84: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.4.0/24 -oA outputs/evasion-84`
- Example 85: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.5.0/24 -oA outputs/evasion-85`
- Example 86: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.6.0/24 -oA outputs/evasion-86`
- Example 87: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.7.0/24 -oA outputs/evasion-87`
- Example 88: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.8.0/24 -oA outputs/evasion-88`
- Example 89: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.9.0/24 -oA outputs/evasion-89`
- Example 90: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.0.0/24 -oA outputs/evasion-90`
- Example 91: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.1.0/24 -oA outputs/evasion-91`
- Example 92: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.2.0/24 -oA outputs/evasion-92`
- Example 93: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.3.0/24 -oA outputs/evasion-93`
- Example 94: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.4.0/24 -oA outputs/evasion-94`
- Example 95: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.5.0/24 -oA outputs/evasion-95`
- Example 96: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.6.0/24 -oA outputs/evasion-96`
- Example 97: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.7.0/24 -oA outputs/evasion-97`
- Example 98: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.8.0/24 -oA outputs/evasion-98`
- Example 99: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.9.0/24 -oA outputs/evasion-99`
- Example 100: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.0.0/24 -oA outputs/evasion-100`
- Example 101: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.1.0/24 -oA outputs/evasion-101`
- Example 102: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.2.0/24 -oA outputs/evasion-102`
- Example 103: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.3.0/24 -oA outputs/evasion-103`
- Example 104: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.4.0/24 -oA outputs/evasion-104`
- Example 105: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.5.0/24 -oA outputs/evasion-105`
- Example 106: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.6.0/24 -oA outputs/evasion-106`
- Example 107: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.7.0/24 -oA outputs/evasion-107`
- Example 108: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.8.0/24 -oA outputs/evasion-108`
- Example 109: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.9.0/24 -oA outputs/evasion-109`
- Example 110: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.0.0/24 -oA outputs/evasion-110`
- Example 111: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.1.0/24 -oA outputs/evasion-111`
- Example 112: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.2.0/24 -oA outputs/evasion-112`
- Example 113: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.3.0/24 -oA outputs/evasion-113`
- Example 114: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.4.0/24 -oA outputs/evasion-114`
- Example 115: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.5.0/24 -oA outputs/evasion-115`
- Example 116: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.6.0/24 -oA outputs/evasion-116`
- Example 117: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.7.0/24 -oA outputs/evasion-117`
- Example 118: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.8.0/24 -oA outputs/evasion-118`
- Example 119: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.9.0/24 -oA outputs/evasion-119`
- Example 120: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.0.0/24 -oA outputs/evasion-120`
- Example 121: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.1.0/24 -oA outputs/evasion-121`
- Example 122: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.2.0/24 -oA outputs/evasion-122`
- Example 123: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.3.0/24 -oA outputs/evasion-123`
- Example 124: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.4.0/24 -oA outputs/evasion-124`
- Example 125: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.5.0/24 -oA outputs/evasion-125`
- Example 126: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.6.0/24 -oA outputs/evasion-126`
- Example 127: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.7.0/24 -oA outputs/evasion-127`
- Example 128: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.8.0/24 -oA outputs/evasion-128`
- Example 129: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.9.0/24 -oA outputs/evasion-129`
- Example 130: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.0.0/24 -oA outputs/evasion-130`
- Example 131: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.1.0/24 -oA outputs/evasion-131`
- Example 132: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.2.0/24 -oA outputs/evasion-132`
- Example 133: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.3.0/24 -oA outputs/evasion-133`
- Example 134: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.4.0/24 -oA outputs/evasion-134`
- Example 135: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.5.0/24 -oA outputs/evasion-135`
- Example 136: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.6.0/24 -oA outputs/evasion-136`
- Example 137: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.7.0/24 -oA outputs/evasion-137`
- Example 138: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.8.0/24 -oA outputs/evasion-138`
- Example 139: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.9.0/24 -oA outputs/evasion-139`
- Example 140: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.0.0/24 -oA outputs/evasion-140`
- Example 141: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.1.0/24 -oA outputs/evasion-141`
- Example 142: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.2.0/24 -oA outputs/evasion-142`
- Example 143: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.3.0/24 -oA outputs/evasion-143`
- Example 144: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.4.0/24 -oA outputs/evasion-144`
- Example 145: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.5.0/24 -oA outputs/evasion-145`
- Example 146: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.6.0/24 -oA outputs/evasion-146`
- Example 147: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.7.0/24 -oA outputs/evasion-147`
- Example 148: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.8.0/24 -oA outputs/evasion-148`
- Example 149: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.9.0/24 -oA outputs/evasion-149`
- Example 150: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.0.0/24 -oA outputs/evasion-150`
- Example 151: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.1.0/24 -oA outputs/evasion-151`
- Example 152: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.2.0/24 -oA outputs/evasion-152`
- Example 153: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.3.0/24 -oA outputs/evasion-153`
- Example 154: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.4.0/24 -oA outputs/evasion-154`
- Example 155: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.5.0/24 -oA outputs/evasion-155`
- Example 156: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.6.0/24 -oA outputs/evasion-156`
- Example 157: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.7.0/24 -oA outputs/evasion-157`
- Example 158: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.8.0/24 -oA outputs/evasion-158`
- Example 159: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.9.0/24 -oA outputs/evasion-159`
- Example 160: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.0.0/24 -oA outputs/evasion-160`
- Example 161: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.1.0/24 -oA outputs/evasion-161`
- Example 162: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.2.0/24 -oA outputs/evasion-162`
- Example 163: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.3.0/24 -oA outputs/evasion-163`
- Example 164: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.4.0/24 -oA outputs/evasion-164`
- Example 165: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.5.0/24 -oA outputs/evasion-165`
- Example 166: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.6.0/24 -oA outputs/evasion-166`
- Example 167: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.7.0/24 -oA outputs/evasion-167`
- Example 168: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.8.0/24 -oA outputs/evasion-168`
- Example 169: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.9.0/24 -oA outputs/evasion-169`
- Example 170: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.0.0/24 -oA outputs/evasion-170`
- Example 171: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.1.0/24 -oA outputs/evasion-171`
- Example 172: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.2.0/24 -oA outputs/evasion-172`
- Example 173: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.3.0/24 -oA outputs/evasion-173`
- Example 174: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.4.0/24 -oA outputs/evasion-174`
- Example 175: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.5.0/24 -oA outputs/evasion-175`
- Example 176: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.6.0/24 -oA outputs/evasion-176`
- Example 177: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.7.0/24 -oA outputs/evasion-177`
- Example 178: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.8.0/24 -oA outputs/evasion-178`
- Example 179: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.9.0/24 -oA outputs/evasion-179`
- Example 180: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.0.0/24 -oA outputs/evasion-180`
- Example 181: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.1.0/24 -oA outputs/evasion-181`
- Example 182: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.2.0/24 -oA outputs/evasion-182`
- Example 183: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.3.0/24 -oA outputs/evasion-183`
- Example 184: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.4.0/24 -oA outputs/evasion-184`
- Example 185: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.5.0/24 -oA outputs/evasion-185`
- Example 186: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.6.0/24 -oA outputs/evasion-186`
- Example 187: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.7.0/24 -oA outputs/evasion-187`
- Example 188: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.8.0/24 -oA outputs/evasion-188`
- Example 189: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.9.0/24 -oA outputs/evasion-189`
- Example 190: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.0.0/24 -oA outputs/evasion-190`
- Example 191: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.1.0/24 -oA outputs/evasion-191`
- Example 192: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.2.0/24 -oA outputs/evasion-192`
- Example 193: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.3.0/24 -oA outputs/evasion-193`
- Example 194: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.4.0/24 -oA outputs/evasion-194`
- Example 195: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.5.0/24 -oA outputs/evasion-195`
- Example 196: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.6.0/24 -oA outputs/evasion-196`
- Example 197: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.7.0/24 -oA outputs/evasion-197`
- Example 198: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.8.0/24 -oA outputs/evasion-198`
- Example 199: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.9.0/24 -oA outputs/evasion-199`
- Example 200: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.1.0.0/24 -oA outputs/evasion-200`
```

## Reflection Questions
1. Which evasion option feels safest for beginners?
2. How do firewalls change scanning strategies?
3. How do IDS alerts affect your scan design?
4. How do Days 18 and 19 help you control evasion scans?
5. What should you document after a firewall-aware scan?

## Next Steps
Tomorrow (Day 22) you will learn about spoofing and decoy scanning.

**Congratulations on completing Day 21!** You now understand how to reduce detection risk responsibly.
