# Day 14 - Brute-Force and Authentication Scripts

## Day Overview
Welcome to Day 14! Today, we're entering a **critical and sensitive area** of Nmap scripting - Brute-Force and Authentication testing. These scripts test the strength of login systems by trying multiple username/password combinations. **IMPORTANT**: This is powerful but dangerous knowledge - you must use it responsibly and legally.

## Learning Objectives
By completing Day 14, you will be able to:
- Understand what brute-force and authentication scripts do
- Use these scripts responsibly and ethically
- Test authentication systems on your own systems
- Understand different authentication protocols
- Configure and control brute-force attacks

## ⚠️ CRITICAL ETHICAL WARNING ⚠️

### The Golden Rules
Before we even look at commands, understand these rules:

1. **NEVER test systems you don't own or control**
2. **ALWAYS get explicit written permission**
3. **NEVER use default or stolen credentials**
4. **ALWAYS test in isolated environments first**
5. **NEVER cause service disruption or account lockouts**

### Legal Consequences
Unauthorized brute-force attempts can result in:
- Criminal charges (Computer Fraud and Abuse Act)
- Civil lawsuits
- Job termination
- Permanent damage to reputation
- Possible imprisonment

## Understanding Authentication Testing

### What Are Authentication Scripts?
These scripts automate the process of testing login credentials. They're like having a robot that tries thousands of key combinations to find the right one.

### Legitimate Uses (Why Learn This?)

```cmd
Legitimate purposes:
1. Testing YOUR OWN systems for weak passwords
2. Security audits with WRITTEN permission
3. Incident response (checking if accounts compromised)
4. Compliance testing (password policy verification)
5. Educational purposes in LAB environments
```

### How Authentication Scripts Work

```cmd
Basic workflow:
Step 1: Identify authentication service (SSH, FTP, HTTP, etc.)
Step 2: Load username and password lists
Step 3: Try combinations systematically
Step 4: Report successful logins
Step 5: Stop when credentials found or list exhausted
```

## Common Authentication Protocols and Ports

### Protocol Overview

| Protocol | Port | Common Use | Risk Level |
|----------|------|------------|------------|
| SSH | 22 | Secure shell access | High |
| FTP | 21 | File transfer | Medium |
| HTTP/HTTPS | 80/443 | Web login forms | Medium |
| SMB | 445 | Windows file sharing | High |
| RDP | 3389 | Remote desktop | High |
| MySQL | 3306 | Database access | High |
| PostgreSQL | 5432 | Database access | High |
| Telnet | 23 | Remote access (insecure) | High |

### Why Different Protocols Matter
Each protocol has different security features:
- **SSH**: Encrypted, can use keys instead of passwords
- **FTP**: Often unencrypted, passwords sent in plain text
- **HTTP**: Various authentication methods (Basic, Digest, Forms)
- **SMB**: Windows authentication, can lock accounts

## Essential Authentication Scripts

### 1. ssh-brute - SSH Authentication Testing

SSH is one of the most common targets because it provides shell access.

```cmd
# Basic SSH brute-force (USE WITH EXTREME CAUTION)
nmap --script ssh-brute -p 22 target-ip

# With custom wordlists (better practice)
nmap --script ssh-brute -p 22 --script-args userdb=usernames.txt,passdb=passwords.txt target-ip

# Sample output on SUCCESS:
PORT   STATE SERVICE
22/tcp open  ssh
| ssh-brute:
|   Accounts:
|     root:password123 - Valid credentials
|   Statistics:
|_    Performed 60 guesses in 10 seconds

# Sample output on FAILURE:
| ssh-brute:
|   Accounts: No valid accounts found
|   Statistics:
|_    Performed 60 guesses in 10 seconds
```

### 2. http-auth - Web Authentication Testing

Tests various web authentication methods.

```cmd
# Test HTTP Basic/Digest authentication
nmap --script http-auth -p 80,443 target

# Test specific authentication realm
nmap --script http-auth --script-args http-auth.path=/admin target

# Common findings:
| http-auth:
|   Basic Authentication
|     Path: /admin
|     Realm: Admin Area
|     Auth type: Basic
|     Credentials found:
|       admin:admin
|       admin:password
|       admin:123456
```

### 3. smb-brute - Windows SMB Authentication

Tests Windows file sharing authentication.

