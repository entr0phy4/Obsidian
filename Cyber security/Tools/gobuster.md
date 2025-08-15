# Gobuster

## Overview
Gobuster is a fast directory/file & DNS busting tool written in Go. It's designed to brute force directories and files on web servers, DNS subdomains, virtual hostnames, and more.

## Installation
```bash
# Ubuntu/Debian
sudo apt install gobuster

# From source
go install github.com/OJ/gobuster/v3@latest

# Pre-compiled binaries available on GitHub
```

## Main Modes

### 1. Directory/File Brute Force (dir)
```bash
# Basic directory scan
gobuster dir -u http://target.com -w /usr/share/wordlists/dirb/common.txt

# With specific extensions
gobuster dir -u http://target.com -w wordlist.txt -x php,html,txt,js

# Follow redirects and show full URLs
gobuster dir -u http://target.com -w wordlist.txt -r -e

# Custom User-Agent and cookies
gobuster dir -u http://target.com -w wordlist.txt -a "Custom-Agent" -c "session=abc123"
```

### 2. DNS Subdomain Enumeration (dns)
```bash
# Basic subdomain scan
gobuster dns -d target.com -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt

# With custom resolver
gobuster dns -d target.com -w wordlist.txt -r 8.8.8.8:53

# Show IPs and CNAMEs
gobuster dns -d target.com -w wordlist.txt --show-ips --show-cname
```

### 3. Virtual Host Discovery (vhost)
```bash
# Discover virtual hosts
gobuster vhost -u http://target.com -w wordlist.txt

# Append domain to wordlist entries
gobuster vhost -u http://target.com -w wordlist.txt --append-domain
```

### 4. S3 Bucket Enumeration (s3)
```bash
# Basic S3 bucket discovery
gobuster s3 -w wordlist.txt

# Specific region
gobuster s3 -w wordlist.txt -r us-west-2
```

## Common Options

### Output and Formatting
```bash
# Output to file
gobuster dir -u http://target.com -w wordlist.txt -o results.txt

# Quiet mode (only results)
gobuster dir -u http://target.com -w wordlist.txt -q

# No status output
gobuster dir -u http://target.com -w wordlist.txt --no-status

# Verbose output
gobuster dir -u http://target.com -w wordlist.txt -v
```

### Performance Tuning
```bash
# Increase threads (default 10)
gobuster dir -u http://target.com -w wordlist.txt -t 50

# Request delay (avoid rate limiting)
gobuster dir -u http://target.com -w wordlist.txt --delay 100ms

# Timeout settings
gobuster dir -u http://target.com -w wordlist.txt --timeout 10s
```

### HTTP Configuration
```bash
# Custom headers
gobuster dir -u http://target.com -w wordlist.txt -H "Authorization: Bearer token"

# HTTP methods
gobuster dir -u http://target.com -w wordlist.txt -m GET,POST

# Status codes to include
gobuster dir -u http://target.com -w wordlist.txt -s 200,204,301,302,307,401,403

# Exclude specific status codes
gobuster dir -u http://target.com -w wordlist.txt -b 404,500
```

## Practical Examples

### Web Directory Discovery
```bash
# Comprehensive directory scan
gobuster dir \
  -u http://target.com \
  -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt \
  -x php,html,txt,js,xml,json \
  -s 200,204,301,302,307,401,403 \
  -o gobuster-dirs.txt

# Admin panel discovery
gobuster dir \
  -u http://target.com \
  -w /usr/share/seclists/Discovery/Web-Content/CMS/admin-panels.txt \
  -s 200,403 \
  -t 30
```

### API Endpoint Discovery
```bash
# API endpoints
gobuster dir \
  -u http://api.target.com \
  -w /usr/share/seclists/Discovery/Web-Content/api/api-endpoints.txt \
  -s 200,201,202,204,300,301,302,307,400,401,403,405 \
  -H "Content-Type: application/json"
```

### Subdomain Enumeration
```bash
# Comprehensive subdomain scan
gobuster dns \
  -d target.com \
  -w /usr/share/seclists/Discovery/DNS/fierce-hostlist.txt \
  -r 8.8.8.8:53 \
  --show-ips \
  -o subdomains.txt
```

## Wordlists

### Common Wordlists
```bash
# Directory/File discovery
/usr/share/wordlists/dirb/common.txt
/usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt
/usr/share/seclists/Discovery/Web-Content/raft-medium-directories.txt

# Subdomain discovery
/usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt
/usr/share/seclists/Discovery/DNS/fierce-hostlist.txt
/usr/share/seclists/Discovery/DNS/dns-Jhaddix.txt
```

### Custom Wordlist Generation
```bash
# Generate custom wordlist from target
cewl http://target.com -w custom-wordlist.txt

# Combine multiple wordlists
cat wordlist1.txt wordlist2.txt | sort -u > combined-wordlist.txt
```

## Advanced Techniques

### Recursive Directory Scanning
```bash
# Manual recursive approach
gobuster dir -u http://target.com -w wordlist.txt | grep "Status: 301" | awk '{print $1}' > found-dirs.txt

# Then scan each found directory
while read dir; do
  gobuster dir -u "http://target.com$dir" -w wordlist.txt
done < found-dirs.txt
```

### Bypass Techniques
```bash
# Different User-Agents
gobuster dir -u http://target.com -w wordlist.txt -a "Mozilla/5.0 (compatible; Googlebot/2.1)"

# Custom headers to bypass WAF
gobuster dir -u http://target.com -w wordlist.txt -H "X-Forwarded-For: 127.0.0.1"

# Using proxies
gobuster dir -u http://target.com -w wordlist.txt --proxy http://proxy:8080
```

## Integration with Other Tools

### With Nmap
```bash
# Scan web ports first
nmap -p 80,443,8080,8443 -sV target.com

# Then run gobuster on discovered services
gobuster dir -u http://target.com:8080 -w wordlist.txt
```

### With Ffuf (Alternative)
```bash
# Similar functionality with ffuf
ffuf -u http://target.com/FUZZ -w wordlist.txt -c

# Comparison: gobuster is often faster for simple directory busting
# ffuf offers more advanced filtering and fuzzing capabilities
```

## Output Analysis
```bash
# Parse gobuster output
grep "Status: 200" results.txt
grep -E "(Status: 301|Status: 302)" results.txt

# Extract URLs
cat results.txt | grep "Status: 200" | awk '{print $1}' > found-urls.txt
```

## Related Tools
- [[ffuf]] - Web fuzzer with advanced filtering
- [[dirb]] - Web content scanner
- [[dirsearch]] - Advanced web path scanner
- [[Burpsuite]] - Manual testing and validation
- [[wfuzz]] - Web application fuzzer

## Related Topics
- [[Web reconnaissance]]
- [[Directory traversal]]
- [[Web application security]]
- [[Subdomain enumeration]]
