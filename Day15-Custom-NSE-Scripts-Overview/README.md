  # Day 15 - Custom NSE Scripts Overview

## Day Overview
Welcome to Day 15! Today, we're taking a huge leap forward - we're learning how to create our **own custom NSE scripts**. This is where you transition from being a tool user to a tool creator. Think of it like moving from driving a car to building one!

## Learning Objectives
By completing Day 15, you will be able to:
- Understand the structure of NSE scripts
- Write basic custom scripts in Lua
- Test and debug your scripts
- Understand when to create custom scripts
- Share and maintain your scripts

## Why Create Custom Scripts?

### The Power of Customization

```
Why write custom scripts when 600+ exist already?

1. Specific Needs: Your organization has unique services
2. New Technologies: Emerging tech not yet in Nmap
3. Custom Checks: Company-specific security policies
4. Automation: Repetitive tasks you do manually
5. Integration: Connect Nmap with other tools
6. Learning: Deep understanding of how NSE works
```

### Real-World Examples

```
Custom scripts you might create:

1. Check for company-specific software versions
2. Test custom API endpoints
3. Validate security configurations unique to your org
4. Automate compliance checks
5. Integrate with internal ticketing systems
6. Create specialized discovery for your network
```

## Lua Programming Basics for NSE

### Why Lua?
Nmap chose Lua because it's:
- Lightweight and fast
- Easy to learn
- Excellent for embedding in applications
- Great for scripting and automation

### Basic Lua Syntax You Need

```
-- Comments start with --
-- Variables
local name = "value"
local number = 42
local is_true = true

-- Tables (Lua's arrays/objects)
local fruits = {"apple", "banana", "orange"}
local person = {name="Alice", age=30}

-- Functions
function greet(name)
    return "Hello " .. name  -- .. concatenates strings
end

-- If statements
if port.number == 80 then
    -- do something
elseif port.number == 443 then
    -- do something else
else
    -- default action
end

-- Loops
for i=1, 5 do
    print("Number: " .. i)
end

for key, value in pairs(fruits) do
    print(key .. ": " .. value)
end
```

## NSE Script Structure (Complete Breakdown)

### Basic Template

```
-- Custom Script: my-first-script.nse

-- HEADER SECTION
description = [[
My First Custom NSE Script
This script does something interesting.
]]

-- CATEGORIES SECTION
categories = {"safe", "discovery"}

-- RULES SECTION
portrule = function(host, port)
    -- When should this script run?
    return port.protocol == "tcp" and port.number == 80
end

hostrule = function(host)
    -- Alternative: Host-based rules
    return host.ip == "127.0.0.1"
end

-- ACTION SECTION
action = function(host, port)
    -- Main script logic here
    return "Found web server on port " .. port.number
end

-- POST-ACTION SECTION (optional)
postrule = function() 
    -- Runs after all scripts
    return true
end
```

### Understanding Each Section

**1. Header Section:**
- `description`: Human-readable explanation
- `author`: Your name (optional but good practice)
- `license`: License information (optional)
- `dependencies`: Other scripts/libs needed

**2. Categories Section:**
- Tells Nmap where this script belongs
- Affects when it runs (with `--script=safe`, etc.)
- Common: safe, intrusive, vuln, discovery, version

**3. Rules Section:**
- `portrule`: Runs when specific ports are open
- `hostrule`: Runs based on host characteristics
- `prerule`: Runs before other scripts
- `postrule`: Runs after other scripts

**4. Action Section:**
- Where the real work happens
- Receives host and port objects
- Returns string with results
- Can use NSE libraries for complex tasks

## Creating Your First Script

### Script 1: Simple Port Checker

Let's create a script that checks for specific ports and reports them.

```
-- File: simple-port-check.nse

description = [[
Simple Port Checker
Checks for common ports and reports findings.
]]

author = "Your Name"
license = "Same as Nmap"

categories = {"safe", "discovery"}

portrule = function(host, port)
    -- Run on these specific ports
    local common_ports = {22, 80, 443, 3389, 8080}
    for _, p in ipairs(common_ports) do
        if port.number == p then
            return true
        end
    end
    return false
end

action = function(host, port)
    local messages = {
        [22] = "SSH service detected",
        [80] = "HTTP web server detected", 
        [443] = "HTTPS web server detected",
        [3389] = "Windows Remote Desktop detected",
        [8080] = "Alternate web server detected"
    }
    
    local message = messages[port.number] or "Unknown service"
    return message .. " on port " .. port.number
end
```

