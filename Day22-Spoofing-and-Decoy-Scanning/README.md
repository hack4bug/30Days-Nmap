# Day 22 - Spoofing and Decoy Scanning

## Day Overview
Welcome to Day 22! Today you will learn what spoofing and decoy scanning mean in Nmap and how they are used in controlled, authorized environments.
This is an advanced topic, but we will keep it beginner-friendly and focused on safe, permitted use.

This lesson builds on the last three days:
- Day 19: Scanning multiple targets and ranges
- Day 20: Stealth and evasion techniques
- Day 21: Firewall and IDS evasion

## Learning Objectives
By completing Day 22, you will be able to:
- Understand what IP spoofing and decoys are
- Use Nmap decoy options safely in a lab
- Recognize the limits and risks of spoofing
- Combine decoys with conservative scan options
- Document and report your scan settings clearly

## Important Ethics Note
Spoofing and decoy scanning must only be used with explicit permission.
Never attempt to hide scanning activity on networks you do not own or control.

## What Is Spoofing?
Spoofing means forging the source IP of packets to make them appear as if they came from another system.
In most real networks, spoofed packets are blocked by routers or firewalls.

## What Is Decoy Scanning?
Decoy scanning adds extra fake source IPs to a scan to hide the true origin among decoys.
It does not make you invisible. It only makes attribution harder in logs.

## How Detection Still Happens
- IDS/IPS can still see unusual traffic patterns
- Firewalls can drop spoofed or suspicious packets
- Logs can still capture your real IP in other ways

## Day 19 Recap: Scope Control
Use small, authorized scopes when testing advanced options like decoys.

## Day 20 Recap: Timing and Noise Reduction
Slower scans with limited ports reduce the chance of alerting systems.

## Day 21 Recap: Evasion Limitations
Modern defenses can still detect stealth scans, so document and communicate.

## Core Nmap Options for Decoys

### 1. Random Decoys
```bash
sudo nmap -sS -D RND:5 target
```
This adds 5 random decoy IPs to the scan.

### 2. Specific Decoy IPs
```bash
sudo nmap -sS -D 192.0.2.10,198.51.100.20,203.0.113.30 target
```
Use documentation/test IPs in labs to avoid harming real hosts.

### 3. Use Your Real IP Last
```bash
sudo nmap -sS -D RND:3,ME target
```
The `ME` keyword inserts your real IP in the decoy list.

### 4. Combine Decoys with Safer Timing
```bash
sudo nmap -sS -T2 --max-rate 50 -D RND:5 target
```

## Spoofing Options in Nmap
Nmap can set a fake source address for certain scans, but it is limited and often blocked.

### Source Address Spoofing (Advanced)
```bash
sudo nmap -S 203.0.113.50 -e eth0 target
```
Notes:
- Requires raw packet privileges
- Many networks drop spoofed packets
- Responses may go to the spoofed IP, not you

## Why Spoofing Often Fails in Real Networks
- ISPs and routers filter spoofed traffic
- Replies go to the spoofed address, so you miss responses
- Modern networks log unusual packet behavior

## Safe Beginner Strategy
- Focus on decoys, not spoofing, in learning labs
- Use small scopes and limited port lists
- Keep timing conservative
- Document all options and outputs

## Practical Examples

### Example 1: Conservative Decoy Scan
```bash
sudo nmap -sS -T2 --max-rate 30 -D RND:3 -p 22,80,443 target -oA outputs/decoy-basic
```

### Example 2: Decoy Scan on a Small Range
```bash
sudo nmap -sS -T2 --randomize-hosts -D RND:5 192.168.1.0/29 -oA outputs/decoy-range
```

### Example 3: Discovery First, Then Decoy Scan
```bash
nmap -sn 192.168.1.0/29 -oA outputs/decoy-discovery
awk '/Up$/{print $2}' outputs/decoy-discovery.gnmap > outputs/decoy-live.txt
sudo nmap -sS -T2 -D RND:3 -iL outputs/decoy-live.txt -oA outputs/decoy-live-scan
```

## Combining with Automation (Day 18)
```bash
#!/usr/bin/env bash
set -euo pipefail
TARGET_FILE="${1:-targets.txt}"
RUN_DIR="outputs/$(date +%Y%m%d-%H%M%S)"
mkdir -p "$RUN_DIR"
sudo nmap -sS -T2 --max-rate 50 -D RND:3 -iL "$TARGET_FILE" -oA "$RUN_DIR/decoy"
```

