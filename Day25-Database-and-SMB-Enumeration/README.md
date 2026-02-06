# Day 25 - Database and SMB Enumeration

## Day Overview
Welcome to Day 25! Today you will learn how to enumerate database services and SMB shares using Nmap, from basic discovery to advanced, safe enumeration techniques.
This guide is detailed and focused only on database and SMB enumeration, with clear explanations and structured concepts.

## Learning Objectives
By completing Day 25, you will be able to:
- Identify common database and SMB ports
- Run safe, basic enumeration scans
- Use Nmap scripts to gather service details
- Interpret results and avoid common pitfalls
- Build a structured workflow for database and SMB enumeration

## Important Ethics Note
Only enumerate systems you own or have explicit permission to test.
Database and SMB services often expose sensitive data if misconfigured.

## Database Enumeration Basics
Databases run on well-known ports, but sometimes use custom ports.
Common database ports:
- 3306 (MySQL/MariaDB)
- 5432 (PostgreSQL)
- 1433 (Microsoft SQL Server)
- 1521 (Oracle)
- 27017 (MongoDB)
- 6379 (Redis)

## SMB Enumeration Basics
SMB is used for Windows file sharing and domain services.
Common SMB-related ports:
- 445 (SMB over TCP)
- 139 (NetBIOS Session Service)
- 137 (NetBIOS Name Service)
- 138 (NetBIOS Datagram Service)

## Start with Port Discovery
Discover database and SMB services first:
```bash
nmap -p 137,138,139,445,1433,1521,3306,5432,27017,6379 target
```
Use version detection to identify database engines:
```bash
nmap -sV -p 1433,1521,3306,5432,27017,6379 target
```

## Database Version Detection
Version detection helps identify:
- Database engine type
- Version numbers for patch analysis
- Authentication requirements

Example:
```bash
nmap -sV -p 3306 target
```

## SMB Version Detection
SMB versions matter for security and compatibility.
Use Nmap scripts to detect SMB version:
```bash
nmap -p 445 --script smb-protocols target
```

## Common Database Nmap Scripts
Useful safe scripts for enumeration:
- `mysql-info`
- `pgsql-info`
- `ms-sql-info`
- `oracle-tns-version`
- `mongodb-info`
- `redis-info`

Examples:
```bash
nmap -p 3306 --script mysql-info target
nmap -p 5432 --script pgsql-info target
nmap -p 1433 --script ms-sql-info target
nmap -p 1521 --script oracle-tns-version target
nmap -p 27017 --script mongodb-info target
nmap -p 6379 --script redis-info target
```

## Common SMB Nmap Scripts
Safe and common SMB scripts:
- `smb-os-discovery`
- `smb-enum-shares`
- `smb-enum-users`
- `smb-security-mode`
- `smb2-security-mode`
- `smb2-time`

Examples:
```bash
nmap -p 445 --script smb-os-discovery target
nmap -p 445 --script smb-enum-shares target
nmap -p 445 --script smb-enum-users target
```

## Database Enumeration Concepts
Key concepts in database enumeration:
- Service discovery (what DB engine is running)
- Version detection (is it outdated)
- Authentication mode (password, no-auth, or integrated)
- Exposure (public vs internal services)

## SMB Enumeration Concepts
Key SMB concepts:
- Workgroup or domain name
- Server OS details
- Share lists and permissions
- SMB signing requirements

## Enumerating SMB Shares
```bash
nmap -p 445 --script smb-enum-shares target
```
Look for:
- Anonymous access
- Sensitive share names
- Writable shares

## Enumerating SMB Users
```bash
nmap -p 445 --script smb-enum-users target
```
Use only with permission; user enumeration can be sensitive.

## SMB OS Discovery
```bash
nmap -p 445 --script smb-os-discovery target
```
This reveals OS, domain/workgroup, and system time.

## SMB Security Mode
Check whether SMB signing is required:
```bash
nmap -p 445 --script smb-security-mode,smb2-security-mode target
```

## Database Authentication Checks
Some scripts can check for authentication modes without brute force.
Example (MySQL):
```bash
nmap -p 3306 --script mysql-info target
```

## Database-Specific Highlights
### MySQL/MariaDB
- Default port: 3306
- Common script: `mysql-info`
- Possible issues: default accounts, outdated versions

### PostgreSQL
- Default port: 5432
- Common script: `pgsql-info`
- Possible issues: weak auth or exposure to public networks

### Microsoft SQL Server
- Default port: 1433
- Common script: `ms-sql-info`
- Extra scripts: `ms-sql-ntlm-info`

