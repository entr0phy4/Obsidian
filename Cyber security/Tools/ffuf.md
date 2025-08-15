# Ffuf (Fuzz Faster U Fool)

## Overview
Ffuf is a fast web fuzzer written in Go. It's designed to be fast, flexible, and feature-rich for web application security testing, directory discovery, parameter fuzzing, and more.

## Installation
```bash
# Ubuntu/Debian
sudo apt install ffuf

# From GitHub releases
wget https://github.com/ffuf/ffuf/releases/download/v1.5.0/ffuf_1.5.0_linux_amd64.tar.gz
tar -xzf ffuf_1.5.0_linux_amd64.tar.gz
sudo mv ffuf /usr/local/bin/

# Using Go
go install github.com/ffuf/ffuf@latest
```

## Basic Usage

### Directory Discovery
```bash
# Basic directory fuzzing
ffuf -u http://target.com/FUZZ -w /usr/share/wordlists/dirb/common.txt

# With specific file extensions
ffuf -u http://target.com/FUZZ -w wordlist.txt -e .php,.html,.txt,.js

# Multiple wordlists
ffuf -u http://target.com/FUZZ -w wordlist1.txt:FILE -w wordlist2.txt:EXT -e FILE.EXT
```

### Subdomain Discovery
```bash
# Subdomain fuzzing
ffuf -u http://FUZZ.target.com -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt

# With custom Host header
ffuf -u http://target.com -H "Host: FUZZ.target.com" -w wordlist.txt
```

### Parameter Discovery
```bash
# GET parameter fuzzing
ffuf -u http://target.com/page.php?FUZZ=value -w parameters.txt

# POST parameter fuzzing  
ffuf -u http://target.com/login -X POST -d "username=admin&FUZZ=value" -w parameters.txt

# Header fuzzing
ffuf -u http://target.com -H "FUZZ: value" -w headers.txt
```

## Advanced Filtering

### Status Code Filtering
```bash
# Match specific status codes
ffuf -u http://target.com/FUZZ -w wordlist.txt -mc 200,204,301,302,307,401,403

# Filter out status codes
ffuf -u http://target.com/FUZZ -w wordlist.txt -fc 404,500

# Match all except specified
ffuf -u http://target.com/FUZZ -w wordlist.txt -fc 404
```

### Response Size Filtering
```bash
# Filter by response size
ffuf -u http://target.com/FUZZ -w wordlist.txt -fs 1234

# Match specific sizes
ffuf -u http://target.com/FUZZ -w wordlist.txt -ms 1000-5000

# Filter multiple sizes
ffuf -u http://target.com/FUZZ -w wordlist.txt -fs 0,1234,5678
```

### Content Filtering
```bash
# Filter by response content
ffuf -u http://target.com/FUZZ -w wordlist.txt -fr "not found"

# Match specific content
ffuf -u http://target.com/FUZZ -w wordlist.txt -mr "admin panel"

# Filter by number of words
ffuf -u http://target.com/FUZZ -w wordlist.txt -fw 100

# Filter by number of lines
ffuf -u http://target.com/FUZZ -w wordlist.txt -fl 50
```

## Performance Options

### Threading and Rate Control
```bash
# Increase threads (default 40)
ffuf -u http://target.com/FUZZ -w wordlist.txt -t 100

# Rate limiting (requests per second)
ffuf -u http://target.com/FUZZ -w wordlist.txt -rate 10

# Request delay
ffuf -u http://target.com/FUZZ -w wordlist.txt -p 0.5
```

### Timeout and Retry
```bash
# Custom timeout
ffuf -u http://target.com/FUZZ -w wordlist.txt -timeout 10

# Retry on failures
ffuf -u http://target.com/FUZZ -w wordlist.txt -retry-all
```

## Output Options

### Output Formats
```bash
# JSON output
ffuf -u http://target.com/FUZZ -w wordlist.txt -o results.json -of json

# CSV output
ffuf -u http://target.com/FUZZ -w wordlist.txt -o results.csv -of csv

# HTML output  
ffuf -u http://target.com/FUZZ -w wordlist.txt -o results.html -of html

# Multiple formats
ffuf -u http://target.com/FUZZ -w wordlist.txt -o results -of all
```

### Verbosity Control
```bash
# Quiet mode
ffuf -u http://target.com/FUZZ -w wordlist.txt -s

# Verbose mode
ffuf -u http://target.com/FUZZ -w wordlist.txt -v

# No banner
ffuf -u http://target.com/FUZZ -w wordlist.txt -nb
```

## Practical Examples

