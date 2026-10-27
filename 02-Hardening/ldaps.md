# Secure LDAP (LDAPS)

## Overview
Configured LDAPS to encrypt all Active Directory
directory traffic using TLS on port 636. This prevents
attackers from capturing credentials and AD data
transmitted in plaintext over the network.

## LDAP vs LDAPS
| Feature | LDAP | LDAPS |
|---------|------|-------|
| Port | 389 | 636 |
| Encryption | None | TLS |
| Credentials | Plaintext | Encrypted |
| Attack risk | High | Low |
| Relay attacks | Possible | Blocked |

## Why LDAPS Matters
Without LDAPS:
- Attacker runs Wireshark on network
- Captures ALL LDAP traffic
- Sees usernames and passwords in plaintext
- LDAP relay attacks possible

With LDAPS:
- All traffic encrypted with TLS
- Attacker captures only encrypted data
- Cannot read credentials
- Relay attacks fail

## Prerequisites
- Active Directory Certificate Services (AD CS)
- Enterprise Root CA installed
- Certificate with Server Authentication EKU

## Implementation

### Step 1 — Install AD CS
\`\`\`powershell
Install-WindowsFeature ADCS-Cert-Authority \`
-IncludeManagementTools
\`\`\`

### Step 2 — Configure Enterprise Root CA
\`\`\`powershell
Install-AdcsCertificationAuthority \`
-CAType EnterpriseRootCa \`
-CaCommonName "SecureLabCA" \`
-KeyLength 2048 \`
-HashAlgorithmName SHA256 \`
-ValidityPeriod Years \`
-ValidityPeriodUnits 5 \`
-Force
\`\`\`

### Step 3 — Restart Certificate Service
\`\`\`powershell
Restart-Service CertSvc
Start-Sleep -Seconds 15
\`\`\`

### Step 4 — Enable LDAP Signing via GPO
\`\`\`
Group Policy Management
→ Default Domain Controllers Policy
→ Edit
→ Computer Configuration
→ Windows Settings → Security Settings
→ Local Policies → Security Options
→ Domain controller: LDAP server signing requirements
→ Set to: Require signing
\`\`\`

## Verification
\`\`\`powershell
# Test LDAPS port 636 is listening
Test-NetConnection -ComputerName "YOUR-DC-HOSTNAME" -Port 636

ComputerName     : YOUR-DC-HOSTNAME
RemotePort       : 636
TcpTestSucceeded : True

# Check certificate installed on DC
Get-ChildItem Cert:\LocalMachine\My |
Select Subject, Issuer, NotAfter |
Format-Table -AutoSize

Subject        Issuer         NotAfter
-------        ------         --------
CN=SecureLabCA CN=SecureLabCA 10/25/2031
\`\`\`

## Security Impact
| Attack | Before LDAPS | After LDAPS |
|--------|-------------|------------|
| Credential interception | Easy | Blocked |
| LDAP relay | Possible | Blocked |
| Directory data theft | Easy | Blocked |
| Pass the credential | Possible | Blocked |

## Key Lessons
- LDAPS requires AD CS to be installed first
- Certificate must have Server Authentication EKU
- Enabling LDAPS does NOT disable plain LDAP automatically
- Must also enable LDAP signing to block unsigned connections
- Port 636 must be open in firewall for LDAPS to work
- Verify with Test-NetConnection after configuration

