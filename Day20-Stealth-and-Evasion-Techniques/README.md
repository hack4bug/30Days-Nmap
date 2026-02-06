# Day 20 - Stealth and Evasion Techniques

## Day Overview
Welcome to Day 20! Today you will learn how Nmap can reduce scan noise and avoid detection in controlled and authorized environments.
This is an advanced topic, but we will keep it beginner-friendly and safe.

This lesson builds on the last three days:
- Day 17: Output formats and parsing (grepable and XML)
- Day 18: Automation with shell scripts
- Day 19: Scanning multiple targets and ranges

## Learning Objectives
By completing Day 20, you will be able to:
- Understand what ?stealth? means in network scanning
- Use Nmap options that reduce scan noise
- Adjust timing and rate limits to avoid triggering alerts
- Use host discovery and scan ordering wisely
- Document your approach for safe and authorized testing

## Important Ethics Note
Stealth and evasion techniques must only be used with explicit permission.
Never attempt to hide scans on networks you do not own or control.

## What Is Stealth in Nmap?
Stealth means scanning in a way that reduces the chance of detection by IDS/IPS or system logs.
It does not mean ?invisible.? Logs can still capture your activity.

## How Detection Works (Beginner View)
Detection usually happens when:
- Many ports are probed quickly
- Scans happen in predictable order
- Unusual packet types are seen
- Traffic patterns look automated

## Day 17 Recap: Why Output Still Matters
Even stealth scans need documentation and repeatability.
Use `-oA` to save all formats for later analysis:
```bash
nmap -sS -T2 -oA outputs/stealth-scan target
```

## Day 18 Recap: Automation and Safety
Automation helps you run careful, consistent scans.
Always log your scan options when doing stealth tests.

## Day 19 Recap: Scope Control
Stealth is not just about timing. It is also about limiting scope.
Use target lists and exclude lists to reduce risk.

## Core Stealth Options (Beginner-Friendly)

### 1. TCP SYN Scan (Stealth Scan)
The SYN scan is the classic ?stealth? scan because it does not complete the TCP handshake.
```bash
sudo nmap -sS target
```
Notes:
- Requires privileges
- Still logged by many systems
- A good balance of speed and stealth

### 2. Slow Timing Templates
Use slower timing to reduce detection:
```bash
nmap -sS -T2 target
```
Slower scans reduce bursty traffic.

### 3. Limit Packet Rate
```bash
nmap -sS --max-rate 50 target
```
Lower rates can reduce alerts on IDS/IPS.

### 4. Randomize Host Order
```bash
nmap -sS --randomize-hosts 192.168.1.0/24
```
This avoids scanning IPs in order.

### 5. Disable DNS Resolution
```bash
nmap -n -sS target
```
This speeds up scans and reduces DNS logs.

### 6. Scan Specific Ports Only
Avoid full port scans on large networks:
```bash
nmap -sS -p 22,80,443 target
```

### 7. Use a Host Discovery First
```bash
nmap -sn 192.168.1.0/24 -oA outputs/ping-sweep
```
Then scan only live hosts for fewer total packets.

## Stealth Is About Tradeoffs
Slower scans reduce detection but take more time.
Faster scans find results quickly but are more visible.

## Timing Templates Explained
- `-T0` Paranoid: very slow, rarely used
- `-T1` Sneaky: very slow and stealthy
- `-T2` Polite: slow, less intrusive
- `-T3` Normal: default
- `-T4` Aggressive: fast and noisy
- `-T5` Insane: fastest, highest detection risk

## Practical Stealth Examples

### Example 1: Low-Noise Single Target Scan
```bash
sudo nmap -sS -T2 --max-rate 30 -p 22,80,443 -oA outputs/low-noise target
```

### Example 2: Stealth Scan on a Small Range
```bash
sudo nmap -sS -T2 --randomize-hosts 192.168.1.0/29 -oA outputs/range-stealth
```

### Example 3: Quiet Scan with Disabled DNS
```bash
sudo nmap -sS -n -T2 10.0.0.0/24 -oA outputs/no-dns
```

## Using Decoys (Advanced, Use with Care)
Decoys add fake source IPs to hide the real one in a list.
Only use in controlled lab environments and with permission.
```bash
sudo nmap -sS -D RND:5 target
```
Notes:
- Decoys do not make you invisible
- Some networks will block or log them anyway

## Fragmentation (Advanced)
Fragmentation breaks packets into smaller pieces.
```bash
sudo nmap -sS -f target
```
Notes:
- May be blocked by modern networks
- Can reduce compatibility

## Source Port Manipulation (Advanced)
```bash
sudo nmap -sS --source-port 53 target
```
Some legacy filters trust traffic from common ports.
Modern firewalls often ignore this trick.

## Idle Scan (Expert Level)
Idle scan uses a ?zombie? host to hide your IP.
Only study this in isolated labs and with permission.
```bash
sudo nmap -sI zombie-host target
```

## Scan Delays and Timeouts
Reduce bursty traffic using delays:
```bash
sudo nmap -sS --scan-delay 200ms target
```
Set timeouts to avoid long hangs:
```bash
nmap -sS --host-timeout 5m target
```

## Combining Stealth with Multi-Target Scanning (Day 19)
```bash
sudo nmap -sS -T2 --max-rate 50 --randomize-hosts -iL targets.txt -oA outputs/stealth-multi
```

## Combining Stealth with Automation (Day 18)
```bash
#!/usr/bin/env bash
set -euo pipefail
TARGET_FILE="${1:-targets.txt}"
RUN_DIR="outputs/$(date +%Y%m%d-%H%M%S)"
mkdir -p "$RUN_DIR"
sudo nmap -sS -T2 --max-rate 50 --randomize-hosts -iL "$TARGET_FILE" -oA "$RUN_DIR/stealth"
```

## Logging and Reporting
Stealth scans should be documented carefully.
Log:
- Date and time
- Target scope
- Options used
- Output files

## Beginner-Friendly Safety Rules
- Always scan with permission
- Start with small ranges
- Use slower timing first
- Keep logs and output files
- Tell your team when scans are running

## Troubleshooting Stealth Scans

### Scan is Too Slow
```text
Fix:
- Move from -T1 to -T2 or -T3
- Increase --max-rate carefully
- Reduce the port list
```

### Too Many Alerts Triggered
```text
Fix:
- Reduce scan speed
- Randomize hosts
- Limit ports to those you need
```

### Results Look Incomplete
```text
Fix:
- Increase timeouts
- Remove too much delay
- Verify connectivity
```

## Mini Glossary
- IDS/IPS: Systems that detect or block suspicious traffic
- Stealth: Reducing scan noise, not becoming invisible
- SYN scan: Half-open TCP scan
- Decoy: Fake source IPs to mask the real one

## Step-by-Step Lab: Quiet Scan Practice
### Step 1: List Targets
```bash
nmap -sL 192.168.1.0/29
```
### Step 2: Find Live Hosts
```bash
nmap -sn 192.168.1.0/29 -oA outputs/quiet-ping
```
### Step 3: Stealth Scan Live Hosts
```bash
awk '/Up$/{print $2}' outputs/quiet-ping.gnmap > outputs/quiet-alive.txt
sudo nmap -sS -T2 -iL outputs/quiet-alive.txt -oA outputs/quiet-scan
```
### Step 4: Review Results
```bash
grep "/open/" outputs/quiet-scan.gnmap
```

## Large Network Strategy
- Do discovery first
- Scan only live hosts
- Randomize hosts
- Use conservative timing

## Example: Conservative Multi-Target Scan
```bash
sudo nmap -sS -T2 --max-rate 30 --randomize-hosts 10.0.0.0/24 -oA outputs/conservative
```

