# Fine-Grained Password Policy (PSO)

## Overview
Created a stricter password policy specifically for
privileged accounts using Fine-Grained Password Policies.
Different account types get different security levels
based on their risk level.

## Why Fine-Grained Policies Are Needed
Default Domain Policy applies same rules to everyone.
Privileged accounts need stricter controls because:
- Compromising one admin account = massive damage
- Higher privilege = higher risk = stricter requirements

## Policy Comparison
| Setting | Regular Users | Domain Admins |
|---------|--------------|---------------|
| Min password length | 15 chars | 20 chars |
| Max password age | 90 days | 30 days |
| Lockout threshold | 10 attempts | 3 attempts |
| Lockout duration | 30 minutes | Until admin unlocks |
| Precedence | 100 | 10 |

## Implementation

### Via GUI (Active Directory Administrative Center)
Start → Active Directory Administrative Center
→ secure (local)
→ System folder
→Password Settings Container
→NNew → Password Settings
→Fill in settings
→Add Domain Admins group
→OK

### Via PowerShell
\`\`\`powershell
New-ADFineGrainedPasswordPolicy \`
-Name "AdminPolicy" \`
-Precedence 10 \`
-MinPasswordLength 20 \`
-MaxPasswordAge "30.00:00:00" \`
-MinPasswordAge "1.00:00:00" \`
-PasswordHistoryCount 24 \`
-ComplexityEnabled $true \`
-LockoutThreshold 3 \`
-LockoutDuration "00:00:00" \`
-LockoutObservationWindow "00:30:00"

Add-ADFineGrainedPasswordPolicySubject \`
-Identity "AdminPolicy" \`
-Subjects "Domain Admins"
\`\`\`

## Verification
\`\`\`powershell
Get-ADFineGrainedPasswordPolicy -Filter * |
Select Name, MinPasswordLength, LockoutThreshold, Precedence

Name        MinPasswordLength LockoutThreshold Precedence
----        ----------------- ---------------- ----------
AdminPolicy                20                3         10
\`\`\`

## Challenge Faced
- Initial creation failed with "modification not permitted"
- Fix: Had to grant permissions on Password Settings Container
- Created policy manually via ADAC GUI instead

## Key Lesson
- Lower precedence number = higher priority
- Only apply to GROUPS not individual users
- Never apply to service accounts or computer accounts