## Logging and Reporting
Always record:
- Time and date
- Target scope
- Decoy settings used
- Output file paths

## Beginner-Friendly Checklists
### Pre-Scan Checklist
- I have written permission
- I confirmed the target scope
- I selected conservative timing
- I prepared output folders
- I documented the scan plan

### Post-Scan Checklist
- I saved all outputs
- I reviewed results for open ports
- I documented anomalies and alerts
- I stored logs safely

## Troubleshooting

### No Responses
```text
Fix:
- Decoys can increase packet loss
- Reduce decoys or increase timeouts
- Verify basic connectivity
```

### Scan Too Slow
```text
Fix:
- Reduce decoy count
- Increase timing slightly
- Limit the port list
```

### Unclear Results
```text
Fix:
- Remove advanced options and re-test
- Use simple scans to verify baseline results
```

## Mini Glossary
- Spoofing: Forging a source address
- Decoy: Fake source IP used to hide the true source
- ME: Nmap keyword to insert your real IP in decoy list

## Step-by-Step Lab: Safe Decoy Scan
### Step 1: List Targets
```bash
nmap -sL 192.168.1.0/29
```
### Step 2: Host Discovery
```bash
nmap -sn 192.168.1.0/29 -oA outputs/lab-discovery
```
### Step 3: Decoy Scan Live Hosts
```bash
awk '/Up$/{print $2}' outputs/lab-discovery.gnmap > outputs/lab-live.txt
sudo nmap -sS -T2 -D RND:3 -iL outputs/lab-live.txt -oA outputs/lab-decoy
```
### Step 4: Review Results
```bash
grep "/open/" outputs/lab-decoy.gnmap
```

## Limited Practice Section
Use these few drills to reinforce concepts without overloading repetition.

### Drill 1: Decoy Practice
Goal: Use decoys safely with a small, authorized scope.
1. Choose a /29 range or short target list.
2. Run discovery to find live hosts.
3. Scan with `-sS -T2` and `-D RND:3`.
4. Limit ports to 22, 80, 443 for speed.
5. Save outputs with `-oA`.
6. Review `.gnmap` for open ports.
7. Document the scan settings and results.

### Drill 2: Decoy Practice
Goal: Use decoys safely with a small, authorized scope.
1. Choose a /29 range or short target list.
2. Run discovery to find live hosts.
3. Scan with `-sS -T2` and `-D RND:3`.
4. Limit ports to 22, 80, 443 for speed.
5. Save outputs with `-oA`.
6. Review `.gnmap` for open ports.
7. Document the scan settings and results.

### Drill 3: Decoy Practice
Goal: Use decoys safely with a small, authorized scope.
1. Choose a /29 range or short target list.
2. Run discovery to find live hosts.
3. Scan with `-sS -T2` and `-D RND:3`.
4. Limit ports to 22, 80, 443 for speed.
5. Save outputs with `-oA`.
6. Review `.gnmap` for open ports.
7. Document the scan settings and results.

### Drill 4: Decoy Practice
Goal: Use decoys safely with a small, authorized scope.
1. Choose a /29 range or short target list.
2. Run discovery to find live hosts.
3. Scan with `-sS -T2` and `-D RND:3`.
4. Limit ports to 22, 80, 443 for speed.
5. Save outputs with `-oA`.
6. Review `.gnmap` for open ports.
7. Document the scan settings and results.

### Drill 5: Decoy Practice
Goal: Use decoys safely with a small, authorized scope.
1. Choose a /29 range or short target list.
2. Run discovery to find live hosts.
3. Scan with `-sS -T2` and `-D RND:3`.
4. Limit ports to 22, 80, 443 for speed.
5. Save outputs with `-oA`.
6. Review `.gnmap` for open ports.
7. Document the scan settings and results.

### Drill 6: Decoy Practice
Goal: Use decoys safely with a small, authorized scope.
1. Choose a /29 range or short target list.
2. Run discovery to find live hosts.
3. Scan with `-sS -T2` and `-D RND:3`.
4. Limit ports to 22, 80, 443 for speed.
5. Save outputs with `-oA`.
6. Review `.gnmap` for open ports.
7. Document the scan settings and results.