## Example: Low-Noise Web Focus
```bash
sudo nmap -sS -T2 -p 80,443,8080,8443 --max-rate 40 10.0.0.0/24 -oA outputs/low-noise-web
```

## Advanced Options Summary (Do Not Use Blindly)
- `-D` for decoys
- `-f` for fragmentation
- `--source-port` for source port spoofing
- `-sI` for idle scan

## Beginner-Friendly Checklists
### Pre-Scan Checklist
- I have permission
- I confirmed target scope
- I selected safe timing options
- I prepared output folders
- I documented the scan plan

### Post-Scan Checklist
- I saved all outputs
- I reviewed results for open ports
- I documented anomalies
- I stored scan logs safely

## Extended Practice Section
Below are repeated drills to build comfort with stealth options.
Use a lab environment and adjust IPs as needed.

### Drill 1: Low-Noise Scan
Goal: Run a stealth scan with safe timing and limited ports.
1. Choose a small target list or /29 range.
2. Run discovery to confirm live hosts.
3. Use `-sS` and `-T2` with a small port list.
4. Add `--max-rate` to control speed.
5. Save output using `-oA`.
6. Review `.gnmap` output for open ports.
7. Document options and results.

### Drill 2: Low-Noise Scan
Goal: Run a stealth scan with safe timing and limited ports.
1. Choose a small target list or /29 range.
2. Run discovery to confirm live hosts.
3. Use `-sS` and `-T2` with a small port list.
4. Add `--max-rate` to control speed.
5. Save output using `-oA`.
6. Review `.gnmap` output for open ports.
7. Document options and results.

### Drill 3: Low-Noise Scan
Goal: Run a stealth scan with safe timing and limited ports.
1. Choose a small target list or /29 range.
2. Run discovery to confirm live hosts.
3. Use `-sS` and `-T2` with a small port list.
4. Add `--max-rate` to control speed.
5. Save output using `-oA`.
6. Review `.gnmap` output for open ports.
7. Document options and results.

### Drill 4: Low-Noise Scan
Goal: Run a stealth scan with safe timing and limited ports.
1. Choose a small target list or /29 range.
2. Run discovery to confirm live hosts.
3. Use `-sS` and `-T2` with a small port list.
4. Add `--max-rate` to control speed.
5. Save output using `-oA`.
6. Review `.gnmap` output for open ports.
7. Document options and results.

### Drill 5: Low-Noise Scan
Goal: Run a stealth scan with safe timing and limited ports.
1. Choose a small target list or /29 range.
2. Run discovery to confirm live hosts.
3. Use `-sS` and `-T2` with a small port list.
4. Add `--max-rate` to control speed.
5. Save output using `-oA`.
6. Review `.gnmap` output for open ports.
7. Document options and results.

### Drill 6: Low-Noise Scan
Goal: Run a stealth scan with safe timing and limited ports.
1. Choose a small target list or /29 range.
2. Run discovery to confirm live hosts.
3. Use `-sS` and `-T2` with a small port list.
4. Add `--max-rate` to control speed.
5. Save output using `-oA`.
6. Review `.gnmap` output for open ports.
7. Document options and results.

### Drill 7: Low-Noise Scan
Goal: Run a stealth scan with safe timing and limited ports.
1. Choose a small target list or /29 range.
2. Run discovery to confirm live hosts.
3. Use `-sS` and `-T2` with a small port list.
4. Add `--max-rate` to control speed.
5. Save output using `-oA`.
6. Review `.gnmap` output for open ports.
7. Document options and results.

### Drill 8: Low-Noise Scan
Goal: Run a stealth scan with safe timing and limited ports.
1. Choose a small target list or /29 range.
2. Run discovery to confirm live hosts.
3. Use `-sS` and `-T2` with a small port list.
4. Add `--max-rate` to control speed.
5. Save output using `-oA`.
6. Review `.gnmap` output for open ports.
7. Document options and results.

### Drill 9: Low-Noise Scan
Goal: Run a stealth scan with safe timing and limited ports.
1. Choose a small target list or /29 range.
2. Run discovery to confirm live hosts.
3. Use `-sS` and `-T2` with a small port list.
4. Add `--max-rate` to control speed.
5. Save output using `-oA`.
6. Review `.gnmap` output for open ports.
7. Document options and results.

### Drill 10: Low-Noise Scan
Goal: Run a stealth scan with safe timing and limited ports.
1. Choose a small target list or /29 range.
2. Run discovery to confirm live hosts.
3. Use `-sS` and `-T2` with a small port list.
4. Add `--max-rate` to control speed.
5. Save output using `-oA`.
6. Review `.gnmap` output for open ports.
7. Document options and results.

### Drill 11: Low-Noise Scan
Goal: Run a stealth scan with safe timing and limited ports.
1. Choose a small target list or /29 range.
2. Run discovery to confirm live hosts.
3. Use `-sS` and `-T2` with a small port list.
4. Add `--max-rate` to control speed.
5. Save output using `-oA`.
6. Review `.gnmap` output for open ports.
7. Document options and results.

### Drill 12: Low-Noise Scan
Goal: Run a stealth scan with safe timing and limited ports.
1. Choose a small target list or /29 range.
2. Run discovery to confirm live hosts.
3. Use `-sS` and `-T2` with a small port list.
4. Add `--max-rate` to control speed.
5. Save output using `-oA`.
6. Review `.gnmap` output for open ports.
7. Document options and results.

### Drill 13: Low-Noise Scan
Goal: Run a stealth scan with safe timing and limited ports.
1. Choose a small target list or /29 range.
2. Run discovery to confirm live hosts.
3. Use `-sS` and `-T2` with a small port list.
4. Add `--max-rate` to control speed.
5. Save output using `-oA`.
6. Review `.gnmap` output for open ports.
7. Document options and results.

### Drill 14: Low-Noise Scan
Goal: Run a stealth scan with safe timing and limited ports.
1. Choose a small target list or /29 range.
2. Run discovery to confirm live hosts.
3. Use `-sS` and `-T2` with a small port list.
4. Add `--max-rate` to control speed.
5. Save output using `-oA`.
6. Review `.gnmap` output for open ports.
7. Document options and results.

### Drill 15: Low-Noise Scan
Goal: Run a stealth scan with safe timing and limited ports.
1. Choose a small target list or /29 range.
2. Run discovery to confirm live hosts.
3. Use `-sS` and `-T2` with a small port list.
4. Add `--max-rate` to control speed.
5. Save output using `-oA`.
6. Review `.gnmap` output for open ports.
7. Document options and results.

### Drill 16: Low-Noise Scan
Goal: Run a stealth scan with safe timing and limited ports.
1. Choose a small target list or /29 range.
2. Run discovery to confirm live hosts.
3. Use `-sS` and `-T2` with a small port list.
4. Add `--max-rate` to control speed.
5. Save output using `-oA`.
6. Review `.gnmap` output for open ports.
7. Document options and results.

### Drill 17: Low-Noise Scan
Goal: Run a stealth scan with safe timing and limited ports.
1. Choose a small target list or /29 range.
2. Run discovery to confirm live hosts.
3. Use `-sS` and `-T2` with a small port list.
4. Add `--max-rate` to control speed.
5. Save output using `-oA`.
6. Review `.gnmap` output for open ports.
7. Document options and results.

### Drill 18: Low-Noise Scan
Goal: Run a stealth scan with safe timing and limited ports.
1. Choose a small target list or /29 range.
2. Run discovery to confirm live hosts.
3. Use `-sS` and `-T2` with a small port list.
4. Add `--max-rate` to control speed.
5. Save output using `-oA`.
6. Review `.gnmap` output for open ports.
7. Document options and results.