```cmd
# Test SMB authentication
nmap --script smb-brute -p 445 target

# With domain (for Windows domains)
nmap --script smb-brute -p 445 --script-args smbdomain=DOMAIN target

# Important warning: This can lock Windows accounts!
# Always use account lockout bypass if testing production:
nmap --script smb-brute -p 445 --script-args smbbasic=1 target
```

### 4. ftp-brute - FTP Authentication Testing

Tests FTP server logins.

```cmd
# Test FTP authentication
nmap --script ftp-brute -p 21 target

# Common default credentials tested:
anonymous:anonymous
anonymous:ftp
admin:admin
ftp:ftp
test:test
```

### 5. mysql-brute - MySQL Database Authentication

Tests MySQL database logins.

```cmd
# Test MySQL authentication
nmap --script mysql-brute -p 3306 target

# Common default credentials:
root:(empty password)
root:root
root:password
admin:admin
test:test
```

## Creating Wordlists for Testing

### Why Custom Wordlists Matter
Using the same wordlists as attackers makes you think like them and defend better.

### Building Effective Wordlists

```cmd
# Create a simple custom wordlist
echo "admin" > usernames.txt
echo "administrator" >> usernames.txt
echo "root" >> usernames.txt
echo "user" >> usernames.txt
echo "test" >> usernames.txt

echo "password" > passwords.txt
echo "password123" >> passwords.txt
echo "admin" >> passwords.txt
echo "123456" >> passwords.txt
echo "letmein" >> passwords.txt

# Add company/organization specific terms
echo "companyname2024" >> passwords.txt
echo "Summer2024!" >> passwords.txt
echo "Welcome123" >> passwords.txt
```

### Common Weak Password Patterns

```cmd
# Season + Year
Spring2024, Summer2024, Fall2024, Winter2024

# Company name variations
Company123, Company!, Company2024

# Simple patterns
Password1, Password123, Password!

# Default/Common
admin, password, 123456, qwerty, letmein

# Keyboard walks
qwertyuiop, 1qaz2wsx, zaq12wsx
```

## Safe Testing Configuration

### Rate Limiting and Delays

```cmd
# SLOW testing to avoid detection/lockouts
nmap --script ssh-brute --script-args brute.delay=3,brute.threads=1 target

# What this does:
- 3 second delay between attempts
- Only 1 thread (serial attempts)
- Much less likely to trigger alarms
```

### Account Lockout Prevention

```cmd
# Test one account at a time to avoid lockouts
nmap --script ssh-brute --script-args useraspass=1,brute.firstonly=true target

# What this does:
- Tries username as password only
- Stops after first account found
- Minimal account lockout risk
```

### Timeout Controls

```cmd
# Set maximum time for testing
nmap --script ssh-brute --script-timeout 5m target

# What this does:
- Stops after 5 minutes regardless
- Prevents runaway scripts
- Allows you to control testing windows
```

## Practical Lab Scenarios (SAFE PRACTICE ONLY)

### Scenario 1: Testing Your Own SSH Server

```cmd
# Setup: Install SSH on your local machine
sudo apt install openssh-server

# Test with KNOWN credentials first
echo "testuser" > test-users.txt
echo "testpass" > test-passwords.txt

# Run test (on your own machine!)
nmap --script ssh-brute -p 22 --script-args userdb=test-users.txt,passdb=test-passwords.txt 127.0.0.1

# Expected: Should find testuser:testpass
# This confirms your setup works
```

### Scenario 2: Web Application Testing Lab

```cmd
# Setup: Install Apache with basic auth
sudo apt install apache2
sudo htpasswd -c /etc/apache2/.htpasswd testuser
# Enter: testpass

# Test the authentication
nmap --script http-auth -p 80 --script-args http-auth.path=/secure 127.0.0.1

# This helps you understand how web auth works
# And how attackers might test it
```

### Scenario 3: Windows Test Environment

```cmd
# In a Windows VM or test machine
# Enable SMB sharing

# Test with safe parameters
nmap --script smb-brute -p 445 --script-args smbbasic=1,brute.retries=1 [windows-test-ip]

# Use known test credentials
# Never test on production Windows systems!
```

## Advanced Script Configuration

### Using Multiple Authentication Methods

```cmd
# Try multiple authentication strategies
nmap --script ssh-brute --script-args "userdb=users.txt,passdb=passwords.txt,brute.emptypass=true" target

# This tries:
1. Username/password combinations from files
2. Empty password for each username
3. Username as password for each username
```

### Customizing Brute-Force Patterns

