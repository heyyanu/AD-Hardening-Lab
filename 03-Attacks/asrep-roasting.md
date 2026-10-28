# AS-REP Roasting Attack Simulation

## Overview
Tested AS-REP Roasting attack against the AD environment
to demonstrate how enabling Kerberos pre-authentication
completely defeats this attack technique.

## What is AS-REP Roasting
AS-REP Roasting exploits accounts where Kerberos
pre-authentication is disabled. An attacker can request
a password hash from AD WITHOUT knowing the password
and crack it offline.

### Why Pre-Authentication Matters
\`\`\`
With Pre-Authentication (secure):
Step 1 → User sends encrypted timestamp to KDC
Step 2 → KDC verifies timestamp using password hash
Step 3 → KDC confirms user knows password
Step 4 → KDC issues Kerberos ticket ✅

Without Pre-Authentication (vulnerable):
Step 1 → Attacker requests AS-REP for any account
Step 2 → KDC sends back encrypted response
          WITHOUT verifying attacker knows password
Step 3 → Attacker takes encrypted response offline
Step 4 → Cracks it to find password 💀
\`\`\`

### Difference from Kerberoasting
| Feature | Kerberoasting | AS-REP Roasting |
|---------|--------------|-----------------|
| Requires auth | Yes | No |
| Targets | SPNs | Pre-auth disabled |
| Gets | Service ticket | AS-REP hash |
| Needs | Valid account | Nothing |
| Impact | Service accounts | Any account |

## Attack Flow
\`\`\`
Step 1 → Find accounts with pre-auth disabled
Step 2 → Send AS-REQ without credentials
Step 3 → KDC returns AS-REP with hash
Step 4 → Extract hash from response
Step 5 → Crack hash offline with hashcat
Step 6 → Get plaintext password 💀
\`\`\`

## Tools Used
- Rubeus (hash extraction)
- Impacket GetNPUsers.py (Linux)
- Hashcat (offline cracking)
- PowerView (account enumeration)

## Finding Vulnerable Accounts

### Check for Pre-Auth Disabled Accounts
\`\`\`powershell
# Find vulnerable accounts
Get-ADUser -Filter {DoesNotRequirePreAuth -eq $true} \`
-Properties DoesNotRequirePreAuth |
Select Name, SamAccountName, DoesNotRequirePreAuth

# Any result here = VULNERABLE ❌
# No results = SECURE ✅
\`\`\`

## Attack Simulation

### Before Hardening
\`\`\`powershell
# Simulate AS-REP Roasting using Rubeus
.\Rubeus.exe asreproast /format:hashcat /outfile:hashes.txt

# Extract hashes for all vulnerable accounts
# Output format for hashcat:
$krb5asrep$23$username@YOUR-DOMAIN.local:HASH...
\`\`\`

### Crack the Hash
\`\`\`bash
# Crack with hashcat
hashcat -m 18200 hashes.txt wordlist.txt

# RC4 hash cracking speed:
# GPU: ~1 billion attempts per second
# Common password found in seconds
\`\`\`

## Attack Results

### Before Hardening
\`\`\`
AS-REP Roasting Result: VULNERABLE
Pre-auth disabled accounts: Found
Hash obtained: YES
Crackable: YES (RC4)
Time to crack: Seconds to minutes
Risk Level: CRITICAL
\`\`\`

### After Hardening
\`\`\`
AS-REP Roasting Result: BLOCKED
Pre-auth disabled accounts: NONE
Hash obtained: NO
Attack possible: NO
Risk Level: NONE ✅
\`\`\`

## Hardening Controls That Defeated This Attack

### 1. Enable Pre-Authentication on All Accounts
\`\`\`powershell
# Find all vulnerable accounts
Get-ADUser \`
-Filter {DoesNotRequirePreAuth -eq $true} \`
-Properties DoesNotRequirePreAuth |
Select Name, SamAccountName

# Fix all vulnerable accounts
Get-ADUser \`
-Filter {DoesNotRequirePreAuth -eq $true} |
ForEach-Object {
    Set-ADAccountControl \`
    -Identity $_.SamAccountName \`
    -DoesNotRequirePreAuth $false
    Write-Host "Fixed: $($_.Name)"
}

# Verify no vulnerable accounts remain
Get-ADUser \`
-Filter {DoesNotRequirePreAuth -eq $true} |
Select Name
# Should return nothing ✅
\`\`\`

### Via GUI
\`\`\`
Active Directory Users and Computers
→ Find each user
→ Right click → Properties
→ Account tab
→ Account options
→ UNCHECK:
   "Do not require Kerberos preauthentication"
→ Apply → OK
\`\`\`

### 2. AES256 Encryption
\`\`\`powershell
# Even if pre-auth somehow disabled
# AES256 makes hash much harder to crack
Set-ADUser -Identity "username" \`
-KerberosEncryptionType AES128,AES256
\`\`\`

### 3. Strong Password Policy
\`\`\`powershell
# 15+ char passwords with complexity
# Makes cracking much harder even if hash obtained
Get-ADDefaultDomainPasswordPolicy |
Select MinPasswordLength, ComplexityEnabled

MinPasswordLength : 15
ComplexityEnabled : True ✅
\`\`\`

## Detection
\`\`\`powershell
# Monitor Event ID 4768 for AS-REP requests
# without pre-authentication
Get-WinEvent -FilterHashtable @{
    LogName = 'Security'
    Id = 4768
} -MaxEvents 20 |
Where-Object {
    $_.Message -like "*0x10*"
} |
Select TimeCreated, Message |
Format-List

# 0x10 in Pre-Authentication Type = No Pre-Auth
# This indicates AS-REP Roasting attempt
\`\`\`

## Prevention Checklist
| Control | Implemented | Effect |
|---------|------------|--------|
| Pre-auth enabled on all accounts | ✅ Yes | Attack impossible |
| AES256 encryption | ✅ Yes | Hash uncrackable |
| Strong password policy | ✅ Yes | Harder to crack |
| Audit logging | ✅ Yes | Attack detected |

## Key Lessons
- AS-REP Roasting requires NO valid credentials
- Pre-authentication disabled is the root cause
- Enabling pre-auth completely defeats the attack
- Regular audits needed to catch newly disabled accounts
- AES256 provides defence in depth
- Monitor Event ID 4768 for pre-auth bypass attempts
- Never disable pre-authentication even temporarily