### API Endpoint Discovery
```bash
# API versioning
ffuf -u http://api.target.com/vFUZZ/users -w <(seq 1 10)

# API endpoint fuzzing
ffuf -u http://api.target.com/FUZZ -w /usr/share/seclists/Discovery/Web-Content/api/api-endpoints.txt -mc 200,201,202,204,400,401,403,405

# RESTful API methods
ffuf -u http://api.target.com/users -X FUZZ -w <(echo -e "GET\nPOST\nPUT\nDELETE\nPATCH\nOPTIONS")
```

### Virtual Host Discovery
```bash
# Virtual host fuzzing
ffuf -u http://target.com -H "Host: FUZZ.target.com" -w wordlist.txt -fs 1234

# With multiple domains
ffuf -u http://FUZZ -w <(cat domains.txt | sed 's/^/http:\/\//')
```

### Login Brute Force
```bash
# Username enumeration
ffuf -u http://target.com/login -X POST -d "username=FUZZ&password=test" -w usernames.txt -fc 200 -mr "invalid password"

# Password brute force
ffuf -u http://target.com/login -X POST -d "username=admin&password=FUZZ" -w passwords.txt -fr "invalid"
```

### File Extension Discovery
```bash
# Discover file extensions
ffuf -u http://target.com/backup.FUZZ -w /usr/share/seclists/Discovery/Web-Content/web-extensions.txt

# Backup file discovery
ffuf -u http://target.com/FUZZ -w <(echo "index" | sed 's/$/\n&.bak\n&.backup\n&.old\n&~/')
```

## Advanced Techniques

### Cluster Bomb Attack
```bash
# Multiple parameter fuzzing
ffuf -u http://target.com/search?category=FUZZ&type=FUZZ2 -w categories.txt:FUZZ -w types.txt:FUZZ2 -mode clusterbomb
```

### Pitchfork Attack  
```bash
# Synchronized parameter fuzzing
ffuf -u http://target.com/login -X POST -d "username=FUZZ&password=FUZZ2" -w usernames.txt:FUZZ -w passwords.txt:FUZZ2 -mode pitchfork
```

### SNI Fuzzing
```bash
# Server Name Indication fuzzing
ffuf -u https://target.com -H "Host: FUZZ.target.com" -w wordlist.txt -sni FUZZ.target.com
```

## Integration and Automation

### With Other Tools
```bash
# Feed results to other tools
ffuf -u http://target.com/FUZZ -w wordlist.txt -s | grep "200" | awk '{print $1}' | httpx

# Chain with curl for detailed analysis
ffuf -u http://target.com/FUZZ -w wordlist.txt -s | while read url; do curl -I "$url"; done
```

### Scripting Integration
```bash
#!/bin/bash
# Automated subdomain and directory discovery
echo "Discovering subdomains..."
ffuf -u http://FUZZ.target.com -w subdomains.txt -o subs.json -of json -s

echo "Discovering directories on main site..."
ffuf -u http://target.com/FUZZ -w directories.txt -o dirs.json -of json -s
```

## Configuration File
```yaml
# ~/.ffufrc or ffufrc.yaml
general:
  threads: 40
  rate: 0
  timeout: 10
  delay: 0
  
output:
  format: json
  file: ffuf-results
  
http:
  headers:
    User-Agent: "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36"
  
filter:
    status: [404, 500]
```

## Wordlists for Different Scenarios
```bash
# Directory discovery
/usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt
/usr/share/seclists/Discovery/Web-Content/raft-medium-directories.txt

# File discovery
/usr/share/seclists/Discovery/Web-Content/raft-medium-files.txt
/usr/share/seclists/Discovery/Web-Content/common.txt

# API endpoints
/usr/share/seclists/Discovery/Web-Content/api/api-endpoints.txt
/usr/share/seclists/Discovery/Web-Content/swagger.txt

# Parameters
/usr/share/seclists/Discovery/Web-Content/burp-parameter-names.txt
```

## Comparison with Other Tools

### Ffuf vs Gobuster
- **Speed**: Ffuf generally faster for complex fuzzing
- **Filtering**: Ffuf has more advanced filtering options
- **Simplicity**: Gobuster simpler for basic directory discovery
- **Features**: Ffuf supports more fuzzing types

### Ffuf vs Wfuzz
- **Performance**: Ffuf typically faster
- **Features**: Similar feature set
- **Learning curve**: Ffuf easier to learn
- **Output**: Both support multiple output formats

## Related Tools
- [[gobuster]] - Directory/DNS brute forcer
- [[wfuzz]] - Web application fuzzer
- [[dirb]] - URL brute forcer
- [[Burpsuite]] - Manual testing platform
- [[dirsearch]] - Web path scanner

## Related Topics
- [[Web reconnaissance]]
- [[Directory traversal]]
- [[API testing]]
- [[Parameter pollution]]
- [[Fuzzing techniques]]