### Oracle
- Default port: 1521
- Common script: `oracle-tns-version`

### MongoDB
- Default port: 27017
- Common script: `mongodb-info`
- Check for no-auth setups with extreme caution

### Redis
- Default port: 6379
- Common script: `redis-info`

## Combining Database Scripts
Run a safe script bundle on multiple DB ports:
```bash
nmap -sV -p 1433,1521,3306,5432,27017,6379 --script mysql-info,pgsql-info,ms-sql-info,oracle-tns-version,mongodb-info,redis-info target
```

## SMB and Database Enumeration at Scale
Use a target list to scan multiple hosts:
```bash
nmap -p 137,138,139,445,1433,1521,3306,5432,27017,6379 -sV -iL targets.txt -oA outputs/db-smb-batch
```

## Output Management
Always use `-oA` for parsing and reporting:
```bash
nmap -p 445 -sV --script smb-os-discovery -oA outputs/smb target
```

## Parsing Results (Day 17 Skills Applied)
Extract SMB hosts:
```bash
grep "445/open" outputs/db-smb-batch.gnmap
```
Extract DB hosts:
```bash
grep "3306/open" outputs/db-smb-batch.gnmap
```

## Handling Non-Standard Ports
Databases sometimes run on custom ports.
Use version detection across a range if needed:
```bash
nmap -sV -p 1-10000 target
```

## SMB Over NetBIOS
Older systems may still expose NetBIOS services:
```bash
nmap -sU -p 137,138 target
nmap -sT -p 139 target
```

## Using Safe Script Categories
Use `safe` and `default` categories when possible:
```bash
nmap -p 445 --script safe target
```

## Risky Scripts Warning
Some scripts attempt brute force or intrusive checks.
Only run them with explicit permission and clear scope.

## SMB Signing and Security Implications
SMB signing prevents certain attacks but can affect performance.
Detection:
```bash
nmap -p 445 --script smb2-security-mode target
```

## SMB Time and Domain Information
```bash
nmap -p 445 --script smb2-time,smb-os-discovery target
```

## Database Exposure Risks
- Publicly accessible databases are high risk
- Weak or default credentials can lead to compromise
- Unencrypted DB traffic can leak data

## SMB Exposure Risks
- Anonymous share access
- Weak SMB signing or outdated protocols
- Legacy SMBv1 exposure

## Advanced Enumeration Concepts (Theory)
### Authenticated Enumeration
Some scripts support credentials for deeper checks.
Always store credentials securely and only use with permission.

### Service Banner Limitations
Banners can be misleading or spoofed.
Confirm with multiple methods when possible.

### Network Segmentation
Database and SMB services should be segmented from public access.
Enumeration can reveal segmentation gaps.

### Encryption and Secure Channels
Look for TLS support on database connections where applicable.

## Example Workflows 
### Example A: Basic SMB Enumeration
```bash
nmap -p 445 --script smb-os-discovery,smb-enum-shares target -oA outputs/smb-basic
```

### Example B: Database Recon Bundle
```bash
nmap -sV -p 1433,3306,5432,1521,27017,6379 --script ms-sql-info,mysql-info,pgsql-info,oracle-tns-version,mongodb-info,redis-info target -oA outputs/db-bundle
```

### Example C: SMB + DB Combined Scan
```bash
nmap -sV -p 137,138,139,445,1433,1521,3306,5432,27017,6379 --script smb-os-discovery,mysql-info,pgsql-info,ms-sql-info target -oA outputs/db-smb-combined
```

## Interpreting Results
Look for:
- Open SMB ports with share listings
- Database banners and version details
- Weak SMB signing or legacy protocols
- Unexpected database services on internal networks

## Common Misconfigurations
- SMB shares accessible anonymously
- SMBv1 still enabled
- Databases exposed to public networks
- Outdated database versions

## Defensive Use Cases
- Inventory internal database servers
- Check SMB share exposure
- Validate segmentation and firewall rules
- Confirm patching levels

## Limitations of Nmap for DB/SMB
- Nmap does not perform full database security audits
- Scripts provide surface-level enumeration
- Deep checks usually require authenticated tools

## Best Practices
- Start with safe scripts
- Keep scope minimal
- Log outputs and options
- Confirm findings with additional tools if needed

## Reflection Questions
1. Why is SMB signing important?
2. What risks exist when databases are public?
3. How do you balance scan coverage with safety?
4. What limitations does Nmap have for DB enumeration?

## Next Steps
Tomorrow (Day 26) you will learn about cloud and external perimeter scanning.

**Congratulations on completing Day 25!** You now have a structured approach to database and SMB enumeration.