### Drill 19: Low-Noise Scan
Goal: Run a stealth scan with safe timing and limited ports.
1. Choose a small target list or /29 range.
2. Run discovery to confirm live hosts.
3. Use `-sS` and `-T2` with a small port list.
4. Add `--max-rate` to control speed.
5. Save output using `-oA`.
6. Review `.gnmap` output for open ports.
7. Document options and results.

### Drill 20: Low-Noise Scan
Goal: Run a stealth scan with safe timing and limited ports.
1. Choose a small target list or /29 range.
2. Run discovery to confirm live hosts.
3. Use `-sS` and `-T2` with a small port list.
4. Add `--max-rate` to control speed.
5. Save output using `-oA`.
6. Review `.gnmap` output for open ports.
7. Document options and results.

### Drill 21: Low-Noise Scan
Goal: Run a stealth scan with safe timing and limited ports.
1. Choose a small target list or /29 range.
2. Run discovery to confirm live hosts.
3. Use `-sS` and `-T2` with a small port list.
4. Add `--max-rate` to control speed.
5. Save output using `-oA`.
6. Review `.gnmap` output for open ports.
7. Document options and results.

### Drill 22: Low-Noise Scan
Goal: Run a stealth scan with safe timing and limited ports.
1. Choose a small target list or /29 range.
2. Run discovery to confirm live hosts.
3. Use `-sS` and `-T2` with a small port list.
4. Add `--max-rate` to control speed.
5. Save output using `-oA`.
6. Review `.gnmap` output for open ports.
7. Document options and results.

### Drill 23: Low-Noise Scan
Goal: Run a stealth scan with safe timing and limited ports.
1. Choose a small target list or /29 range.
2. Run discovery to confirm live hosts.
3. Use `-sS` and `-T2` with a small port list.
4. Add `--max-rate` to control speed.
5. Save output using `-oA`.
6. Review `.gnmap` output for open ports.
7. Document options and results.

### Drill 24: Low-Noise Scan
Goal: Run a stealth scan with safe timing and limited ports.
1. Choose a small target list or /29 range.
2. Run discovery to confirm live hosts.
3. Use `-sS` and `-T2` with a small port list.
4. Add `--max-rate` to control speed.
5. Save output using `-oA`.
6. Review `.gnmap` output for open ports.
7. Document options and results.

### Drill 25: Low-Noise Scan
Goal: Run a stealth scan with safe timing and limited ports.
1. Choose a small target list or /29 range.
2. Run discovery to confirm live hosts.
3. Use `-sS` and `-T2` with a small port list.
4. Add `--max-rate` to control speed.
5. Save output using `-oA`.
6. Review `.gnmap` output for open ports.
7. Document options and results.

### Drill 26: Low-Noise Scan
Goal: Run a stealth scan with safe timing and limited ports.
1. Choose a small target list or /29 range.
2. Run discovery to confirm live hosts.
3. Use `-sS` and `-T2` with a small port list.
4. Add `--max-rate` to control speed.
5. Save output using `-oA`.
6. Review `.gnmap` output for open ports.
7. Document options and results.

### Drill 27: Low-Noise Scan
Goal: Run a stealth scan with safe timing and limited ports.
1. Choose a small target list or /29 range.
2. Run discovery to confirm live hosts.
3. Use `-sS` and `-T2` with a small port list.
4. Add `--max-rate` to control speed.
5. Save output using `-oA`.
6. Review `.gnmap` output for open ports.
7. Document options and results.

### Drill 28: Low-Noise Scan
Goal: Run a stealth scan with safe timing and limited ports.
1. Choose a small target list or /29 range.
2. Run discovery to confirm live hosts.
3. Use `-sS` and `-T2` with a small port list.
4. Add `--max-rate` to control speed.
5. Save output using `-oA`.
6. Review `.gnmap` output for open ports.
7. Document options and results.

### Drill 29: Low-Noise Scan
Goal: Run a stealth scan with safe timing and limited ports.
1. Choose a small target list or /29 range.
2. Run discovery to confirm live hosts.
3. Use `-sS` and `-T2` with a small port list.
4. Add `--max-rate` to control speed.
5. Save output using `-oA`.
6. Review `.gnmap` output for open ports.
7. Document options and results.

### Drill 30: Low-Noise Scan
Goal: Run a stealth scan with safe timing and limited ports.
1. Choose a small target list or /29 range.
2. Run discovery to confirm live hosts.
3. Use `-sS` and `-T2` with a small port list.
4. Add `--max-rate` to control speed.
5. Save output using `-oA`.
6. Review `.gnmap` output for open ports.
7. Document options and results.

### Drill 31: Low-Noise Scan
Goal: Run a stealth scan with safe timing and limited ports.
1. Choose a small target list or /29 range.
2. Run discovery to confirm live hosts.
3. Use `-sS` and `-T2` with a small port list.
4. Add `--max-rate` to control speed.
5. Save output using `-oA`.
6. Review `.gnmap` output for open ports.
7. Document options and results.

### Drill 32: Low-Noise Scan
Goal: Run a stealth scan with safe timing and limited ports.
1. Choose a small target list or /29 range.
2. Run discovery to confirm live hosts.
3. Use `-sS` and `-T2` with a small port list.
4. Add `--max-rate` to control speed.
5. Save output using `-oA`.
6. Review `.gnmap` output for open ports.
7. Document options and results.

### Drill 33: Low-Noise Scan
Goal: Run a stealth scan with safe timing and limited ports.
1. Choose a small target list or /29 range.
2. Run discovery to confirm live hosts.
3. Use `-sS` and `-T2` with a small port list.
4. Add `--max-rate` to control speed.
5. Save output using `-oA`.
6. Review `.gnmap` output for open ports.
7. Document options and results.

### Drill 34: Low-Noise Scan
Goal: Run a stealth scan with safe timing and limited ports.
1. Choose a small target list or /29 range.
2. Run discovery to confirm live hosts.
3. Use `-sS` and `-T2` with a small port list.
4. Add `--max-rate` to control speed.
5. Save output using `-oA`.
6. Review `.gnmap` output for open ports.
7. Document options and results.

### Drill 35: Low-Noise Scan
Goal: Run a stealth scan with safe timing and limited ports.
1. Choose a small target list or /29 range.
2. Run discovery to confirm live hosts.
3. Use `-sS` and `-T2` with a small port list.
4. Add `--max-rate` to control speed.
5. Save output using `-oA`.
6. Review `.gnmap` output for open ports.
7. Document options and results.

### Drill 36: Low-Noise Scan
Goal: Run a stealth scan with safe timing and limited ports.
1. Choose a small target list or /29 range.
2. Run discovery to confirm live hosts.
3. Use `-sS` and `-T2` with a small port list.
4. Add `--max-rate` to control speed.
5. Save output using `-oA`.
6. Review `.gnmap` output for open ports.
7. Document options and results.

### Drill 37: Low-Noise Scan
Goal: Run a stealth scan with safe timing and limited ports.
1. Choose a small target list or /29 range.
2. Run discovery to confirm live hosts.
3. Use `-sS` and `-T2` with a small port list.
4. Add `--max-rate` to control speed.
5. Save output using `-oA`.
6. Review `.gnmap` output for open ports.
7. Document options and results.

### Drill 38: Low-Noise Scan
Goal: Run a stealth scan with safe timing and limited ports.
1. Choose a small target list or /29 range.
2. Run discovery to confirm live hosts.
3. Use `-sS` and `-T2` with a small port list.
4. Add `--max-rate` to control speed.
5. Save output using `-oA`.
6. Review `.gnmap` output for open ports.
7. Document options and results.