### Testing Your First Script

```
# Save the script
nano simple-port-check.nse

# Test it locally
nmap --script ./simple-port-check.nse 127.0.0.1

# If it works, move to Nmap scripts directory
sudo cp simple-port-check.nse /usr/share/nmap/scripts/
sudo nmap --script-updatedb

# Now use it like any other script
nmap --script simple-port-check target
```

## Working with NSE Libraries

### Essential NSE Libraries

Nmap provides powerful libraries you can use in your scripts:

```
-- stdnse: Standard NSE functions
local stdnse = require "stdnse"
stdnse.print_verbose(1, "Debug message")
stdnse.debug("More details")

-- shortport: Easy port rules
local shortport = require "shortport"
portrule = shortport.port_or_service({22,80,443}, {"ssh","http","https"})

-- http: HTTP client
local http = require "http"
local response = http.get(host, port, "/")
if response.status == 200 then
    return "Page accessible"
end

-- smb: Windows SMB protocol
local smb = require "smb"
local status, result = smb.get_os(host)

-- dns: DNS resolution  
local dns = require "dns"
local status, ips = dns.resolve("example.com")

-- json: JSON parsing
local json = require "json"
local data = json.parse(response.body)
```

### Library Example: HTTP Check Script

```
-- File: http-status-check.nse

description = [[
HTTP Status Checker
Checks HTTP response codes and content.
]]

author = "Your Name"
categories = {"safe", "discovery"}

local shortport = require "shortport"
local http = require "http"
local stdnse = require "stdnse"

portrule = shortport.http

action = function(host, port)
    -- Make HTTP request
    local response = http.get(host, port, "/")
    
    if not response then
        return "No response from server"
    end
    
    -- Build result string
    local result = string.format("Status: %d %s\n", 
                                response.status, 
                                response.status_line)
    
    -- Check for interesting headers
    if response.header["server"] then
        result = result .. "Server: " .. response.header["server"] .. "\n"
    end
    
    if response.header["x-powered-by"] then
        result = result .. "Powered by: " .. response.header["x-powered-by"] .. "\n"
    end
    
    -- Check page size
    if response.body then
        result = result .. "Page size: " .. #response.body .. " bytes\n"
    end
    
    return result
end
```

## Advanced Script Features

### Script Arguments

Allow users to pass arguments to your script:

```
-- File: custom-http-check.nse

description = [[
Custom HTTP Check with arguments
]]

categories = {"safe", "discovery"}

-- Define script arguments
local args = {
    path = {default="/", description="Path to check"},
    method = {default="GET", description="HTTP method"},
    timeout = {default=5000, description="Timeout in ms"}
}

-- Access arguments in action
action = function(host, port)
    local path = stdnse.get_script_args("custom-http-check.path") or "/"
    local method = stdnse.get_script_args("custom-http-check.method") or "GET"
    
    -- Use arguments
    local response = http.generic_request(host, port, method, path)
    
    return string.format("Checked %s with %s: Status %d", 
                        path, method, response.status)
end
```

### Using the script with arguments:

```
nmap --script custom-http-check --script-args custom-http-check.path=/admin target
```

### Output Formatting

Make your script output look professional:

```
action = function(host, port)
    local output = {}
    
    -- Add sections to output
    table.insert(output, "=== Service Information ===")
    table.insert(output, "Port: " .. port.number)
    table.insert(output, "Service: " .. port.service)
    
    if port.version then
        table.insert(output, "Version: " .. port.version)
    end
    
    -- Add findings
    table.insert(output, "\n=== Findings ===")
    table.insert(output, "• Found open port")
    table.insert(output, "• Service is responsive")
    
    -- Return formatted output
    return stdnse.format_output(true, output)
end
```

## Real-World Custom Script Examples

### Example 1: Custom Service Detector