### Drill 7: Decoy Practice
Goal: Use decoys safely with a small, authorized scope.
1. Choose a /29 range or short target list.
2. Run discovery to find live hosts.
3. Scan with `-sS -T2` and `-D RND:3`.
4. Limit ports to 22, 80, 443 for speed.
5. Save outputs with `-oA`.
6. Review `.gnmap` for open ports.
7. Document the scan settings and results.

### Drill 8: Decoy Practice
Goal: Use decoys safely with a small, authorized scope.
1. Choose a /29 range or short target list.
2. Run discovery to find live hosts.
3. Scan with `-sS -T2` and `-D RND:3`.
4. Limit ports to 22, 80, 443 for speed.
5. Save outputs with `-oA`.
6. Review `.gnmap` for open ports.
7. Document the scan settings and results.

### Drill 9: Decoy Practice
Goal: Use decoys safely with a small, authorized scope.
1. Choose a /29 range or short target list.
2. Run discovery to find live hosts.
3. Scan with `-sS -T2` and `-D RND:3`.
4. Limit ports to 22, 80, 443 for speed.
5. Save outputs with `-oA`.
6. Review `.gnmap` for open ports.
7. Document the scan settings and results.

### Drill 10: Decoy Practice
Goal: Use decoys safely with a small, authorized scope.
1. Choose a /29 range or short target list.
2. Run discovery to find live hosts.
3. Scan with `-sS -T2` and `-D RND:3`.
4. Limit ports to 22, 80, 443 for speed.
5. Save outputs with `-oA`.
6. Review `.gnmap` for open ports.
7. Document the scan settings and results.

### Drill 11: Decoy Practice
Goal: Use decoys safely with a small, authorized scope.
1. Choose a /29 range or short target list.
2. Run discovery to find live hosts.
3. Scan with `-sS -T2` and `-D RND:3`.
4. Limit ports to 22, 80, 443 for speed.
5. Save outputs with `-oA`.
6. Review `.gnmap` for open ports.
7. Document the scan settings and results.

### Drill 12: Decoy Practice
Goal: Use decoys safely with a small, authorized scope.
1. Choose a /29 range or short target list.
2. Run discovery to find live hosts.
3. Scan with `-sS -T2` and `-D RND:3`.
4. Limit ports to 22, 80, 443 for speed.
5. Save outputs with `-oA`.
6. Review `.gnmap` for open ports.
7. Document the scan settings and results.