### Drill 39: Low-Noise Scan
Goal: Run a stealth scan with safe timing and limited ports.
1. Choose a small target list or /29 range.
2. Run discovery to confirm live hosts.
3. Use `-sS` and `-T2` with a small port list.
4. Add `--max-rate` to control speed.
5. Save output using `-oA`.
6. Review `.gnmap` output for open ports.
7. Document options and results.

### Drill 40: Low-Noise Scan
Goal: Run a stealth scan with safe timing and limited ports.
1. Choose a small target list or /29 range.
2. Run discovery to confirm live hosts.
3. Use `-sS` and `-T2` with a small port list.
4. Add `--max-rate` to control speed.
5. Save output using `-oA`.
6. Review `.gnmap` output for open ports.
7. Document options and results.

### Drill 41: Low-Noise Scan
Goal: Run a stealth scan with safe timing and limited ports.
1. Choose a small target list or /29 range.
2. Run discovery to confirm live hosts.
3. Use `-sS` and `-T2` with a small port list.
4. Add `--max-rate` to control speed.
5. Save output using `-oA`.
6. Review `.gnmap` output for open ports.
7. Document options and results.

### Drill 42: Low-Noise Scan
Goal: Run a stealth scan with safe timing and limited ports.
1. Choose a small target list or /29 range.
2. Run discovery to confirm live hosts.
3. Use `-sS` and `-T2` with a small port list.
4. Add `--max-rate` to control speed.
5. Save output using `-oA`.
6. Review `.gnmap` output for open ports.
7. Document options and results.

### Drill 43: Low-Noise Scan
Goal: Run a stealth scan with safe timing and limited ports.
1. Choose a small target list or /29 range.
2. Run discovery to confirm live hosts.
3. Use `-sS` and `-T2` with a small port list.
4. Add `--max-rate` to control speed.
5. Save output using `-oA`.
6. Review `.gnmap` output for open ports.
7. Document options and results.

### Drill 44: Low-Noise Scan
Goal: Run a stealth scan with safe timing and limited ports.
1. Choose a small target list or /29 range.
2. Run discovery to confirm live hosts.
3. Use `-sS` and `-T2` with a small port list.
4. Add `--max-rate` to control speed.
5. Save output using `-oA`.
6. Review `.gnmap` output for open ports.
7. Document options and results.

### Drill 45: Low-Noise Scan
Goal: Run a stealth scan with safe timing and limited ports.
1. Choose a small target list or /29 range.
2. Run discovery to confirm live hosts.
3. Use `-sS` and `-T2` with a small port list.
4. Add `--max-rate` to control speed.
5. Save output using `-oA`.
6. Review `.gnmap` output for open ports.
7. Document options and results.

### Drill 46: Low-Noise Scan
Goal: Run a stealth scan with safe timing and limited ports.
1. Choose a small target list or /29 range.
2. Run discovery to confirm live hosts.
3. Use `-sS` and `-T2` with a small port list.
4. Add `--max-rate` to control speed.
5. Save output using `-oA`.
6. Review `.gnmap` output for open ports.
7. Document options and results.

### Drill 47: Low-Noise Scan
Goal: Run a stealth scan with safe timing and limited ports.
1. Choose a small target list or /29 range.
2. Run discovery to confirm live hosts.
3. Use `-sS` and `-T2` with a small port list.
4. Add `--max-rate` to control speed.
5. Save output using `-oA`.
6. Review `.gnmap` output for open ports.
7. Document options and results.

### Drill 48: Low-Noise Scan
Goal: Run a stealth scan with safe timing and limited ports.
1. Choose a small target list or /29 range.
2. Run discovery to confirm live hosts.
3. Use `-sS` and `-T2` with a small port list.
4. Add `--max-rate` to control speed.
5. Save output using `-oA`.
6. Review `.gnmap` output for open ports.
7. Document options and results.

### Drill 49: Low-Noise Scan
Goal: Run a stealth scan with safe timing and limited ports.
1. Choose a small target list or /29 range.
2. Run discovery to confirm live hosts.
3. Use `-sS` and `-T2` with a small port list.
4. Add `--max-rate` to control speed.
5. Save output using `-oA`.
6. Review `.gnmap` output for open ports.
7. Document options and results.

### Drill 50: Low-Noise Scan
Goal: Run a stealth scan with safe timing and limited ports.
1. Choose a small target list or /29 range.
2. Run discovery to confirm live hosts.
3. Use `-sS` and `-T2` with a small port list.
4. Add `--max-rate` to control speed.
5. Save output using `-oA`.
6. Review `.gnmap` output for open ports.
7. Document options and results.

### Drill 51: Low-Noise Scan
Goal: Run a stealth scan with safe timing and limited ports.
1. Choose a small target list or /29 range.
2. Run discovery to confirm live hosts.
3. Use `-sS` and `-T2` with a small port list.
4. Add `--max-rate` to control speed.
5. Save output using `-oA`.
6. Review `.gnmap` output for open ports.
7. Document options and results.

### Drill 52: Low-Noise Scan
Goal: Run a stealth scan with safe timing and limited ports.
1. Choose a small target list or /29 range.
2. Run discovery to confirm live hosts.
3. Use `-sS` and `-T2` with a small port list.
4. Add `--max-rate` to control speed.
5. Save output using `-oA`.
6. Review `.gnmap` output for open ports.
7. Document options and results.

### Drill 53: Low-Noise Scan
Goal: Run a stealth scan with safe timing and limited ports.
1. Choose a small target list or /29 range.
2. Run discovery to confirm live hosts.
3. Use `-sS` and `-T2` with a small port list.
4. Add `--max-rate` to control speed.
5. Save output using `-oA`.
6. Review `.gnmap` output for open ports.
7. Document options and results.

### Drill 54: Low-Noise Scan
Goal: Run a stealth scan with safe timing and limited ports.
1. Choose a small target list or /29 range.
2. Run discovery to confirm live hosts.
3. Use `-sS` and `-T2` with a small port list.
4. Add `--max-rate` to control speed.
5. Save output using `-oA`.
6. Review `.gnmap` output for open ports.
7. Document options and results.

### Drill 55: Low-Noise Scan
Goal: Run a stealth scan with safe timing and limited ports.
1. Choose a small target list or /29 range.
2. Run discovery to confirm live hosts.
3. Use `-sS` and `-T2` with a small port list.
4. Add `--max-rate` to control speed.
5. Save output using `-oA`.
6. Review `.gnmap` output for open ports.
7. Document options and results.

### Drill 56: Low-Noise Scan
Goal: Run a stealth scan with safe timing and limited ports.
1. Choose a small target list or /29 range.
2. Run discovery to confirm live hosts.
3. Use `-sS` and `-T2` with a small port list.
4. Add `--max-rate` to control speed.
5. Save output using `-oA`.
6. Review `.gnmap` output for open ports.
7. Document options and results.

### Drill 57: Low-Noise Scan
Goal: Run a stealth scan with safe timing and limited ports.
1. Choose a small target list or /29 range.
2. Run discovery to confirm live hosts.
3. Use `-sS` and `-T2` with a small port list.
4. Add `--max-rate` to control speed.
5. Save output using `-oA`.
6. Review `.gnmap` output for open ports.
7. Document options and results.

### Drill 58: Low-Noise Scan
Goal: Run a stealth scan with safe timing and limited ports.
1. Choose a small target list or /29 range.
2. Run discovery to confirm live hosts.
3. Use `-sS` and `-T2` with a small port list.
4. Add `--max-rate` to control speed.
5. Save output using `-oA`.
6. Review `.gnmap` output for open ports.
7. Document options and results.

