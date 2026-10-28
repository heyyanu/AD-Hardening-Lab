# PingCastle Security Assessment Report

## Overview
Used PingCastle to assess the security posture of the
Active Directory environment before and after hardening.
PingCastle scores AD security on a scale of 0-100 where
LOWER score = MORE secure (like golf).

## Tool Information
```
Tool: PingCastle
Version: 4.0.0.31
Website: www.pingcastle.com
Cost: Free (basic edition)
Run as: Domain Admin on DC
```

## How to Run PingCastle
```powershell
# Download latest version
Invoke-WebRequest -UseBasicParsing `
    -Uri "https://github.com/vletoux/pingcastle/releases/download/4.0.0.31/PingCastle_4.0.0.31.zip" `
    -OutFile "C:\Tools\PingCastle_4.0.zip"

# Extract
Expand-Archive -Path "C:\Tools\PingCastle_4.0.zip" `
    -DestinationPath "C:\Tools\PingCastle" -Force

# Run health check
cd C:\Tools\PingCastle
.\PingCastle.exe --healthcheck
```

## Assessment Categories
PingCastle checks four main risk categories:

| Category | What it checks |
|----------|---------------|
| Stale Objects | Old unused accounts and computers |
| Privileged Accounts | Too many admins, risky memberships |
| Trusts | Domain trust relationship risks |
| Anomalies | Misconfigurations and vulnerabilities |

## Before Hardening Assessment

### Overall Score
```
Score: TBD / 100
Date: Before hardening
Status: Baseline measurement
```

### Key Findings Before Hardening
```
🔴 Critical Risks:
→ No account lockout policy configured
→ Weak password policy (8 chars minimum)
→ No audit logging configured
→ SMB signing not required
→ NTLM not restricted

🟠 High Risks:
→ No Fine-Grained Password Policies
→ LAPS not deployed
→ Protected Users group empty
→ Kerberos pre-auth not enforced

🟡 Medium Risks:
→ No LDAPS configured
→ Legacy protocols enabled
→ Debug rights not removed
```

## After Hardening Assessment

### Overall Score
```
Score: TBD / 100
Date: After hardening
Status: Hardened measurement
Improvement: TBD points
```

### Key Improvements After Hardening
```
✅ Fixed Critical Risks:
→ Account lockout: 10 attempts, 30 min
→ Password policy: 15 chars minimum
→ Audit logging: All critical events
→ SMB signing: Required on all machines
→ NTLM: Level 5 — NTLMv2 only

✅ Fixed High Risks:
→ Fine-Grained PSO for Domain Admins
→ LAPS deployed on all OUs
→ Protected Users: All admins added
→ Kerberos pre-auth: Enabled all accounts

✅ Fixed Medium Risks:
→ LDAPS: Port 636 active with certificate
→ Legacy protocols: LLMNR disabled
→ Debug rights: Removed from all accounts
```

## Security Controls Verification

### Password Policy
```powershell
Get-ADDefaultDomainPasswordPolicy

ComplexityEnabled    : True  ✅
LockoutThreshold     : 10    ✅
MaxPasswordAge       : 90    ✅
MinPasswordLength    : 15    ✅
PasswordHistoryCount : 24    ✅
```

### Protocol Security
```powershell
[PSCustomObject]@{
    SMBv1_Disabled  = $true   ✅
    SMB_Signing     = $true   ✅
    NTLM_Level      = 5       ✅
    LDAPS_Active    = $true   ✅
    LLMNR_Disabled  = $true   ✅
    WDigest_Off     = $true   ✅
}
```

### Privileged Accounts
```powershell
# Domain Admins — minimal membership
Get-ADGroupMember "Domain Admins" |
    Select-Object Name
# Only essential accounts ✅

# Protected Users — all admins added
Get-ADGroupMember "Protected Users" |
    Select-Object Name
# Administrator, itadmin ✅
```

### LAPS Verification
```powershell
Get-LapsADPassword -Identity "YOUR-DC-HOSTNAME" -AsPlainText

ComputerName       : YOUR-DC-HOSTNAME
Account            : Administrator
Password           : **REDACTED**
ExpirationTimestamp: 30 days from deployment
DecryptionStatus   : Success ✅
```

### LDAPS Verification
```powershell
Test-NetConnection -ComputerName "YOUR-DC-HOSTNAME" -Port 636

TcpTestSucceeded : True ✅
```

### Kerberos Verification
```powershell
Get-ADUser -Filter * -Properties KerberosEncryptionType |
    Select-Object Name, KerberosEncryptionType

# All users showing AES128 and AES256 ✅

Get-ADUser -Filter {DoesNotRequirePreAuth -eq $true} |
    Select-Object Name
# No results — all accounts require pre-auth ✅
```

## Recommendations for Further Hardening
```
1. Deploy Microsoft Defender for Identity
   → Detects Golden Ticket attacks
   → Detects DCSync attempts
   → Detects password spray

2. Implement Privileged Access Workstations
   → Dedicated machines for admin tasks
   → No internet or email access
   → Enhanced logging enabled

3. Enable Azure AD Password Protection
   → Blocks commonly used passwords
   → Blocks company name variations
   → Works for on-premises AD

4. Regular KRBTGT Password Rotation
   → Rotate every 180 days minimum
   → Must rotate TWICE each time
   → Invalidates Golden Tickets

5. Implement Just-In-Time Access
   → Use Microsoft Entra PIM
   → Admin rights granted temporarily
   → Automatically removed after task
```
