# NTLM Relay Attack Simulation

## Overview
Tested NTLM relay attack against the AD environment
to demonstrate the effectiveness of SMB signing and
NTLM restrictions in defeating this common network
attack technique.

## What is NTLM Relay
NTLM relay intercepts authentication from one machine
and forwards it to another server to gain unauthorized
access — without ever knowing the actual password.

### Simple Analogy
\`\`\`
Normal:
You call bank → "I am John, code 1234"
Bank verifies → Access granted

NTLM Relay:
You call bank → attacker intercepts
Attacker calls another bank → "I am John, code 1234"
Second bank believes it → attacker gets access
\`\`\`

### Attack Flow
\`\`\`
Step 1 → Attacker runs Responder tool
          Listens for LLMNR/NetBIOS requests

Step 2 → User types wrong server name
          Example: \\fileservre (typo)
          DNS fails to find it

Step 3 → LLMNR broadcasts request
          "Anyone know fileservre?"

Step 4 → Attacker responds
          "Yes that is me!"

Step 5 → User sends NTLM authentication
          Attacker captures it

Step 6 → Attacker relays to REAL server
          Authenticates as the user

Step 7 → Attacker has access
          Without knowing password 💀
\`\`\`

## Tools Used
- Responder (credential capture)
- ntlmrelayx (credential relay)
- Impacket suite

## Attack Simulation

### Before Hardening Setup
\`\`\`
LLMNR: Enabled
NetBIOS: Enabled
SMB Signing: Disabled
NTLM: Allowed
\`\`\`

### Attacker Commands
\`\`\`bash
# Step 1 — Start Responder
python3 Responder.py -I eth0 -rdwv

# Step 2 — Start ntlmrelayx
python3 ntlmrelayx.py -t smb://YOUR-DC-IP -smb2support

# Step 3 — Wait for user to make typo
# or trigger via other means
\`\`\`

## Attack Results

### Before Hardening
\`\`\`
NTLM Relay Result: VULNERABLE
SMB Signing: Disabled
LLMNR: Enabled
NTLM Level: 2
Result: Credentials captured and relayed
Risk Level: CRITICAL
\`\`\`

### After Hardening
\`\`\`
NTLM Relay Result: BLOCKED
SMB Signing: Required
LLMNR: Disabled
NTLM Level: 5
Result: Relay failed — signature verification failed
Risk Level: LOW
\`\`\`

## Hardening Controls That Defeated This Attack

### 1. SMB Signing Enabled
\`\`\`powershell
# Verify SMB signing required
Get-SmbServerConfiguration |
Select RequireSecuritySignature

RequireSecuritySignature : True ✅

# What this does:
# Every SMB packet is signed
# Attacker cannot forge signature
# Relay fails immediately
\`\`\`

### 2. LLMNR Disabled
\`\`\`powershell
# Verify LLMNR is disabled
Get-ItemProperty \`
"HKLM:\SOFTWARE\Policies\Microsoft\Windows NT\DNSClient" \`
-Name EnableMulticast

EnableMulticast : 0 ✅

# What this does:
# Responder cannot capture requests
# Attack cannot even start
\`\`\`

### 3. NTLM Level Set to 5
\`\`\`powershell
# Verify NTLM level
(Get-ItemProperty \`
"HKLM:\SYSTEM\CurrentControlSet\Control\Lsa").LmCompatibilityLevel

Result: 5 ✅

# What this does:
# Only NTLMv2 allowed
# LM and NTLM refused
# Weaker relay attacks fail
\`\`\`

### 4. Protected Users Group
\`\`\`powershell
# Verify admin accounts in Protected Users
Get-ADGroupMember -Identity "Protected Users" |
Select Name

# What this does:
# Admin accounts use Kerberos only
# No NTLM authentication at all
# Nothing to relay ✅
\`\`\`

## Detection
\`\`\`powershell
# Monitor for NTLM relay attempts
# Look for Event ID 4624 with NTLM auth
# from unexpected sources
Get-WinEvent -FilterHashtable @{
    LogName = 'Security'
    Id = 4624
} -MaxEvents 20 |
Where-Object {$_.Message -like "*NTLM*"} |
Select TimeCreated, Message |
Format-List
\`\`\`

## Key Lessons
- NTLM relay needs no password — just network access
- SMB signing is the most effective countermeasure
- Disabling LLMNR stops attack before it starts
- Protected Users removes NTLM for privileged accounts
- Takes less than 60 seconds without protections
- Monitor for NTLM auth events from unexpected sources
- Combine SMB signing + LLMNR disable + NTLM level 5
  for complete protection

## Complete Protection Checklist
| Control | Status | Attack Impact |
|---------|--------|--------------|
| SMB Signing | ✅ Enabled | Relay fails |
| LLMNR Disabled | ✅ Disabled | Capture fails |
| NTLM Level 5 | ✅ Configured | Weak NTLM refused |
| Protected Users | ✅ Configured | No NTLM for admins |
| LDAP Signing | ✅ Enabled | LDAP relay blocked |