```
-- File: custom-service-detector.nse
-- Detects custom company applications

description = [[
Custom Company Application Detector
Checks for internally developed applications.
]]

author = "Your Company Security Team"
categories = {"safe", "discovery"}

local http = require "http"
local stdnse = require "stdnse"
local shortport = require "shortport"

portrule = shortport.http

-- Company-specific application signatures
local app_signatures = {
    {
        name = "Company CRM",
        path = "/crm/login",
        keyword = "CRM Dashboard"
    },
    {
        name = "Internal Wiki", 
        path = "/wiki",
        keyword = "Company Knowledge Base"
    },
    {
        name = "Time Tracking",
        path = "/time",
        keyword = "Time Entry System"
    }
}

action = function(host, port)
    local results = {}
    
    for _, app in ipairs(app_signatures) do
        local response = http.get(host, port, app.path)
        
        if response and response.status == 200 then
            if response.body and string.find(response.body, app.keyword) then
                table.insert(results, string.format("Found: %s at %s", 
                                                   app.name, app.path))
            end
        end
    end
    
    if #results > 0 then
        return stdnse.format_output(true, results)
    end
    
    return nil  -- No output if nothing found
end
```

### Example 2: Compliance Check Script

```
-- File: ssl-compliance-check.nse
-- Checks SSL/TLS compliance against company policy

description = [[
SSL/TLS Compliance Checker
Validates SSL configuration against company security policy.
]]

author = "Company Security Team"
categories = {"safe", "vuln"}

local shortport = require "shortport"
local sslcert = require "sslcert"
local stdnse = require "stdnse"

portrule = shortport.ssl

action = function(host, port)
    local findings = {}
    local status, cert = sslcert.getCertificate(host, port)
    
    if not status then
        return "Could not retrieve certificate"
    end
    
    -- Check 1: Certificate expiration
    local days_remaining = (cert.validity - os.time()) / 86400
    if days_remaining < 30 then
        table.insert(findings, "⚠️ Certificate expires in " .. 
                              math.floor(days_remaining) .. " days")
    end
    
    -- Check 2: Key strength
    if cert.pubkey.type == "rsa" and cert.pubkey.bits < 2048 then
        table.insert(findings, "⚠️ Weak RSA key: " .. cert.pubkey.bits .. " bits (min 2048 required)")
    end
    
    -- Check 3: Signature algorithm
    if cert.sig_alg:find("SHA1") then
        table.insert(findings, "⚠️ Weak signature algorithm: " .. cert.sig_alg)
    end
    
    -- Check 4: Compliance summary
    if #findings == 0 then
        table.insert(findings, "✅ SSL/TLS configuration compliant with company policy")
    else
        table.insert(findings, 1, "SSL Compliance Check Failed:")
        table.insert(findings, "Total issues: " .. #findings - 1)
    end
    
    return stdnse.format_output(true, findings)
end
```

## Testing and Debugging Your Scripts

### Development Workflow

```
# Step 1: Write script
nano my-script.nse

# Step 2: Test with debug output
nmap -d --script ./my-script.nse 127.0.0.1 2>&1 | less

# Step 3: Fix errors, test again
# Common errors:
# - Syntax errors (missing end, etc.)
# - Library not found
# - Variable scope issues

# Step 4: Add verbose debugging
local stdnse = require "stdnse"
stdnse.debug1("Starting script")
stdnse.debug2("More details: %s", variable)
stdnse.debug3("Even more details")

# Step 5: Test on various targets
nmap --script ./my-script.nse test-server-1
nmap --script ./my-script.nse test-server-2

# Step 6: Package and share
# Create README, add to company script repository
```

### Debugging Techniques

```
-- Add debug levels to your script
action = function(host, port)
    stdnse.debug1("Script started for " .. host.ip)
    
    -- Check variables
    stdnse.debug2("Port number: %d", port.number)
    stdnse.debug2("Port state: %s", port.state)
    
    -- Conditional debugging
    if port.version then
        stdnse.debug3("Version info: %s", port.version)
    end
    
    -- Use -d flag with nmap to see debug output
    -- Levels: -d (level 1), -dd (level 2), -ddd (level 3)
end
```

### Unit Testing Your Scripts

Create test files to verify your script works:

```
-- File: test-my-script.lua
-- Separate test script

local myscript = require "my-script"

-- Mock host and port objects
local test_host = {
    ip = "127.0.0.1",
    name = "localhost"
}

local test_port = {
    number = 80,
    protocol = "tcp",
    state = "open",
    service = "http"
}

-- Test port rule
print("Port rule test:")
print("Should be true:", myscript.portrule(test_host, test_port))

-- Test with different port
test_port.number = 22
print("Should be false:", myscript.portrule(test_host, test_port))

-- Reset and test action
test_port.number = 80
local result = myscript.action(test_host, test_port)
print("Action result:", result)
```

## Best Practices for Script Development

### Code Organization

```
-- 1. Clear structure
-- Header
-- Imports
-- Constants
-- Helper functions
-- Rules
-- Main action

-- 2. Use helper functions
local function check_http_response(response)
    if not response then return false, "No response" end
    if response.status ~= 200 then return false, "Status: " .. response.status end
    return true, "OK"
end

-- 3. Consistent naming
local is_service_active  -- boolean prefix
local max_retries_count  -- descriptive
local server_list        -- clear purpose

-- 4. Comments for complex logic
-- This algorithm checks for buffer overflow patterns
-- Based on CVE-2021-1234 specification
local pattern = "A" * 1024  -- 1KB buffer test
```

### Error Handling

```
-- Always handle potential errors
action = function(host, port)
    local status, result = pcall(function()
        -- Risky operations here
        local response = http.get(host, port, "/admin")
        if not response then
            error("HTTP request failed")
        end
        return response
    end)
    
    if not status then
        return "Error: " .. result
    end
    
    -- Process successful result
    return "Success: " .. result.status
end
```

### Performance Considerations

```
-- 1. Minimize network calls
-- Cache results when possible
local cached_results = {}

-- 2. Use timeouts
local http = require "http"
http.set_timeout(5000)  -- 5 second timeout

-- 3. Clean up resources
-- Close connections, free memory

-- 4. Rate limiting for network operations
local delay = 100  -- milliseconds
stdnse.sleep(delay)
```

## Security Considerations for Custom Scripts

### Script Safety Levels

When creating scripts, classify them appropriately:

```
-- Safe scripts (information gathering only)
categories = {"safe"}

-- Intrusive scripts (may affect services)
categories = {"intrusive"}

-- Vulnerability scripts (testing for flaws)
categories = {"vuln"}

-- Never create destructive scripts without warnings
if dangerous_operation then
    stdnse.print_verbose(1, "WARNING: This may disrupt service")
    -- Ask for confirmation or add safety checks
end
```

### Input Validation

```
-- Never trust script arguments blindly
local user_input = stdnse.get_script_args("script-name.path") or "/"
-- Validate input
if string.find(user_input, "%.%.") then
    return "Error: Directory traversal attempt detected"
end

-- Sanitize inputs
local safe_path = string.gsub(user_input, "[^a-zA-Z0-9/_%-%.]", "")
```

## Next Steps in Script Development

### Advanced Topics to Explore

1. **Parallel execution**: Making scripts faster
2. **State management**: Remembering between runs
3. **Protocol implementation**: Adding new protocol support
4. **GUI integration**: Zenmap plugin development
5. **Database integration**: Storing results

### Learning Resources

```
Official documentation:
- Nmap NSE Guide: https://nmap.org/book/nse.html
- NSE API Reference: https://nmap.org/nsedoc/

Books:
- "Nmap Network Scanning" (Official Nmap Guide)
- "Mastering Nmap Scripting Engine"

Communities:
- Nmap-dev mailing list
- GitHub Nmap organization
- Stack Overflow (nmap tag)
```

## Final Project: Create a Complete Script

### Project Requirements
Create a script that:
1. Checks a specific service on your network
2. Uses at least one NSE library
3. Accepts arguments from users
4. Provides formatted output
5. Includes error handling
6. Has proper documentation

### Example Project: Company Web App Monitor