### Drill 59: Low-Noise Scan
Goal: Run a stealth scan with safe timing and limited ports.
1. Choose a small target list or /29 range.
2. Run discovery to confirm live hosts.
3. Use `-sS` and `-T2` with a small port list.
4. Add `--max-rate` to control speed.
5. Save output using `-oA`.
6. Review `.gnmap` output for open ports.
7. Document options and results.

### Drill 60: Low-Noise Scan
Goal: Run a stealth scan with safe timing and limited ports.
1. Choose a small target list or /29 range.
2. Run discovery to confirm live hosts.
3. Use `-sS` and `-T2` with a small port list.
4. Add `--max-rate` to control speed.
5. Save output using `-oA`.
6. Review `.gnmap` output for open ports.
7. Document options and results.

### Drill 61: Low-Noise Scan
Goal: Run a stealth scan with safe timing and limited ports.
1. Choose a small target list or /29 range.
2. Run discovery to confirm live hosts.
3. Use `-sS` and `-T2` with a small port list.
4. Add `--max-rate` to control speed.
5. Save output using `-oA`.
6. Review `.gnmap` output for open ports.
7. Document options and results.

### Drill 62: Low-Noise Scan
Goal: Run a stealth scan with safe timing and limited ports.
1. Choose a small target list or /29 range.
2. Run discovery to confirm live hosts.
3. Use `-sS` and `-T2` with a small port list.
4. Add `--max-rate` to control speed.
5. Save output using `-oA`.
6. Review `.gnmap` output for open ports.
7. Document options and results.

### Drill 63: Low-Noise Scan
Goal: Run a stealth scan with safe timing and limited ports.
1. Choose a small target list or /29 range.
2. Run discovery to confirm live hosts.
3. Use `-sS` and `-T2` with a small port list.
4. Add `--max-rate` to control speed.
5. Save output using `-oA`.
6. Review `.gnmap` output for open ports.
7. Document options and results.

### Drill 64: Low-Noise Scan
Goal: Run a stealth scan with safe timing and limited ports.
1. Choose a small target list or /29 range.
2. Run discovery to confirm live hosts.
3. Use `-sS` and `-T2` with a small port list.
4. Add `--max-rate` to control speed.
5. Save output using `-oA`.
6. Review `.gnmap` output for open ports.
7. Document options and results.

### Drill 65: Low-Noise Scan
Goal: Run a stealth scan with safe timing and limited ports.
1. Choose a small target list or /29 range.
2. Run discovery to confirm live hosts.
3. Use `-sS` and `-T2` with a small port list.
4. Add `--max-rate` to control speed.
5. Save output using `-oA`.
6. Review `.gnmap` output for open ports.
7. Document options and results.

### Drill 66: Low-Noise Scan
Goal: Run a stealth scan with safe timing and limited ports.
1. Choose a small target list or /29 range.
2. Run discovery to confirm live hosts.
3. Use `-sS` and `-T2` with a small port list.
4. Add `--max-rate` to control speed.
5. Save output using `-oA`.
6. Review `.gnmap` output for open ports.
7. Document options and results.

### Drill 67: Low-Noise Scan
Goal: Run a stealth scan with safe timing and limited ports.
1. Choose a small target list or /29 range.
2. Run discovery to confirm live hosts.
3. Use `-sS` and `-T2` with a small port list.
4. Add `--max-rate` to control speed.
5. Save output using `-oA`.
6. Review `.gnmap` output for open ports.
7. Document options and results.

### Drill 68: Low-Noise Scan
Goal: Run a stealth scan with safe timing and limited ports.
1. Choose a small target list or /29 range.
2. Run discovery to confirm live hosts.
3. Use `-sS` and `-T2` with a small port list.
4. Add `--max-rate` to control speed.
5. Save output using `-oA`.
6. Review `.gnmap` output for open ports.
7. Document options and results.

### Drill 69: Low-Noise Scan
Goal: Run a stealth scan with safe timing and limited ports.
1. Choose a small target list or /29 range.
2. Run discovery to confirm live hosts.
3. Use `-sS` and `-T2` with a small port list.
4. Add `--max-rate` to control speed.
5. Save output using `-oA`.
6. Review `.gnmap` output for open ports.
7. Document options and results.

### Drill 70: Low-Noise Scan
Goal: Run a stealth scan with safe timing and limited ports.
1. Choose a small target list or /29 range.
2. Run discovery to confirm live hosts.
3. Use `-sS` and `-T2` with a small port list.
4. Add `--max-rate` to control speed.
5. Save output using `-oA`.
6. Review `.gnmap` output for open ports.
7. Document options and results.

### Drill 71: Low-Noise Scan
Goal: Run a stealth scan with safe timing and limited ports.
1. Choose a small target list or /29 range.
2. Run discovery to confirm live hosts.
3. Use `-sS` and `-T2` with a small port list.
4. Add `--max-rate` to control speed.
5. Save output using `-oA`.
6. Review `.gnmap` output for open ports.
7. Document options and results.

### Drill 72: Low-Noise Scan
Goal: Run a stealth scan with safe timing and limited ports.
1. Choose a small target list or /29 range.
2. Run discovery to confirm live hosts.
3. Use `-sS` and `-T2` with a small port list.
4. Add `--max-rate` to control speed.
5. Save output using `-oA`.
6. Review `.gnmap` output for open ports.
7. Document options and results.

### Drill 73: Low-Noise Scan
Goal: Run a stealth scan with safe timing and limited ports.
1. Choose a small target list or /29 range.
2. Run discovery to confirm live hosts.
3. Use `-sS` and `-T2` with a small port list.
4. Add `--max-rate` to control speed.
5. Save output using `-oA`.
6. Review `.gnmap` output for open ports.
7. Document options and results.

### Drill 74: Low-Noise Scan
Goal: Run a stealth scan with safe timing and limited ports.
1. Choose a small target list or /29 range.
2. Run discovery to confirm live hosts.
3. Use `-sS` and `-T2` with a small port list.
4. Add `--max-rate` to control speed.
5. Save output using `-oA`.
6. Review `.gnmap` output for open ports.
7. Document options and results.

### Drill 75: Low-Noise Scan
Goal: Run a stealth scan with safe timing and limited ports.
1. Choose a small target list or /29 range.
2. Run discovery to confirm live hosts.
3. Use `-sS` and `-T2` with a small port list.
4. Add `--max-rate` to control speed.
5. Save output using `-oA`.
6. Review `.gnmap` output for open ports.
7. Document options and results.

### Drill 76: Low-Noise Scan
Goal: Run a stealth scan with safe timing and limited ports.
1. Choose a small target list or /29 range.
2. Run discovery to confirm live hosts.
3. Use `-sS` and `-T2` with a small port list.
4. Add `--max-rate` to control speed.
5. Save output using `-oA`.
6. Review `.gnmap` output for open ports.
7. Document options and results.

### Drill 77: Low-Noise Scan
Goal: Run a stealth scan with safe timing and limited ports.
1. Choose a small target list or /29 range.
2. Run discovery to confirm live hosts.
3. Use `-sS` and `-T2` with a small port list.
4. Add `--max-rate` to control speed.
5. Save output using `-oA`.
6. Review `.gnmap` output for open ports.
7. Document options and results.

### Drill 78: Low-Noise Scan
Goal: Run a stealth scan with safe timing and limited ports.
1. Choose a small target list or /29 range.
2. Run discovery to confirm live hosts.
3. Use `-sS` and `-T2` with a small port list.
4. Add `--max-rate` to control speed.
5. Save output using `-oA`.
6. Review `.gnmap` output for open ports.
7. Document options and results.

