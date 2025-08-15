# WPScan

## Overview
WPScan is a security scanner specifically designed for WordPress websites. It can enumerate users, plugins, themes, and known vulnerabilities in WordPress installations.

## Installation
```bash
# Ubuntu/Debian
sudo apt install wpscan

# Ruby gem installation
sudo gem install wpscan

# Docker
docker run -it --rm wpscanteam/wpscan

# Update WordPress vulnerability database
wpscan --update
```

## Basic Usage

### Simple WordPress Scan
```bash
# Basic scan
wpscan --url http://target.com

# Scan with WordPress vulnerability database updates
wpscan --url http://target.com --update

# Aggressive scan (may trigger WAF)
wpscan --url http://target.com --detection-mode aggressive
```

### User Enumeration
```bash
# Enumerate users (default method)
wpscan --url http://target.com --enumerate u

# Enumerate users with specific range
wpscan --url http://target.com --enumerate u1-10

# All user enumeration methods
wpscan --url http://target.com --enumerate u --enumerate ap --enumerate at
```

## Enumeration Options

### Plugins
```bash
# Enumerate vulnerable plugins
wpscan --url http://target.com --enumerate vp

# Enumerate all plugins (popular + installed)
wpscan --url http://target.com --enumerate ap

# Enumerate popular plugins
wpscan --url http://target.com --enumerate p

# Most aggressive plugin enumeration
wpscan --url http://target.com --enumerate ap --plugins-detection aggressive
```

### Themes
```bash
# Enumerate vulnerable themes
wpscan --url http://target.com --enumerate vt

# Enumerate all themes
wpscan --url http://target.com --enumerate at

# Enumerate popular themes
wpscan --url http://target.com --enumerate t
```

### Comprehensive Enumeration
```bash
# Enumerate everything
wpscan --url http://target.com --enumerate u,vp,vt,tt,cb,dbe

# Explanation of options:
# u   = users
# vp  = vulnerable plugins
# ap  = all plugins  
# p   = popular plugins
# vt  = vulnerable themes
# at  = all themes
# t   = popular themes  
# tt  = timthumbs
# cb  = config backups
# dbe = database exports
```

## Authentication and Brute Force

### Brute Force Attack
```bash
# Basic brute force
wpscan --url http://target.com --usernames admin --passwords passwords.txt

# Multiple usernames
wpscan --url http://target.com --usernames admin,administrator,user --passwords rockyou.txt

# User enumeration + brute force
wpscan --url http://target.com --enumerate u --passwords passwords.txt

# Custom wordlists
wpscan --url http://target.com --usernames users.txt --passwords passwords.txt
```

### Authentication Options
```bash
# HTTP basic auth
wpscan --url http://target.com --basic-auth username:password

# Custom cookies  
wpscan --url http://target.com --cookie "session=abc123"

# Custom headers
wpscan --url http://target.com --headers "Authorization: Bearer token"
```

## Advanced Options

### Output and Reporting
```bash
# Output to file
wpscan --url http://target.com --output results.txt

# JSON output
wpscan --url http://target.com --format json --output results.json

# XML output  
wpscan --url http://target.com --format xml --output results.xml

# CLI format (default)
wpscan --url http://target.com --format cli
```

### Performance and Stealth
```bash
# Increase request timeout
wpscan --url http://target.com --request-timeout 60

# Limit concurrent requests
wpscan --url http://target.com --max-threads 5

# Random User-Agent
wpscan --url http://target.com --random-user-agent

# Custom User-Agent
wpscan --url http://target.com --user-agent "Custom Agent"

# Throttle requests (stealth)
wpscan --url http://target.com --throttle 1000
```

### Proxy and Network
```bash
# Use proxy
wpscan --url http://target.com --proxy http://127.0.0.1:8080

# Proxy with authentication
wpscan --url http://target.com --proxy http://user:pass@proxy:8080

# Disable SSL verification
wpscan --url https://target.com --disable-tls-checks

# Follow redirects
wpscan --url http://target.com --follow-redirection
```

