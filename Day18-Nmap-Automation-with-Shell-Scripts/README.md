# Day 18 - Nmap Automation with Shell Scripts

## Day Overview
Welcome to Day 18! Today you will learn how to automate Nmap scans using simple shell scripts. Automation helps you run repeatable scans, reduce mistakes, and create consistent outputs for reporting and later analysis.

## Learning Objectives
By completing Day 18, you will be able to:
- Understand when automation is useful in Nmap workflows
- Write basic shell scripts that run Nmap safely
- Use variables, loops, and input files for multiple targets
- Save organized scan outputs for later parsing
- Add guardrails to prevent accidental misuse

## Why Automate Nmap?

### Manual Scanning vs Automation

```text
Manual Scanning:
- Type commands repeatedly
- Easy to make typos
- Hard to keep results consistent
- Slow when scanning many targets

Automated Scanning:
- One command runs many scans
- Consistent options every time
- Outputs stored in a clear structure
- Easier to repeat or schedule
```

## Before You Start

### Prerequisites
- Nmap installed and working
- Basic terminal knowledge
- A safe and authorized environment to scan

### Folder Structure Recommendation
Create a simple folder layout for outputs:

```text
day18-automation/
  scripts/
  outputs/
  targets/
```

## Your First Automation Script

### 1. Simple Single-Target Scan
Create a script called `basic_scan.sh`:

```bash
#!/usr/bin/env bash
set -euo pipefail

TARGET="${1:-}"

if [[ -z "$TARGET" ]]; then
  echo "Usage: $0 <target>"
  exit 1
fi

OUTDIR="outputs"
mkdir -p "$OUTDIR"

echo "Scanning $TARGET..."
nmap -sV -oA "$OUTDIR/scan-$TARGET" "$TARGET"
echo "Done. Results saved to $OUTDIR/scan-$TARGET.*"
```

Run it:

```bash
chmod +x basic_scan.sh
./basic_scan.sh 192.168.1.10
```

### What This Script Does
- Accepts a target from the command line
- Creates an output folder if needed
- Runs a version scan
- Saves output in normal, XML, and grepable formats

## Automating Multiple Targets

### 2. Scan a List of Targets
Create a file in `targets/targets.txt`:

```text
192.168.1.10
192.168.1.11
192.168.1.12
```

Now create `multi_scan.sh`:

```bash
#!/usr/bin/env bash
set -euo pipefail

TARGET_FILE="${1:-targets/targets.txt}"
OUTDIR="outputs"
mkdir -p "$OUTDIR"

if [[ ! -f "$TARGET_FILE" ]]; then
  echo "Target file not found: $TARGET_FILE"
  exit 1
fi

while IFS= read -r TARGET; do
  [[ -z "$TARGET" ]] && continue
  echo "Scanning $TARGET..."
  nmap -sV -oA "$OUTDIR/scan-$TARGET" "$TARGET"
done < "$TARGET_FILE"

echo "All scans completed."
```

Run it:

```bash
chmod +x multi_scan.sh
./multi_scan.sh targets/targets.txt
```

## Adding Common Safety Checks

### 3. Confirm Permission and Scope
Add a confirmation step to reduce mistakes:

```bash
read -rp "Do you have permission to scan these targets? (yes/no): " ANSWER
if [[ "$ANSWER" != "yes" ]]; then
  echo "Scan canceled."
  exit 1
fi
```

### 4. Limit Speed for Safer Scans
Use timing templates and rate control:

```bash
nmap -T3 --max-rate 100 -sV -oA "$OUTDIR/scan-$TARGET" "$TARGET"
```

## Automating Common Scan Types

### 5. Fast Discovery + Focused Scan
First discover hosts, then scan them:

```bash
#!/usr/bin/env bash
set -euo pipefail

RANGE="${1:-192.168.1.0/24}"
OUTDIR="outputs"
mkdir -p "$OUTDIR"

DISCOVERY="$OUTDIR/host-discovery"
echo "Discovering hosts in $RANGE..."
nmap -sn "$RANGE" -oA "$DISCOVERY"

echo "Scanning discovered hosts..."
awk '/Up$/{print $2}' "$DISCOVERY.gnmap" > "$OUTDIR/alive.txt"
nmap -iL "$OUTDIR/alive.txt" -sV -oA "$OUTDIR/service-scan"
```

