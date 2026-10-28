# Kerberoasting Attack Simulation

## Overview
Tested Kerberoasting attack against the AD environment
before and after hardening to demonstrate the effectiveness
of AES256 encryption and gMSA deployment in defeating
this attack technique.

## What is Kerberoasting
Kerberoasting is an attack that targets service accounts
with Service Principal Names (SPNs) registered in AD.

### Attack Flow
```
Step 1 → Attacker authenticates to AD
          (needs any valid domain account)

Step 2 → Requests Kerberos service ticket
          for target service account

Step 3 → AD returns ticket encrypted
          with service account password hash

Step 4 → Attacker takes ticket offline
          No further network activity needed

Step 5 → Cracks ticket using hashcat/john
          RC4 = crackable in hours
          AES256 = practically impossible
```

## Tools Used
- Rubeus (ticket extraction)
- Hashcat (offline cracking)
- PowerView (SPN enumeration)

## Finding Kerberoastable Accounts

### Before Hardening
```powershell
# Find accounts with SPNs registered
Get-ADUser -Filter 'ServicePrincipalName -like "*"' `
    -Properties ServicePrincipalName |
    Select-Object Name, SamAccountName, ServicePrincipalName

# Result — service account found:
Name       SamAccountName  ServicePrincipalName
----       --------------  --------------------
svc-webapp svc-webapp      HTTP/webapp.YOUR-DOMAIN.local
```

## Attack Results

### Before Hardening (RC4 Encryption)
```
Kerberoasting Result: VULNERABLE
Encryption: RC4
Time to crack: Hours to days
Risk level: CRITICAL
```

### After Hardening (AES256 Encryption)
```
Kerberoasting Result: RESISTANT
Encryption: AES256
Time to crack: Millions of years
Risk level: LOW
```

## Hardening Controls That Defeated This Attack

### 1. AES256 Encryption Enforcement
```powershell
# Set AES on service account
Set-ADUser -Identity "svc-webapp" `
    -KerberosEncryptionType AES128, AES256

# Verify
Get-ADUser "svc-webapp" -Properties KerberosEncryptionType |
    Select-Object -ExpandProperty KerberosEncryptionType
# Result: {AES128, AES256} ✅
```

### 2. Replace Service Account with gMSA
```powershell
# Create gMSA — password auto-rotates every 30 days
# 128-character random password = uncrackable
New-ADServiceAccount -Name "svc-webapp-gmsa" `
    -DNSHostName "webapp.YOUR-DOMAIN.local" `
    -PrincipalsAllowedToRetrieveManagedPassword "WebServers"
```

### 3. Remove Unnecessary SPNs
```powershell
# Remove SPN if service account no longer needs it
Set-ADUser -Identity "svc-webapp" -ServicePrincipalNames @{
    Remove = "HTTP/webapp.YOUR-DOMAIN.local"
}
```

## Detection — Event IDs to Monitor
| Event ID | Meaning | Action |
|----------|---------|--------|
| 4769 | Kerberos service ticket requested | Alert on RC4 ticket requests |
| 4770 | Kerberos ticket renewed | Monitor frequency |

```powershell
# Monitor for Kerberoasting attempts
# Look for Event 4769 with RC4 encryption (0x17)
$rc4Events = Get-WinEvent -FilterHashtable @{
    LogName = 'Security'
    Id = 4769
} -MaxEvents 100

$rc4Events |
    Where-Object { $_.Message -match 'Ticket Encryption Type:\s+0x17' } |
    Select-Object TimeCreated, Id, Message
```

## Key Lessons
- Any valid domain user can perform Kerberoasting
- Service accounts with SPNs are high value targets
- AES256 makes cracking practically impossible
- gMSAs eliminate Kerberoasting risk completely
- Monitor Event ID 4769 for RC4 ticket requests
- Principle of least privilege reduces attack surface