## API Token Integration
```bash
# Get API token from wpvulndb.com (now WPScan Vulnerability Database)
# Register at https://wpscan.com/register

# Use API token for vulnerability data
wpscan --url http://target.com --api-token YOUR_API_TOKEN

# Save API token in config
echo "api_token: YOUR_API_TOKEN" > ~/.wpscan/cli_options.yml
```

## Practical Examples

### Complete WordPress Security Assessment
```bash
# Comprehensive scan with API token
wpscan \
  --url http://target.com \
  --api-token YOUR_TOKEN \
  --enumerate u,vp,vt,tt,cb,dbe \
  --plugins-detection aggressive \
  --random-user-agent \
  --output comprehensive-scan.json \
  --format json
```

### Stealthy Reconnaissance
```bash
# Low-profile scan
wpscan \
  --url http://target.com \
  --enumerate u \
  --detection-mode passive \
  --random-user-agent \
  --throttle 2000 \
  --max-threads 1
```

### Focused Vulnerability Assessment
```bash
# Target specific version vulnerabilities
wpscan \
  --url http://target.com \
  --api-token YOUR_TOKEN \
  --enumerate vp,vt \
  --plugins-detection aggressive
```

## Common WordPress Security Issues

### Default Credentials
```bash
# Test common WordPress credentials
wpscan --url http://target.com --usernames admin --passwords <(echo -e "admin\npassword\n123456\nwp123")
```

### Information Disclosure
- **Version disclosure** in meta tags, readme files
- **Plugin/theme disclosure** through directory listing  
- **User enumeration** via author pages, JSON API
- **Configuration files** exposed (wp-config.php backups)

### Vulnerable Components
- **Outdated WordPress core**
- **Vulnerable plugins** with known CVEs
- **Vulnerable themes** with security flaws
- **Timthumb vulnerabilities** in themes

## Output Analysis

### Interpreting Results
```bash
# Parse JSON output
cat results.json | jq '.plugins[] | select(.vulnerabilities != null)'

# Extract usernames
cat results.json | jq '.users[].username'

# Find high-severity vulnerabilities  
cat results.json | jq '.plugins[].vulnerabilities[] | select(.fixed_in == null)'
```

### Vulnerability Prioritization
1. **WordPress core vulnerabilities** (highest priority)
2. **Actively exploited plugin vulnerabilities**
3. **Authentication bypass vulnerabilities**
4. **Remote code execution vulnerabilities**  
5. **SQL injection vulnerabilities**

## Defense Recommendations

### WordPress Hardening
```php
// wp-config.php security settings
define('DISALLOW_FILE_EDIT', true);
define('WP_DEBUG', false);
define('FORCE_SSL_ADMIN', true);

// Disable XML-RPC
add_filter('xmlrpc_enabled', '__return_false');

// Hide WordPress version
remove_action('wp_head', 'wp_generator');
```

### Security Plugins
- **Wordfence** - Comprehensive security suite
- **Sucuri** - Malware detection and cleanup
- **iThemes Security** - Security hardening
- **All In One WP Security** - Security configurations

## Automation and Integration

### Bash Script Integration
```bash
#!/bin/bash
# WordPress security scanner script

sites=(
    "http://site1.com"
    "http://site2.com"
    "http://site3.com"
)

for site in "${sites[@]}"; do
    echo "Scanning $site..."
    wpscan \
        --url "$site" \
        --api-token "$WPSCAN_API_TOKEN" \
        --enumerate vp,vt,u \
        --format json \
        --output "${site##*/}-scan.json"
done
```

### Continuous Monitoring
```bash
# Cron job for regular WordPress monitoring
# Add to crontab: 0 2 * * * /path/to/wp-monitor.sh

#!/bin/bash
wpscan --url http://target.com --api-token $API_TOKEN --enumerate vp,vt > /var/log/wpscan/daily-scan.log
```

## Related Tools
- [[Burpsuite]] - Manual WordPress testing
- [[nikto]] - General web vulnerability scanner  
- [[sqlmap]] - SQL injection testing
- [[gobuster]] - Directory/file discovery
- [[whatweb]] - Technology identification

## Related Topics
- [[WordPress security]]
- [[Web application security]]
- [[CMS vulnerabilities]]
- [[Plugin vulnerabilities]]
- [[Brute force attacks]]