### Drill 79: Low-Noise Scan
Goal: Run a stealth scan with safe timing and limited ports.
1. Choose a small target list or /29 range.
2. Run discovery to confirm live hosts.
3. Use `-sS` and `-T2` with a small port list.
4. Add `--max-rate` to control speed.
5. Save output using `-oA`.
6. Review `.gnmap` output for open ports.
7. Document options and results.

### Drill 80: Low-Noise Scan
Goal: Run a stealth scan with safe timing and limited ports.
1. Choose a small target list or /29 range.
2. Run discovery to confirm live hosts.
3. Use `-sS` and `-T2` with a small port list.
4. Add `--max-rate` to control speed.
5. Save output using `-oA`.
6. Review `.gnmap` output for open ports.
7. Document options and results.

### Drill 81: Low-Noise Scan
Goal: Run a stealth scan with safe timing and limited ports.
1. Choose a small target list or /29 range.
2. Run discovery to confirm live hosts.
3. Use `-sS` and `-T2` with a small port list.
4. Add `--max-rate` to control speed.
5. Save output using `-oA`.
6. Review `.gnmap` output for open ports.
7. Document options and results.

### Drill 82: Low-Noise Scan
Goal: Run a stealth scan with safe timing and limited ports.
1. Choose a small target list or /29 range.
2. Run discovery to confirm live hosts.
3. Use `-sS` and `-T2` with a small port list.
4. Add `--max-rate` to control speed.
5. Save output using `-oA`.
6. Review `.gnmap` output for open ports.
7. Document options and results.

### Drill 83: Low-Noise Scan
Goal: Run a stealth scan with safe timing and limited ports.
1. Choose a small target list or /29 range.
2. Run discovery to confirm live hosts.
3. Use `-sS` and `-T2` with a small port list.
4. Add `--max-rate` to control speed.
5. Save output using `-oA`.
6. Review `.gnmap` output for open ports.
7. Document options and results.

### Drill 84: Low-Noise Scan
Goal: Run a stealth scan with safe timing and limited ports.
1. Choose a small target list or /29 range.
2. Run discovery to confirm live hosts.
3. Use `-sS` and `-T2` with a small port list.
4. Add `--max-rate` to control speed.
5. Save output using `-oA`.
6. Review `.gnmap` output for open ports.
7. Document options and results.

### Drill 85: Low-Noise Scan
Goal: Run a stealth scan with safe timing and limited ports.
1. Choose a small target list or /29 range.
2. Run discovery to confirm live hosts.
3. Use `-sS` and `-T2` with a small port list.
4. Add `--max-rate` to control speed.
5. Save output using `-oA`.
6. Review `.gnmap` output for open ports.
7. Document options and results.

### Drill 86: Low-Noise Scan
Goal: Run a stealth scan with safe timing and limited ports.
1. Choose a small target list or /29 range.
2. Run discovery to confirm live hosts.
3. Use `-sS` and `-T2` with a small port list.
4. Add `--max-rate` to control speed.
5. Save output using `-oA`.
6. Review `.gnmap` output for open ports.
7. Document options and results.

### Drill 87: Low-Noise Scan
Goal: Run a stealth scan with safe timing and limited ports.
1. Choose a small target list or /29 range.
2. Run discovery to confirm live hosts.
3. Use `-sS` and `-T2` with a small port list.
4. Add `--max-rate` to control speed.
5. Save output using `-oA`.
6. Review `.gnmap` output for open ports.
7. Document options and results.

### Drill 88: Low-Noise Scan
Goal: Run a stealth scan with safe timing and limited ports.
1. Choose a small target list or /29 range.
2. Run discovery to confirm live hosts.
3. Use `-sS` and `-T2` with a small port list.
4. Add `--max-rate` to control speed.
5. Save output using `-oA`.
6. Review `.gnmap` output for open ports.
7. Document options and results.

### Drill 89: Low-Noise Scan
Goal: Run a stealth scan with safe timing and limited ports.
1. Choose a small target list or /29 range.
2. Run discovery to confirm live hosts.
3. Use `-sS` and `-T2` with a small port list.
4. Add `--max-rate` to control speed.
5. Save output using `-oA`.
6. Review `.gnmap` output for open ports.
7. Document options and results.

### Drill 90: Low-Noise Scan
Goal: Run a stealth scan with safe timing and limited ports.
1. Choose a small target list or /29 range.
2. Run discovery to confirm live hosts.
3. Use `-sS` and `-T2` with a small port list.
4. Add `--max-rate` to control speed.
5. Save output using `-oA`.
6. Review `.gnmap` output for open ports.
7. Document options and results.

### Drill 91: Low-Noise Scan
Goal: Run a stealth scan with safe timing and limited ports.
1. Choose a small target list or /29 range.
2. Run discovery to confirm live hosts.
3. Use `-sS` and `-T2` with a small port list.
4. Add `--max-rate` to control speed.
5. Save output using `-oA`.
6. Review `.gnmap` output for open ports.
7. Document options and results.

### Drill 92: Low-Noise Scan
Goal: Run a stealth scan with safe timing and limited ports.
1. Choose a small target list or /29 range.
2. Run discovery to confirm live hosts.
3. Use `-sS` and `-T2` with a small port list.
4. Add `--max-rate` to control speed.
5. Save output using `-oA`.
6. Review `.gnmap` output for open ports.
7. Document options and results.

### Drill 93: Low-Noise Scan
Goal: Run a stealth scan with safe timing and limited ports.
1. Choose a small target list or /29 range.
2. Run discovery to confirm live hosts.
3. Use `-sS` and `-T2` with a small port list.
4. Add `--max-rate` to control speed.
5. Save output using `-oA`.
6. Review `.gnmap` output for open ports.
7. Document options and results.

### Drill 94: Low-Noise Scan
Goal: Run a stealth scan with safe timing and limited ports.
1. Choose a small target list or /29 range.
2. Run discovery to confirm live hosts.
3. Use `-sS` and `-T2` with a small port list.
4. Add `--max-rate` to control speed.
5. Save output using `-oA`.
6. Review `.gnmap` output for open ports.
7. Document options and results.

### Drill 95: Low-Noise Scan
Goal: Run a stealth scan with safe timing and limited ports.
1. Choose a small target list or /29 range.
2. Run discovery to confirm live hosts.
3. Use `-sS` and `-T2` with a small port list.
4. Add `--max-rate` to control speed.
5. Save output using `-oA`.
6. Review `.gnmap` output for open ports.
7. Document options and results.

### Drill 96: Low-Noise Scan
Goal: Run a stealth scan with safe timing and limited ports.
1. Choose a small target list or /29 range.
2. Run discovery to confirm live hosts.
3. Use `-sS` and `-T2` with a small port list.
4. Add `--max-rate` to control speed.
5. Save output using `-oA`.
6. Review `.gnmap` output for open ports.
7. Document options and results.

### Drill 97: Low-Noise Scan
Goal: Run a stealth scan with safe timing and limited ports.
1. Choose a small target list or /29 range.
2. Run discovery to confirm live hosts.
3. Use `-sS` and `-T2` with a small port list.
4. Add `--max-rate` to control speed.
5. Save output using `-oA`.
6. Review `.gnmap` output for open ports.
7. Document options and results.

### Drill 98: Low-Noise Scan
Goal: Run a stealth scan with safe timing and limited ports.
1. Choose a small target list or /29 range.
2. Run discovery to confirm live hosts.
3. Use `-sS` and `-T2` with a small port list.
4. Add `--max-rate` to control speed.
5. Save output using `-oA`.
6. Review `.gnmap` output for open ports.
7. Document options and results.