```
-- File: company-app-monitor.nse

description = [[
Company Web Application Monitor
Checks the health and security of internal web applications.
]]

author = "Your Name"
license = "MIT"

categories = {"safe", "discovery"}

local http = require "http"
local stdnse = require "stdnse"
local shortport = require "shortport"

-- Script arguments
local args = {
    check_auth = {default=true, description="Check authentication"},
    timeout = {default=10000, description="Timeout in milliseconds"}
}

portrule = shortport.http

action = function(host, port)
    local results = {}
    local check_auth = stdnse.get_script_args("company-app-monitor.check_auth")
    
    -- Check 1: Basic connectivity
    table.insert(results, "=== Basic Connectivity ===")
    local response = http.get(host, port, "/", {timeout=10000})
    
    if not response then
        table.insert(results, "❌ Cannot connect to application")
        return stdnse.format_output(true, results)
    end
    
    table.insert(results, "✅ Application is reachable")
    table.insert(results, "Status: " .. response.status .. " " .. response.status_line)
    
    -- Check 2: Response time
    if response.time then
        table.insert(results, "Response time: " .. string.format("%.2f", response.time) .. " seconds")
    end
    
    -- Check 3: Security headers
    table.insert(results, "\n=== Security Headers ===")
    local security_headers = {
        "X-Frame-Options",
        "X-Content-Type-Options", 
        "Content-Security-Policy",
        "Strict-Transport-Security"
    }
    
    local has_security = false
    for _, header in ipairs(security_headers) do
        if response.header[header] then
            table.insert(results, "✅ " .. header .. ": " .. response.header[header])
            has_security = true
        else
            table.insert(results, "❌ Missing: " .. header)
        end
    end
    
    if not has_security then
        table.insert(results, "⚠️ No security headers detected")
    end
    
    -- Check 4: Authentication (if enabled)
    if check_auth then
        table.insert(results, "\n=== Authentication Check ===")
        local auth_response = http.get(host, port, "/login")
        
        if auth_response and auth_response.status == 401 then
            table.insert(results, "✅ Authentication required for /login")
        elseif auth_response and auth_response.status == 200 then
            table.insert(results, "⚠️ /login page accessible without authentication")
        else
            table.insert(results, "ℹ️ Could not determine authentication status")
        end
    end
    
    -- Summary
    table.insert(results, "\n=== Summary ===")
    table.insert(results, "Application: " .. host.ip .. ":" .. port.number)
    table.insert(results, "Checks performed: " .. #results - 5)  -- Adjust for headers
    
    return stdnse.format_output(true, results)
end
```

## Sharing and Maintaining Scripts

### Documentation Standards

```
-- File: well-documented-script.nse

---[[
Script: Well Documented Example
Author: Your Name
Version: 1.0
Date: 2024-01-15

Description:
This script demonstrates proper documentation standards.
It checks for specific conditions and reports findings.

Usage:
nmap --script well-documented-script target
nmap --script well-documented-script --script-args arg1=value target

Arguments:
arg1: Description of first argument
arg2: Description of second argument

Output:
- Success: Detailed findings
- Failure: Error message or nil

Examples:
1. Basic usage
2. With arguments
3. Expected output samples

Dependencies:
- Requires Nmap 7.90+
- Requires http library

Changelog:
1.0 (2024-01-15): Initial release
--]]
```

### Version Control

```
# Use Git for script management
git init nse-scripts
git add my-script.nse
git commit -m "Initial version of custom script"

# Create branches for features
git checkout -b feature/new-check
# Develop feature
git checkout main
git merge feature/new-check

# Tag releases
git tag -a v1.0 -m "First stable release"
git push --tags
```

## Conclusion

### What You've Accomplished Today

1. **Learned Lua basics** for NSE scripting
2. **Understood script structure** and components
3. **Created your first scripts** from scratch
4. **Worked with NSE libraries** for powerful functionality
5. **Implemented best practices** for script development
6. **Explored real-world examples** you can build upon

### The Path Forward

Custom NSE scripting opens unlimited possibilities:
- **Automate repetitive security tasks**
- **Create specialized tools** for your organization
- **Contribute to the Nmap community**
- **Build a portfolio** of security tools
- **Develop deeper understanding** of network security

### Final Challenge
Create a script that solves a real problem in your environment. Start simple, test thoroughly, and share what you learn. The best way to master NSE scripting is by doing.

> *"First you learn the tools, then you master the tools, then you become the toolmaker. Today you took the first step toward becoming a creator in the world of cybersecurity."*

> Remember: Every expert was once a beginner who kept practicing. Keep writing, keep testing, keep learning. Your custom scripts could one day help secure networks around the world.

**🎉 Congratulations on completing Day 15!** You've crossed an important threshold - from using tools to creating them. This skill will serve you throughout your cybersecurity career.
