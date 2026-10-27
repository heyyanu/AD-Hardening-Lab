# Local Administrator Password Solution (LAPS)

## Overview
Deployed Windows LAPS to automatically generate and rotate
unique local administrator passwords for each domain-joined
machine. This eliminates the most common lateral movement
technique — using one shared local admin password to access
every machine in the domain.

## Problem LAPS Solves
Without LAPS:
- Every machine has same local admin password
- Attacker compromises one machine
- Uses same password on ALL other machines
- Entire network compromised instantly

With LAPS:
- Every machine has UNIQUE random password
- Password rotates automatically every 30 days
- Attacker compromises one machine
- Password does NOT work anywhere else
- Lateral movement blocked

## Implementation

### Step 1 — Install LAPS Module
\`\`\`powershell
Import-Module LAPS
Get-Command -Module LAPS
\`\`\`

### Step 2 — Extend AD Schema
\`\`\`powershell
Update-LapsADSchema
# Answer Y to all prompts
# Adds these attributes to AD:
# ms-LAPS-Password
# ms-LAPS-PasswordExpirationTime
# ms-LAPS-EncryptedPassword
# ms-LAPS-EncryptedPasswordHistory
# ms-LAPS-EncryptedDSRMPassword
\`\`\`

### Step 3 — Set Computer Self Permissions
\`\`\`powershell
Set-LapsADComputerSelfPermission \`
-Identity "OU=Domain Controllers,DC=YOUR-DOMAIN,DC=local"
\`\`\`

### Step 4 — Configure GPO
\`\`\`
Group Policy Management
→ Create GPO: "LAPS Policy"
→ Computer Configuration
→ Administrative Templates
→ System → LAPS
→ Configure password backup directory: Active Directory
→ Password Settings: Length 20, Age 30 days
→ Enable password encryption: Enabled
→ Link GPO to all OUs
\`\`\`

### Step 5 — Configure via Registry
\`\`\`powershell
Set-GPRegistryValue -Name "LAPS Policy" \`
-Key "HKLM\SOFTWARE\Policies\Microsoft\Windows\LAPS" \`
-ValueName "BackupDirectory" \`
-Type DWord -Value 2

Set-GPRegistryValue -Name "LAPS Policy" \`
-Key "HKLM\SOFTWARE\Policies\Microsoft\Windows\LAPS" \`
-ValueName "ADBackupDSRMPassword" \`
-Type DWord -Value 1
\`\`\`

## Verification
\`\`\`powershell
Get-LapsADPassword -Identity "YOUR-DC-NAME" -AsPlainText

ComputerName       : YOUR-DC-NAME
Account            : Administrator
Password           : **REDACTED**
PasswordUpdateTime : 10/25/2026 8:32:57 AM
ExpirationTimestamp: 11/24/2026 8:32:57 AM
Source             : EncryptedDSRMPassword
DecryptionStatus   : Success
\`\`\`

## Challenges Faced
- Windows Server 2019 does not have LAPS built in
- Had to install legacy LAPS MSI first
- GPO ADMX template needed manual verification
- BackupDirectory registry path was incorrect initially
- Required BackupDirectory value 2 not 1 for DC password backup
- Multiple troubleshooting steps needed to get password generated

## Key Lessons
- Windows LAPS and Legacy LAPS use different registry paths
- DCs require BackupDSRMPassword setting enabled
- Always verify with Get-LapsADPassword after configuration
- Link GPO to Domain Controllers OU separately
- Check LAPS event log for errors:
  Event ID 10004 = Success
  Event ID 10005 = Failed
  Event ID 10018 = Password updated in AD

## Security Impact
| Before LAPS | After LAPS |
|------------|-----------|
| Shared password across all machines | Unique password per machine |
| Never rotated | Rotates every 30 days |
| Lateral movement easy | Lateral movement blocked |
| Manual management | Fully automated |