```powershell
## Command Reference (One-Liners)
- Example 1: `sudo nmap -sS -T2 --max-rate 50 -D RND:3 10.2.1.0/24 -oA outputs/decoy-1`
- Example 2: `sudo nmap -sS -T2 --max-rate 50 -D RND:3 10.2.2.0/24 -oA outputs/decoy-2`
- Example 3: `sudo nmap -sS -T2 --max-rate 50 -D RND:3 10.2.3.0/24 -oA outputs/decoy-3`
- Example 4: `sudo nmap -sS -T2 --max-rate 50 -D RND:3 10.2.4.0/24 -oA outputs/decoy-4`
- Example 5: `sudo nmap -sS -T2 --max-rate 50 -D RND:3 10.2.5.0/24 -oA outputs/decoy-5`
- Example 6: `sudo nmap -sS -T2 --max-rate 50 -D RND:3 10.2.6.0/24 -oA outputs/decoy-6`
- Example 7: `sudo nmap -sS -T2 --max-rate 50 -D RND:3 10.2.7.0/24 -oA outputs/decoy-7`
- Example 8: `sudo nmap -sS -T2 --max-rate 50 -D RND:3 10.2.8.0/24 -oA outputs/decoy-8`
- Example 9: `sudo nmap -sS -T2 --max-rate 50 -D RND:3 10.2.9.0/24 -oA outputs/decoy-9`
- Example 10: `sudo nmap -sS -T2 --max-rate 50 -D RND:3 10.2.0.0/24 -oA outputs/decoy-10`
- Example 11: `sudo nmap -sS -T2 --max-rate 50 -D RND:3 10.2.1.0/24 -oA outputs/decoy-11`
- Example 12: `sudo nmap -sS -T2 --max-rate 50 -D RND:3 10.2.2.0/24 -oA outputs/decoy-12`
- Example 13: `sudo nmap -sS -T2 --max-rate 50 -D RND:3 10.2.3.0/24 -oA outputs/decoy-13`
- Example 14: `sudo nmap -sS -T2 --max-rate 50 -D RND:3 10.2.4.0/24 -oA outputs/decoy-14`
- Example 15: `sudo nmap -sS -T2 --max-rate 50 -D RND:3 10.2.5.0/24 -oA outputs/decoy-15`
- Example 16: `sudo nmap -sS -T2 --max-rate 50 -D RND:3 10.2.6.0/24 -oA outputs/decoy-16`
- Example 17: `sudo nmap -sS -T2 --max-rate 50 -D RND:3 10.2.7.0/24 -oA outputs/decoy-17`
- Example 18: `sudo nmap -sS -T2 --max-rate 50 -D RND:3 10.2.8.0/24 -oA outputs/decoy-18`
- Example 19: `sudo nmap -sS -T2 --max-rate 50 -D RND:3 10.2.9.0/24 -oA outputs/decoy-19`
- Example 20: `sudo nmap -sS -T2 --max-rate 50 -D RND:3 10.2.0.0/24 -oA outputs/decoy-20`
- Example 21: `sudo nmap -sS -T2 --max-rate 50 -D RND:3 10.2.1.0/24 -oA outputs/decoy-21`
- Example 22: `sudo nmap -sS -T2 --max-rate 50 -D RND:3 10.2.2.0/24 -oA outputs/decoy-22`
- Example 23: `sudo nmap -sS -T2 --max-rate 50 -D RND:3 10.2.3.0/24 -oA outputs/decoy-23`
- Example 24: `sudo nmap -sS -T2 --max-rate 50 -D RND:3 10.2.4.0/24 -oA outputs/decoy-24`
- Example 25: `sudo nmap -sS -T2 --max-rate 50 -D RND:3 10.2.5.0/24 -oA outputs/decoy-25`
- Example 26: `sudo nmap -sS -T2 --max-rate 50 -D RND:3 10.2.6.0/24 -oA outputs/decoy-26`
- Example 27: `sudo nmap -sS -T2 --max-rate 50 -D RND:3 10.2.7.0/24 -oA outputs/decoy-27`
- Example 28: `sudo nmap -sS -T2 --max-rate 50 -D RND:3 10.2.8.0/24 -oA outputs/decoy-28`
- Example 29: `sudo nmap -sS -T2 --max-rate 50 -D RND:3 10.2.9.0/24 -oA outputs/decoy-29`
- Example 30: `sudo nmap -sS -T2 --max-rate 50 -D RND:3 10.2.0.0/24 -oA outputs/decoy-30`
- Example 31: `sudo nmap -sS -T2 --max-rate 50 -D RND:3 10.2.1.0/24 -oA outputs/decoy-31`
- Example 32: `sudo nmap -sS -T2 --max-rate 50 -D RND:3 10.2.2.0/24 -oA outputs/decoy-32`
- Example 33: `sudo nmap -sS -T2 --max-rate 50 -D RND:3 10.2.3.0/24 -oA outputs/decoy-33`
- Example 34: `sudo nmap -sS -T2 --max-rate 50 -D RND:3 10.2.4.0/24 -oA outputs/decoy-34`
- Example 35: `sudo nmap -sS -T2 --max-rate 50 -D RND:3 10.2.5.0/24 -oA outputs/decoy-35`
- Example 36: `sudo nmap -sS -T2 --max-rate 50 -D RND:3 10.2.6.0/24 -oA outputs/decoy-36`
- Example 37: `sudo nmap -sS -T2 --max-rate 50 -D RND:3 10.2.7.0/24 -oA outputs/decoy-37`
- Example 38: `sudo nmap -sS -T2 --max-rate 50 -D RND:3 10.2.8.0/24 -oA outputs/decoy-38`
- Example 39: `sudo nmap -sS -T2 --max-rate 50 -D RND:3 10.2.9.0/24 -oA outputs/decoy-39`
- Example 40: `sudo nmap -sS -T2 --max-rate 50 -D RND:3 10.2.0.0/24 -oA outputs/decoy-40`
- Example 41: `sudo nmap -sS -T2 --max-rate 50 -D RND:3 10.2.1.0/24 -oA outputs/decoy-41`
- Example 42: `sudo nmap -sS -T2 --max-rate 50 -D RND:3 10.2.2.0/24 -oA outputs/decoy-42`
- Example 43: `sudo nmap -sS -T2 --max-rate 50 -D RND:3 10.2.3.0/24 -oA outputs/decoy-43`
- Example 44: `sudo nmap -sS -T2 --max-rate 50 -D RND:3 10.2.4.0/24 -oA outputs/decoy-44`
- Example 45: `sudo nmap -sS -T2 --max-rate 50 -D RND:3 10.2.5.0/24 -oA outputs/decoy-45`
- Example 46: `sudo nmap -sS -T2 --max-rate 50 -D RND:3 10.2.6.0/24 -oA outputs/decoy-46`
- Example 47: `sudo nmap -sS -T2 --max-rate 50 -D RND:3 10.2.7.0/24 -oA outputs/decoy-47`
- Example 48: `sudo nmap -sS -T2 --max-rate 50 -D RND:3 10.2.8.0/24 -oA outputs/decoy-48`
- Example 49: `sudo nmap -sS -T2 --max-rate 50 -D RND:3 10.2.9.0/24 -oA outputs/decoy-49`
- Example 50: `sudo nmap -sS -T2 --max-rate 50 -D RND:3 10.2.0.0/24 -oA outputs/decoy-50`
- Example 51: `sudo nmap -sS -T2 --max-rate 50 -D RND:3 10.2.1.0/24 -oA outputs/decoy-51`
- Example 52: `sudo nmap -sS -T2 --max-rate 50 -D RND:3 10.2.2.0/24 -oA outputs/decoy-52`
- Example 53: `sudo nmap -sS -T2 --max-rate 50 -D RND:3 10.2.3.0/24 -oA outputs/decoy-53`
- Example 54: `sudo nmap -sS -T2 --max-rate 50 -D RND:3 10.2.4.0/24 -oA outputs/decoy-54`
- Example 55: `sudo nmap -sS -T2 --max-rate 50 -D RND:3 10.2.5.0/24 -oA outputs/decoy-55`
- Example 56: `sudo nmap -sS -T2 --max-rate 50 -D RND:3 10.2.6.0/24 -oA outputs/decoy-56`
- Example 57: `sudo nmap -sS -T2 --max-rate 50 -D RND:3 10.2.7.0/24 -oA outputs/decoy-57`
- Example 58: `sudo nmap -sS -T2 --max-rate 50 -D RND:3 10.2.8.0/24 -oA outputs/decoy-58`
- Example 59: `sudo nmap -sS -T2 --max-rate 50 -D RND:3 10.2.9.0/24 -oA outputs/decoy-59`
- Example 60: `sudo nmap -sS -T2 --max-rate 50 -D RND:3 10.2.0.0/24 -oA outputs/decoy-60`
- Example 61: `sudo nmap -sS -T2 --max-rate 50 -D RND:3 10.2.1.0/24 -oA outputs/decoy-61`
- Example 62: `sudo nmap -sS -T2 --max-rate 50 -D RND:3 10.2.2.0/24 -oA outputs/decoy-62`
- Example 63: `sudo nmap -sS -T2 --max-rate 50 -D RND:3 10.2.3.0/24 -oA outputs/decoy-63`
- Example 64: `sudo nmap -sS -T2 --max-rate 50 -D RND:3 10.2.4.0/24 -oA outputs/decoy-64`
- Example 65: `sudo nmap -sS -T2 --max-rate 50 -D RND:3 10.2.5.0/24 -oA outputs/decoy-65`
- Example 66: `sudo nmap -sS -T2 --max-rate 50 -D RND:3 10.2.6.0/24 -oA outputs/decoy-66`
- Example 67: `sudo nmap -sS -T2 --max-rate 50 -D RND:3 10.2.7.0/24 -oA outputs/decoy-67`
- Example 68: `sudo nmap -sS -T2 --max-rate 50 -D RND:3 10.2.8.0/24 -oA outputs/decoy-68`
- Example 69: `sudo nmap -sS -T2 --max-rate 50 -D RND:3 10.2.9.0/24 -oA outputs/decoy-69`
- Example 70: `sudo nmap -sS -T2 --max-rate 50 -D RND:3 10.2.0.0/24 -oA outputs/decoy-70`
- Example 71: `sudo nmap -sS -T2 --max-rate 50 -D RND:3 10.2.1.0/24 -oA outputs/decoy-71`
- Example 72: `sudo nmap -sS -T2 --max-rate 50 -D RND:3 10.2.2.0/24 -oA outputs/decoy-72`
- Example 73: `sudo nmap -sS -T2 --max-rate 50 -D RND:3 10.2.3.0/24 -oA outputs/decoy-73`
- Example 74: `sudo nmap -sS -T2 --max-rate 50 -D RND:3 10.2.4.0/24 -oA outputs/decoy-74`
- Example 75: `sudo nmap -sS -T2 --max-rate 50 -D RND:3 10.2.5.0/24 -oA outputs/decoy-75`
- Example 76: `sudo nmap -sS -T2 --max-rate 50 -D RND:3 10.2.6.0/24 -oA outputs/decoy-76`
- Example 77: `sudo nmap -sS -T2 --max-rate 50 -D RND:3 10.2.7.0/24 -oA outputs/decoy-77`
- Example 78: `sudo nmap -sS -T2 --max-rate 50 -D RND:3 10.2.8.0/24 -oA outputs/decoy-78`
- Example 79: `sudo nmap -sS -T2 --max-rate 50 -D RND:3 10.2.9.0/24 -oA outputs/decoy-79`
- Example 80: `sudo nmap -sS -T2 --max-rate 50 -D RND:3 10.2.0.0/24 -oA outputs/decoy-80`
- Example 81: `sudo nmap -sS -T2 --max-rate 50 -D RND:3 10.2.1.0/24 -oA outputs/decoy-81`
- Example 82: `sudo nmap -sS -T2 --max-rate 50 -D RND:3 10.2.2.0/24 -oA outputs/decoy-82`
- Example 83: `sudo nmap -sS -T2 --max-rate 50 -D RND:3 10.2.3.0/24 -oA outputs/decoy-83`
- Example 84: `sudo nmap -sS -T2 --max-rate 50 -D RND:3 10.2.4.0/24 -oA outputs/decoy-84`
- Example 85: `sudo nmap -sS -T2 --max-rate 50 -D RND:3 10.2.5.0/24 -oA outputs/decoy-85`
- Example 86: `sudo nmap -sS -T2 --max-rate 50 -D RND:3 10.2.6.0/24 -oA outputs/decoy-86`
- Example 87: `sudo nmap -sS -T2 --max-rate 50 -D RND:3 10.2.7.0/24 -oA outputs/decoy-87`
- Example 88: `sudo nmap -sS -T2 --max-rate 50 -D RND:3 10.2.8.0/24 -oA outputs/decoy-88`
- Example 89: `sudo nmap -sS -T2 --max-rate 50 -D RND:3 10.2.9.0/24 -oA outputs/decoy-89`
- Example 90: `sudo nmap -sS -T2 --max-rate 50 -D RND:3 10.2.0.0/24 -oA outputs/decoy-90`
- Example 91: `sudo nmap -sS -T2 --max-rate 50 -D RND:3 10.2.1.0/24 -oA outputs/decoy-91`
- Example 92: `sudo nmap -sS -T2 --max-rate 50 -D RND:3 10.2.2.0/24 -oA outputs/decoy-92`
- Example 93: `sudo nmap -sS -T2 --max-rate 50 -D RND:3 10.2.3.0/24 -oA outputs/decoy-93`
- Example 94: `sudo nmap -sS -T2 --max-rate 50 -D RND:3 10.2.4.0/24 -oA outputs/decoy-94`
- Example 95: `sudo nmap -sS -T2 --max-rate 50 -D RND:3 10.2.5.0/24 -oA outputs/decoy-95`
- Example 96: `sudo nmap -sS -T2 --max-rate 50 -D RND:3 10.2.6.0/24 -oA outputs/decoy-96`
- Example 97: `sudo nmap -sS -T2 --max-rate 50 -D RND:3 10.2.7.0/24 -oA outputs/decoy-97`
- Example 98: `sudo nmap -sS -T2 --max-rate 50 -D RND:3 10.2.8.0/24 -oA outputs/decoy-98`
- Example 99: `sudo nmap -sS -T2 --max-rate 50 -D RND:3 10.2.9.0/24 -oA outputs/decoy-99`
- Example 100: `sudo nmap -sS -T2 --max-rate 50 -D RND:3 10.2.0.0/24 -oA outputs/decoy-100`
```

## Reflection Questions
1. Why do decoys not make scans invisible?
2. When would you avoid spoofing entirely?
3. How do Days 20 and 21 influence your decoy scan design?
4. What documentation should you keep for decoy scans?

## Next Steps
Tomorrow (Day 23) you will learn about fragmentation and packet manipulation.

**Congratulations on completing Day 22!** You now understand spoofing and decoy scanning responsibly.