```cmd
# Define custom brute-force patterns
nmap --script ssh-brute --script-args "brute.passonly=true,brute.credfile=credentials.txt" target

# In credentials.txt:
admin:admin
admin:password
user:user
test:test

# This tests specific known weak combinations
```

### Session Management and Resumption

```cmd
# Save session state for resumption
nmap --script ssh-brute --script-args "brute.mode=creds,brute.credfile=creds.txt,brume.save=true" target

# Later, resume where you left off
nmap --script ssh-brute --script-args "brute.restore=true" target
```

## Monitoring and Logging

### Verbose Output for Monitoring

```cmd
# See detailed progress
nmap -v --script ssh-brute target

# Sample verbose output:
NSE: Starting ssh-brute against target:22
NSE: ssh-brute Trying username 'admin' with password 'admin'
NSE: ssh-brute Trying username 'admin' with password 'password'
NSE: ssh-brute Trying username 'admin' with password '123456'
NSE: ssh-brute Valid credentials found: admin:password123
```

### Saving Results Securely

```cmd
# Save results with encryption consideration
nmap --script ssh-brute -oA results --script-args userdb=users.txt,passdb=passwords.txt target

# IMPORTANT: Results contain valid credentials!
# Store encrypted: gpg -c results.nmap
# Delete after use: shred -u results.*
```

## Security Implications and Defenses

### What Attackers Look For

```cmd
Common attack patterns:
1. Default credentials (admin:admin, root:root)
2. Common weak passwords (password123, 123456)
3. Season-based passwords (Summer2024!)
4. Company name variations
5. Password reuse across services
```

### Defensive Measures You Should Implement

```cmd
1. STRONG password policies:
   - Minimum 12 characters
   - Mix of character types
   - No dictionary words
   - Regular rotation

2. Account lockout policies:
   - Lock after 5 failed attempts
   - Auto-unlock after 30 minutes
   - Alert on lockouts

3. Multi-factor authentication (MFA):
   - Required for all external access
   - Required for administrative accounts
   - Use authenticator apps, not SMS

4. Monitoring:
   - Log all authentication attempts
   - Alert on brute-force patterns
   - Regular review of authentication logs
```

### Tools for Defensive Testing

```cmd
# Use these to test YOUR OWN defenses:

1. Fail2ban: Automatically blocks brute-force attempts
   sudo apt install fail2ban

2. CrowdSec: Modern intrusion detection
   https://www.crowdsec.net/

3. Custom monitoring scripts:
   # Monitor auth logs for brute-force
   tail -f /var/log/auth.log | grep "Failed password"
   
4. Password auditing tools:
   john --wordlist=rockyou.txt hashed_passwords.txt
   hashcat -m 1000 hashed_passwords.txt rockyou.txt
```

## Legal and Compliance Considerations

### Industry Standards

```cmd
Compliance frameworks requiring auth testing:
1. PCI-DSS: Requires regular password testing
2. HIPAA: Requires access control testing
3. ISO 27001: Requires security control testing
4. NIST SP 800-53: Requires authentication testing

BUT: All require PROPER AUTHORIZATION and CONTROLS
```

### Documentation Requirements

When conducting authorized testing:

```cmd
Required documentation:
1. Written authorization letter
2. Test scope definition
3. Emergency contact information
4. Rollback procedures
5. Results handling procedures
6. Clean-up confirmation

Sample authorization template:
Date: [Date]
To: [Tester Name]
From: [System Owner]
Subject: Authorization for Security Testing

You are authorized to test the following systems:
[System List]

During this period: [Date Range]
Using these methods: [Method List]
With these constraints: [Constraints]

Emergency Contact: [Contact Info]
All findings confidential.
```

## Common Mistakes to Avoid

### Technical Mistakes

```cmd
# BAD: No rate limiting (causes lockouts)
nmap --script ssh-brute -T5 target

# BAD: Testing production during business hours
# BAD: Using default wordlists without customization
# BAD: Not monitoring for system impact
# BAD: Saving results insecurely
```

### Ethical/Legal Mistakes

```cmd
# NEVER: Test without written permission
# NEVER: Share results containing credentials
# NEVER: Test third-party systems (cloud, SaaS)
# NEVER: Continue after causing disruption
# NEVER: Use for unauthorized access
```

## Practice Exercises (IN LAB ENVIRONMENT ONLY)

### Exercise 1: SSH Security Lab

