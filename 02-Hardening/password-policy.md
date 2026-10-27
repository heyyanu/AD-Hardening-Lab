# Password & Account Lockout Policy

## Overview
Configured hardened password and account lockout policies
via Default Domain Policy GPO to protect against brute
force and password spray attacks.

## Settings Configured

### Password Policy
| Setting | Value | Attack Defeated |
|---------|-------|----------------|
| Minimum password length | 15 characters | Brute force |
| Maximum password age | 90 days | Stolen passwords |
| Minimum password age | 1 day | Password cycling |
| Password history | 24 passwords | Password reuse |
| Complexity enabled | Yes | Dictionary attacks |

### Account Lockout Policy
| Setting | Value | Attack Defeated |
|---------|-------|----------------|
| Lockout threshold | 10 attempts | Password spray |
| Lockout duration | 30 minutes | Brute force |
| Observation window | 30 minutes | Slow brute force |

## Implementation

### Via PowerShell
\`\`\`powershell
Set-ADDefaultDomainPasswordPolicy \`
-Identity "YOUR-DOMAIN.local" \`
-MinPasswordLength 15 \`
-MaxPasswordAge "90.00:00:00" \`
-MinPasswordAge "1.00:00:00" \`
-PasswordHistoryCount 24 \`
-ComplexityEnabled $true \`
-LockoutThreshold 10 \`
-LockoutDuration "00:30:00" \`
-LockoutObservationWindow "00:30:00"
\`\`\`

### Via GUI
Group Policy Management
→ Default Domain Policy → Edit
→ Computer Configuration
→ Windows Settings
→ Security Settings
→ Account Policies
→ Password Policy
→ Configure each setting

## Verification
\`\`\`powershell
Get-ADDefaultDomainPasswordPolicy

ComplexityEnabled    : True
LockoutDuration      : 00:30:00
LockoutThreshold     : 10
MaxPasswordAge       : 90.00:00:00
MinPasswordAge       : 1.00:00:00
MinPasswordLength    : 15
PasswordHistoryCount : 24
\`\`\`