### What This Workflow Does
- Runs a ping sweep to find live hosts
- Extracts IPs from the grepable output
- Runs a service scan only on live hosts

## Organizing Output and Logs

### Use Date-Based Folders

```bash
RUN_DIR="outputs/$(date +%Y%m%d-%H%M%S)"
mkdir -p "$RUN_DIR"
nmap -sV -oA "$RUN_DIR/scan-$TARGET" "$TARGET"
```

### Keep a Simple Log

```bash
echo "$(date): Scanned $TARGET with -sV" >> "$RUN_DIR/scan.log"
```

## Adding Basic Error Handling

### Detect Missing Tools

```bash
if ! command -v nmap >/dev/null 2>&1; then
  echo "Nmap is not installed or not in PATH."
  exit 1
fi
```

### Skip Invalid Targets

```bash
if [[ ! "$TARGET" =~ ^[0-9]+\.[0-9]+\.[0-9]+\.[0-9]+$ ]]; then
  echo "Skipping invalid target: $TARGET"
  continue
fi
```

## Practical Automation Scenarios

### Scenario 1: Weekly Internal Scan

```bash
#!/usr/bin/env bash
set -euo pipefail

TARGET_FILE="targets/weekly.txt"
RUN_DIR="outputs/weekly-$(date +%Y%m%d)"
mkdir -p "$RUN_DIR"

nmap -iL "$TARGET_FILE" -sV -oA "$RUN_DIR/weekly-scan"
echo "Weekly scan complete: $RUN_DIR"
```

### Scenario 2: Quick Web Services Check

```bash
#!/usr/bin/env bash
set -euo pipefail

TARGET_FILE="targets/web.txt"
RUN_DIR="outputs/web-$(date +%Y%m%d)"
mkdir -p "$RUN_DIR"

nmap -iL "$TARGET_FILE" -p 80,443,8080,8443 -sV --script http-title -oA "$RUN_DIR/web-scan"
```

### Scenario 3: Scan and Parse for Open Ports

```bash
#!/usr/bin/env bash
set -euo pipefail

TARGET="${1:-}"
OUTDIR="outputs"
mkdir -p "$OUTDIR"

if [[ -z "$TARGET" ]]; then
  echo "Usage: $0 <target>"
  exit 1
fi

nmap -oG "$OUTDIR/scan-$TARGET.gnmap" "$TARGET"
grep "/open/" "$OUTDIR/scan-$TARGET.gnmap" > "$OUTDIR/open-ports-$TARGET.txt"
echo "Open ports saved to $OUTDIR/open-ports-$TARGET.txt"
```

## Troubleshooting Automation Issues

### Script Fails with Permission Errors
```text
Fix:
- Ensure the script is executable: chmod +x script.sh
- Run with appropriate privileges if required (sudo)
```

### Script Takes Too Long
```text
Fix:
- Use faster timing: -T4
- Reduce ports: -p 22,80,443
- Limit rate: --max-rate 100
```

### Output Files Overwrite Each Other
```text
Fix:
- Use a date-based output directory
- Add target names to output files
```

## Best Practices
- Always verify permission before scanning
- Keep scripts simple and readable
- Store outputs with timestamps
- Start with safe scan options
- Document the options you used

## Next Steps
Tomorrow (Day 19) you will learn:
- Scanning multiple targets and ranges
- Efficient target management
- Performance strategies for large networks

## Reflection Questions
1. Which parts of scanning are most repetitive in your workflow?
2. How can automation reduce mistakes in your scans?
3. What guardrails would you add to your scripts?
4. How would you organize outputs for long-term reporting?

## Need Help?
- Check Nmap help: `nmap --help`
- Learn script basics: `man bash`
- Practice in a lab environment first

**Congratulations on completing Day 18!** Automation will save you time and keep your scans consistent and repeatable.
