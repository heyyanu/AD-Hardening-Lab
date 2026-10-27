# Security Options

## Overview
Configured critical Security Options via Group Policy to
control low-level Windows authentication and protocol
behaviour. These settings fix weak Windows defaults that
attackers exploit to perform credential theft and relay attacks.

## What are Security Options
Deep core Windows settings that control HOW the system
authenticates and behaves. Regular users cannot see or
change these — only admins can configure them via GPO.

Think of them like engine settings in a car:
- Not visible on the surface
- But control how everything fundamentally works
- Wrong settings = serious security risk

## Settings Configured

### 1. LAN Manager Authentication Level
| Setting | Value |
|---------|-------|
| Before | Level 2 — Send NTLM only |
| After | Level 5 — Send NTLMv2 only, Refuse LM and NTLM |
| Attack defeated | NTLM relay, Pass the Hash |

### 2. SMB Signing
| Setting | Value |
|---------|-------|
| Server signing | Always enabled |
| Client signing | Always enabled |
| Attack defeated | SMB relay attacks |

### 3. Anonymous Enumeration
| Setting | Value |
|---------|-------|
| Before | Allowed |
| After | Blocked |
| Attack defeated | AD reconnaissance |

### 4. Hide Last Username
| Setting | Value |
|---------|-------|
| Before | Username displayed |
| After | Username hidden |
| Attack defeated | Username harvesting |

### 5. WDigest Authentication
| Setting | Value |
|---------|-------|
| Before | Enabled |
| After | Disabled |
| Attack defeated | Plaintext credential theft |

## Implementation

### Via GPO
```text
Group Policy Management
→ Default Domain Controllers Policy
→ Edit
→ Computer Configuration
→ Policies → Windows Settings
→ Security Settings → Local Policies
→ Security Options
→ Configure each setting
```

### Via PowerShell
```powershell
# Set LAN Manager level to 5
Set-ItemProperty `
    -Path "HKLM:\SYSTEM\CurrentControlSet\Control\Lsa" `
    -Name "LmCompatibilityLevel" -Value 5

# Enable SMB Signing
Set-SmbServerConfiguration `
    -RequireSecuritySignature $true `
    -EnableSecuritySignature $true -Force

# Block anonymous enumeration
Set-ItemProperty `
    -Path "HKLM:\SYSTEM\CurrentControlSet\Control\Lsa" `
    -Name "RestrictAnonymous" -Value 1

# Hide last username
Set-ItemProperty `
    -Path "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System" `
    -Name "DontDisplayLastUserName" -Value 1

# Disable WDigest
Set-ItemProperty `
    -Path "HKLM:\SYSTEM\CurrentControlSet\Control\SecurityProviders\WDigest" `
    -Name "UseLogonCredential" -Value 0
```

## Verification
```powershell
[PSCustomObject]@{
    LM_Auth_Level      = (Get-ItemProperty "HKLM:\SYSTEM\CurrentControlSet\Control\Lsa").LmCompatibilityLevel
    SMB_Signing        = (Get-SmbServerConfiguration).RequireSecuritySignature
    Anonymous_Enum     = (Get-ItemProperty "HKLM:\SYSTEM\CurrentControlSet\Control\Lsa").RestrictAnonymous
    Hide_Last_Username = (Get-ItemProperty "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System").DontDisplayLastUserName
    WDigest            = (Get-ItemProperty "HKLM:\SYSTEM\CurrentControlSet\Control\Lsa").NoLMHash
} | Format-List
```

Example verification output (as documented):
```text
LM_Auth_Level      : 5     
SMB_Signing        : True  
Anonymous_Enum     : 1     
Hide_Last_Username : 1     
WDigest            : 1     
```

## Key Lessons
- Security Options fix weak Windows defaults
- LM Auth Level 5 blocks all NTLM relay attacks
- SMB Signing must be enabled on both client AND server
- WDigest disabled prevents Mimikatz plaintext extraction
- Always verify settings via registry after GPO update

## Security Impact
| Attack | Before | After |
|--------|--------|-------|
| NTLM Relay | Possible | Blocked |
| Pass the Hash | Possible | Harder |
| SMB Relay | Possible | Blocked |
| Credential Harvesting | Easy | Blocked |
| AD Reconnaissance | Easy | Blocked |