```cmd
# Setup a test SSH server in a VM
# 1. Install Ubuntu Server in VirtualBox
# 2. Install OpenSSH: sudo apt install openssh-server
# 3. Create test users with weak passwords

# Test your defenses:
# 1. Install and configure fail2ban
# 2. Set up account lockout policies
# 3. Test with nmap scripts
# 4. Monitor logs and see what gets blocked

# Goal: Understand how attacks work to better defend
```

### Exercise 2: Web Authentication Lab

```cmd
# Setup Apache with different auth methods
# 1. Basic authentication
# 2. Digest authentication  
# 3. Form-based authentication

# Test each with:
nmap --script http-auth --script-args http-auth.path=/secure

# Compare security of different methods
# Implement rate limiting and monitoring
```

### Exercise 3: Password Policy Testing

```cmd
# Create users with different password strengths
# User1: password (weak)
# User2: P@ssw0rd! (medium)
# User3: Xk8&g2#qL9!mP (strong)

# Test with common wordlists
# See which get compromised
# Use results to educate users about password strength
```

## Recovery and Cleanup Procedures

### If You Cause Issues

```cmd
Immediate actions if problems occur:
1. STOP all testing immediately
2. NOTIFY system owners/contacts
3. DOCUMENT what happened
4. ASSIST with recovery if asked
5. REVIEW what went wrong

Example notification template:
"Testing incident at [time] on [system]
Action taken: [what you did]
Issue caused: [what happened]
Immediate fix: [what you did to fix]
Prevention: [how to prevent recurrence]"
```

### Post-Testing Cleanup

```cmd
# Remove all test accounts
sudo userdel testuser
sudo userdel testadmin

# Remove wordlists and results
shred -u usernames.txt passwords.txt results.nmap

# Clear command history
history -c

# Document cleanup
echo "Cleanup completed $(date)" >> testing-log.txt
```

## Next Steps in Your Learning Journey

### Building on This Knowledge

```cmd
Related skills to develop:
1. Password hashing and cryptography
2. Multi-factor authentication systems
3. Identity and Access Management (IAM)
4. Single Sign-On (SSO) systems
5. Privileged Access Management (PAM)

Certifications to consider:
- Certified Ethical Hacker (CEH)
- Offensive Security Certified Professional (OSCP)
- GIAC Penetration Tester (GPEN)
```

### Resources for Further Learning

```cmd
Safe practice environments:
1. Hack The Box (https://www.hackthebox.com/)
2. TryHackMe (https://tryhackme.com/)
3. VulnHub (https://www.vulnhub.com/)
4. PentesterLab (https://pentesterlab.com/)

Important: Always use these platforms LEGALLY
and follow their terms of service.
```

## Final Thoughts and Responsibility

### The Ethical Hacker's Creed

As you learn these powerful techniques, remember:

1. **With great power comes great responsibility**
2. **Always act with integrity**
3. **Use knowledge to protect, not to harm**
4. **Respect privacy and boundaries**
5. **Contribute to making the digital world safer**

### Your Role in Cybersecurity

Whether you become:
- A security analyst protecting networks
- A penetration tester finding vulnerabilities
- A system administrator hardening systems
- A developer writing secure code

This knowledge helps you understand threats so you can build better defenses.

## Quick Reference Cheat Sheet

```cmd
# SAFE PRACTICE COMMANDS (LAB ONLY)
nmap --script ssh-brute -p 22 127.0.0.1 --script-args userdb=test-users.txt
nmap --script http-auth -p 80 127.0.0.1 --script-args http-auth.path=/test
nmap --script ftp-brute -p 21 127.0.0.1 --script-args brute.delay=2

# SAFETY PARAMETERS (ALWAYS USE)
--script-args brute.delay=2        # 2 second delay
--script-args brute.threads=1      # One attempt at a time
--script-timeout 10m               # Stop after 10 minutes
--max-retries 1                    # Minimal retries

# DOCUMENTATION (ALWAYS DO)
-oA results                         # Save all output formats
--append-output                     # Add to existing files
echo "$(date): Test started" >> log.txt  # Manual logging
```

> **Use this knowledge wisely, ethically, and always with permission.** You now have the power to test authentication systems - use that power to make systems more secure, not less.

> Stay ethical, stay legal, and keep making the digital world safer for everyone. 🛡️

**🎉 Congratulations on completing Day 14!** You've learned one of the most powerful and dangerous aspects of security testing. Remember: This knowledge is like a surgeon's scalpel - in skilled, ethical hands it heals and protects; in the wrong hands it can cause great harm.

*"The strongest lock is useless if the key is under the mat. Today you learned to check for hidden keys - so you can advise people to keep them safe."*