### Drill 99: Low-Noise Scan
Goal: Run a stealth scan with safe timing and limited ports.
1. Choose a small target list or /29 range.
2. Run discovery to confirm live hosts.
3. Use `-sS` and `-T2` with a small port list.
4. Add `--max-rate` to control speed.
5. Save output using `-oA`.
6. Review `.gnmap` output for open ports.
7. Document options and results.

### Drill 100: Low-Noise Scan
Goal: Run a stealth scan with safe timing and limited ports.
1. Choose a small target list or /29 range.
2. Run discovery to confirm live hosts.
3. Use `-sS` and `-T2` with a small port list.
4. Add `--max-rate` to control speed.
5. Save output using `-oA`.
6. Review `.gnmap` output for open ports.
7. Document options and results.

```powershell
## Command Reference (One-Liners)
- Example 1: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.1.0/24 -oA outputs/stealth-1`
- Example 2: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.2.0/24 -oA outputs/stealth-2`
- Example 3: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.3.0/24 -oA outputs/stealth-3`
- Example 4: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.4.0/24 -oA outputs/stealth-4`
- Example 5: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.5.0/24 -oA outputs/stealth-5`
- Example 6: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.6.0/24 -oA outputs/stealth-6`
- Example 7: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.7.0/24 -oA outputs/stealth-7`
- Example 8: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.8.0/24 -oA outputs/stealth-8`
- Example 9: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.9.0/24 -oA outputs/stealth-9`
- Example 10: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.0.0/24 -oA outputs/stealth-10`
- Example 11: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.1.0/24 -oA outputs/stealth-11`
- Example 12: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.2.0/24 -oA outputs/stealth-12`
- Example 13: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.3.0/24 -oA outputs/stealth-13`
- Example 14: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.4.0/24 -oA outputs/stealth-14`
- Example 15: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.5.0/24 -oA outputs/stealth-15`
- Example 16: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.6.0/24 -oA outputs/stealth-16`
- Example 17: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.7.0/24 -oA outputs/stealth-17`
- Example 18: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.8.0/24 -oA outputs/stealth-18`
- Example 19: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.9.0/24 -oA outputs/stealth-19`
- Example 20: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.0.0/24 -oA outputs/stealth-20`
- Example 21: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.1.0/24 -oA outputs/stealth-21`
- Example 22: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.2.0/24 -oA outputs/stealth-22`
- Example 23: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.3.0/24 -oA outputs/stealth-23`
- Example 24: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.4.0/24 -oA outputs/stealth-24`
- Example 25: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.5.0/24 -oA outputs/stealth-25`
- Example 26: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.6.0/24 -oA outputs/stealth-26`
- Example 27: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.7.0/24 -oA outputs/stealth-27`
- Example 28: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.8.0/24 -oA outputs/stealth-28`
- Example 29: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.9.0/24 -oA outputs/stealth-29`
- Example 30: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.0.0/24 -oA outputs/stealth-30`
- Example 31: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.1.0/24 -oA outputs/stealth-31`
- Example 32: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.2.0/24 -oA outputs/stealth-32`
- Example 33: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.3.0/24 -oA outputs/stealth-33`
- Example 34: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.4.0/24 -oA outputs/stealth-34`
- Example 35: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.5.0/24 -oA outputs/stealth-35`
- Example 36: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.6.0/24 -oA outputs/stealth-36`
- Example 37: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.7.0/24 -oA outputs/stealth-37`
- Example 38: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.8.0/24 -oA outputs/stealth-38`
- Example 39: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.9.0/24 -oA outputs/stealth-39`
- Example 40: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.0.0/24 -oA outputs/stealth-40`
- Example 41: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.1.0/24 -oA outputs/stealth-41`
- Example 42: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.2.0/24 -oA outputs/stealth-42`
- Example 43: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.3.0/24 -oA outputs/stealth-43`
- Example 44: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.4.0/24 -oA outputs/stealth-44`
- Example 45: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.5.0/24 -oA outputs/stealth-45`
- Example 46: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.6.0/24 -oA outputs/stealth-46`
- Example 47: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.7.0/24 -oA outputs/stealth-47`
- Example 48: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.8.0/24 -oA outputs/stealth-48`
- Example 49: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.9.0/24 -oA outputs/stealth-49`
- Example 50: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.0.0/24 -oA outputs/stealth-50`
- Example 51: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.1.0/24 -oA outputs/stealth-51`
- Example 52: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.2.0/24 -oA outputs/stealth-52`
- Example 53: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.3.0/24 -oA outputs/stealth-53`
- Example 54: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.4.0/24 -oA outputs/stealth-54`
- Example 55: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.5.0/24 -oA outputs/stealth-55`
- Example 56: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.6.0/24 -oA outputs/stealth-56`
- Example 57: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.7.0/24 -oA outputs/stealth-57`
- Example 58: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.8.0/24 -oA outputs/stealth-58`
- Example 59: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.9.0/24 -oA outputs/stealth-59`
- Example 60: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.0.0/24 -oA outputs/stealth-60`
- Example 61: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.1.0/24 -oA outputs/stealth-61`
- Example 62: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.2.0/24 -oA outputs/stealth-62`
- Example 63: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.3.0/24 -oA outputs/stealth-63`
- Example 64: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.4.0/24 -oA outputs/stealth-64`
- Example 65: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.5.0/24 -oA outputs/stealth-65`
- Example 66: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.6.0/24 -oA outputs/stealth-66`
- Example 67: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.7.0/24 -oA outputs/stealth-67`
- Example 68: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.8.0/24 -oA outputs/stealth-68`
- Example 69: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.9.0/24 -oA outputs/stealth-69`
- Example 70: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.0.0/24 -oA outputs/stealth-70`
- Example 71: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.1.0/24 -oA outputs/stealth-71`
- Example 72: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.2.0/24 -oA outputs/stealth-72`
- Example 73: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.3.0/24 -oA outputs/stealth-73`
- Example 74: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.4.0/24 -oA outputs/stealth-74`
- Example 75: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.5.0/24 -oA outputs/stealth-75`
- Example 76: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.6.0/24 -oA outputs/stealth-76`
- Example 77: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.7.0/24 -oA outputs/stealth-77`
- Example 78: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.8.0/24 -oA outputs/stealth-78`
- Example 79: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.9.0/24 -oA outputs/stealth-79`
- Example 80: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.0.0/24 -oA outputs/stealth-80`
- Example 81: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.1.0/24 -oA outputs/stealth-81`
- Example 82: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.2.0/24 -oA outputs/stealth-82`
- Example 83: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.3.0/24 -oA outputs/stealth-83`
- Example 84: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.4.0/24 -oA outputs/stealth-84`
- Example 85: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.5.0/24 -oA outputs/stealth-85`
- Example 86: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.6.0/24 -oA outputs/stealth-86`
- Example 87: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.7.0/24 -oA outputs/stealth-87`
- Example 88: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.8.0/24 -oA outputs/stealth-88`
- Example 89: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.9.0/24 -oA outputs/stealth-89`
- Example 90: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.0.0/24 -oA outputs/stealth-90`
- Example 91: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.1.0/24 -oA outputs/stealth-91`
- Example 92: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.2.0/24 -oA outputs/stealth-92`
- Example 93: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.3.0/24 -oA outputs/stealth-93`
- Example 94: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.4.0/24 -oA outputs/stealth-94`
- Example 95: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.5.0/24 -oA outputs/stealth-95`
- Example 96: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.6.0/24 -oA outputs/stealth-96`
- Example 97: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.7.0/24 -oA outputs/stealth-97`
- Example 98: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.8.0/24 -oA outputs/stealth-98`
- Example 99: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.9.0/24 -oA outputs/stealth-99`
- Example 100: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.0.0/24 -oA outputs/stealth-100`
- Example 101: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.1.0/24 -oA outputs/stealth-101`
- Example 102: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.2.0/24 -oA outputs/stealth-102`
- Example 103: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.3.0/24 -oA outputs/stealth-103`
- Example 104: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.4.0/24 -oA outputs/stealth-104`
- Example 105: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.5.0/24 -oA outputs/stealth-105`
- Example 106: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.6.0/24 -oA outputs/stealth-106`
- Example 107: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.7.0/24 -oA outputs/stealth-107`
- Example 108: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.8.0/24 -oA outputs/stealth-108`
- Example 109: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.9.0/24 -oA outputs/stealth-109`
- Example 110: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.0.0/24 -oA outputs/stealth-110`
- Example 111: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.1.0/24 -oA outputs/stealth-111`
- Example 112: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.2.0/24 -oA outputs/stealth-112`
- Example 113: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.3.0/24 -oA outputs/stealth-113`
- Example 114: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.4.0/24 -oA outputs/stealth-114`
- Example 115: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.5.0/24 -oA outputs/stealth-115`
- Example 116: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.6.0/24 -oA outputs/stealth-116`
- Example 117: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.7.0/24 -oA outputs/stealth-117`
- Example 118: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.8.0/24 -oA outputs/stealth-118`
- Example 119: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.9.0/24 -oA outputs/stealth-119`
- Example 120: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.0.0/24 -oA outputs/stealth-120`
- Example 121: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.1.0/24 -oA outputs/stealth-121`
- Example 122: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.2.0/24 -oA outputs/stealth-122`
- Example 123: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.3.0/24 -oA outputs/stealth-123`
- Example 124: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.4.0/24 -oA outputs/stealth-124`
- Example 125: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.5.0/24 -oA outputs/stealth-125`
- Example 126: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.6.0/24 -oA outputs/stealth-126`
- Example 127: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.7.0/24 -oA outputs/stealth-127`
- Example 128: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.8.0/24 -oA outputs/stealth-128`
- Example 129: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.9.0/24 -oA outputs/stealth-129`
- Example 130: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.0.0/24 -oA outputs/stealth-130`
- Example 131: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.1.0/24 -oA outputs/stealth-131`
- Example 132: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.2.0/24 -oA outputs/stealth-132`
- Example 133: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.3.0/24 -oA outputs/stealth-133`
- Example 134: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.4.0/24 -oA outputs/stealth-134`
- Example 135: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.5.0/24 -oA outputs/stealth-135`
- Example 136: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.6.0/24 -oA outputs/stealth-136`
- Example 137: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.7.0/24 -oA outputs/stealth-137`
- Example 138: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.8.0/24 -oA outputs/stealth-138`
- Example 139: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.9.0/24 -oA outputs/stealth-139`
- Example 140: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.0.0/24 -oA outputs/stealth-140`
- Example 141: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.1.0/24 -oA outputs/stealth-141`
- Example 142: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.2.0/24 -oA outputs/stealth-142`
- Example 143: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.3.0/24 -oA outputs/stealth-143`
- Example 144: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.4.0/24 -oA outputs/stealth-144`
- Example 145: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.5.0/24 -oA outputs/stealth-145`
- Example 146: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.6.0/24 -oA outputs/stealth-146`
- Example 147: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.7.0/24 -oA outputs/stealth-147`
- Example 148: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.8.0/24 -oA outputs/stealth-148`
- Example 149: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.9.0/24 -oA outputs/stealth-149`
- Example 150: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.0.0/24 -oA outputs/stealth-150`
- Example 151: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.1.0/24 -oA outputs/stealth-151`
- Example 152: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.2.0/24 -oA outputs/stealth-152`
- Example 153: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.3.0/24 -oA outputs/stealth-153`
- Example 154: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.4.0/24 -oA outputs/stealth-154`
- Example 155: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.5.0/24 -oA outputs/stealth-155`
- Example 156: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.6.0/24 -oA outputs/stealth-156`
- Example 157: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.7.0/24 -oA outputs/stealth-157`
- Example 158: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.8.0/24 -oA outputs/stealth-158`
- Example 159: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.9.0/24 -oA outputs/stealth-159`
- Example 160: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.0.0/24 -oA outputs/stealth-160`
- Example 161: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.1.0/24 -oA outputs/stealth-161`
- Example 162: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.2.0/24 -oA outputs/stealth-162`
- Example 163: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.3.0/24 -oA outputs/stealth-163`
- Example 164: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.4.0/24 -oA outputs/stealth-164`
- Example 165: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.5.0/24 -oA outputs/stealth-165`
- Example 166: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.6.0/24 -oA outputs/stealth-166`
- Example 167: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.7.0/24 -oA outputs/stealth-167`
- Example 168: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.8.0/24 -oA outputs/stealth-168`
- Example 169: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.9.0/24 -oA outputs/stealth-169`
- Example 170: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.0.0/24 -oA outputs/stealth-170`
- Example 171: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.1.0/24 -oA outputs/stealth-171`
- Example 172: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.2.0/24 -oA outputs/stealth-172`
- Example 173: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.3.0/24 -oA outputs/stealth-173`
- Example 174: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.4.0/24 -oA outputs/stealth-174`
- Example 175: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.5.0/24 -oA outputs/stealth-175`
- Example 176: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.6.0/24 -oA outputs/stealth-176`
- Example 177: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.7.0/24 -oA outputs/stealth-177`
- Example 178: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.8.0/24 -oA outputs/stealth-178`
- Example 179: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.9.0/24 -oA outputs/stealth-179`
- Example 180: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.0.0/24 -oA outputs/stealth-180`
- Example 181: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.1.0/24 -oA outputs/stealth-181`
- Example 182: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.2.0/24 -oA outputs/stealth-182`
- Example 183: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.3.0/24 -oA outputs/stealth-183`
- Example 184: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.4.0/24 -oA outputs/stealth-184`
- Example 185: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.5.0/24 -oA outputs/stealth-185`
- Example 186: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.6.0/24 -oA outputs/stealth-186`
- Example 187: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.7.0/24 -oA outputs/stealth-187`
- Example 188: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.8.0/24 -oA outputs/stealth-188`
- Example 189: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.9.0/24 -oA outputs/stealth-189`
- Example 190: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.0.0/24 -oA outputs/stealth-190`
- Example 191: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.1.0/24 -oA outputs/stealth-191`
- Example 192: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.2.0/24 -oA outputs/stealth-192`
- Example 193: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.3.0/24 -oA outputs/stealth-193`
- Example 194: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.4.0/24 -oA outputs/stealth-194`
- Example 195: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.5.0/24 -oA outputs/stealth-195`
- Example 196: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.6.0/24 -oA outputs/stealth-196`
- Example 197: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.7.0/24 -oA outputs/stealth-197`
- Example 198: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.8.0/24 -oA outputs/stealth-198`
- Example 199: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.9.0/24 -oA outputs/stealth-199`
- Example 200: `sudo nmap -sS -T2 --max-rate 50 --randomize-hosts 10.0.0.0/24 -oA outputs/stealth-200`
```
## Reflection Questions
1. Which stealth option feels safest to you?
2. How do timing templates change scan visibility?
3. When should you avoid advanced evasion options?
4. How do Days 18 and 19 support safer stealth scanning?
5. What documentation would you provide to a client after a stealth scan?

## Next Steps
Tomorrow (Day 21) you will learn about firewall and IDS evasion.

**Congratulations on completing Day 20!** You now understand how to reduce scan noise responsibly.
