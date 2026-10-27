# Advanced Audit Policy

## Overview
Configured Advanced Audit Policy to record all critical
security events across the domain. Audit policy is the
early warning system for Active Directory — even the best
hardened environment needs detection when prevention fails.

## Why Audit Policy Matters
Prevention controls = stop attacks
Audit policy = detect attacks

Without audit policy:
→ Attacker breaks in
→ Does whatever they want
→ Leaves no trace
→ You never know 💀

With audit policy:
→ Attacker breaks in
→ Every action recorded
→ Alert fires immediately
→ You catch them fast ✅


## Basic vs Advanced Audit Policy
| Feature | Basic | Advanced |
|---------|-------|----------|
| Categories | 9 | 53 |
| Granularity | Low | High |
| Recommended | No | Yes |
| Location | Local Policies | Advanced Audit Policy |

## Critical Categories Configured

### Account Logon
| Subcategory | Setting | Event IDs | Detects |
|-------------|---------|-----------|---------|
| Credential Validation | Success + Failure | 4776 | Password spray |

### Account Management
| Subcategory | Setting | Event IDs | Detects |
|-------------|---------|-----------|---------|
| User Account Management | Success + Failure | 4720 | New account created |
| Security Group Management | Success + Failure | 4728 | Added to privileged group |

### Logon/Logoff
| Subcategory | Setting | Event IDs | Detects |
|-------------|---------|-----------|---------|
| Logon | Success + Failure | 4624, 4625 | Failed login attempts |
| Special Logon | Success | 4672 | Admin privilege used |

### DS Access
| Subcategory | Setting | Event IDs | Detects |
|-------------|---------|-----------|---------|
| Directory Service Access | Success + Failure | 4662 | AD object accessed |
| Directory Service Changes | Success + Failure | 5136 | AD object modified |

### Policy Change
| Subcategory | Setting | Event IDs | Detects |
|-------------|---------|-----------|---------|
| Authentication Policy Change | Success | 4713 | Kerberos policy changed |

## Key Event IDs Reference
| Event ID | Meaning | Attack Detected |
|----------|---------|----------------|
| 4624 | Successful login | Baseline |
| 4625 | Failed login | Password spray |
| 4720 | Account created | Attacker persistence |
| 4728 | Added to privileged group | Privilege escalation |
| 4648 | Explicit credential logon | Pass the Hash |
| 4672 | Admin privileges used | Privilege abuse |
| 4771 | Kerberos pre-auth failed | Password spray |
| 5136 | AD object modified | DCSync / persistence |
| 7045 | New service installed | Malware installation |

## Implementation

### Via PowerShell
```powershell
# Enable all critical audit categories
auditpol /set /subcategory:"Credential Validation" `
    /success:enable /failure:enable

auditpol /set /subcategory:"User Account Management" `
    /success:enable /failure:enable

auditpol /set /subcategory:"Security Group Management" `
    /success:enable /failure:enable

auditpol /set /subcategory:"Logon" `
    /success:enable /failure:enable

auditpol /set /subcategory:"Special Logon" `
    /success:enable /failure:enable

auditpol /set /subcategory:"Directory Service Access" `
    /success:enable /failure:enable

auditpol /set /subcategory:"Directory Service Changes" `
    /success:enable /failure:enable

auditpol /set /subcategory:"Authentication Policy Change" `
    /success:enable /failure:enable
```

### Via GPO
```text
Group Policy Management
→ Default Domain Controllers Policy
→ Edit
→ Computer Configuration
→ Policies → Windows Settings
→ Security Settings
→ Advanced Audit Policy Configuration
→ Configure each subcategory
```

## Verification
```powershell
# Check all audit settings
auditpol /get /category:* |
Where-Object {$_ -notlike "*No Auditing*"}

# Check specific category
auditpol /get /subcategory:"Logon"

# Verify by generating test event
# Failed logon attempt generates Event ID 4625
# Check in Event Viewer:
# Windows Logs → Security
```

## Monitoring These Events
```powershell
# Check recent failed logons
Get-WinEvent -FilterHashtable @{
    LogName = 'Security'
    Id = 4625
} -MaxEvents 10 |
Select-Object TimeCreated, Message |
Format-List

# Check privileged group changes
Get-WinEvent -FilterHashtable @{
    LogName = 'Security'
    Id = 4728
} -MaxEvents 10 |
Select-Object TimeCreated, Message |
Format-List
```

## Key Lessons
- Audit policy alone is not enough
- Need SIEM to collect and analyse logs
- Set up alerting rules for suspicious patterns
- Review logs regularly for anomalies
- Advanced Audit Policy overrides Basic Audit Policy
- Always audit both Success AND Failure for critical events
