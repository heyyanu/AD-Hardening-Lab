# Active Directory Hardening Lab — Final Report

## Executive Summary
This report documents the complete process of building,
hardening, and testing an enterprise Active Directory
environment on AWS EC2. The project demonstrates
real-world AD security controls, attack simulations,
challenges faced, and lessons learned.

The lab successfully implemented 11 security controls
across the AD environment, tested 5 attack techniques,
and documented multiple real-world challenges and their
resolutions. The overall security posture was significantly
improved from a baseline of weak default settings to a
hardened enterprise-grade configuration.

## Project Scope
```
Environment:
→ Cloud: Amazon Web Services (AWS)
→ Server: Windows Server 2019 Datacenter
→ Service: Active Directory Domain Services
→ Domain: YOUR-DOMAIN.local
→ Instance: EC2 t2.medium

Duration: 4 days intensive study and implementation

Objectives:
→ Build AD environment from scratch on AWS
→ Implement enterprise security controls
→ Test real attack techniques
→ Document challenges and resolutions
→ Produce professional security report
```

## Infrastructure Setup

### AWS EC2 Configuration
```
Instance Type: t2.medium (2 vCPU, 4GB RAM)
Storage: 50GB SSD
OS: Windows Server 2019 Datacenter
Region: Asia Pacific (Sydney)
Security Group: RDP port 3389 restricted
```

### Active Directory Structure
```
YOUR-DOMAIN.local
├── OU=Domain Controllers
│   └── YOUR-DC-HOSTNAME
├── OU=IT_Department
│   ├── John Smith (jsmith)
│   └── IT Admin (itadmin)
├── OU=HR_Department
│   └── Jane Doe (jdoe)
├── OU=Servers
├── OU=Workstations
└── OU=ServiceAccounts
    └── svc-webapp
```

## Security Controls Implemented

### 1. Password Policy
```
Control: Default Domain Password Policy
Location: Default Domain Policy GPO

Settings:
→ Minimum length: 15 characters
→ Complexity: Enabled
→ Maximum age: 90 days
→ Minimum age: 1 day
→ History: 24 passwords
→ Lockout threshold: 10 attempts
→ Lockout duration: 30 minutes

Attacks Defeated:
→ Brute force attacks
→ Password spray attacks
→ Dictionary attacks
→ Password reuse attacks
```

### 2. Fine-Grained Password Policy
```
Control: Password Settings Object (PSO)
Name: AdminPolicy
Applied to: Domain Admins group

Settings:
→ Minimum length: 20 characters
→ Maximum age: 30 days
→ Lockout threshold: 3 attempts
→ Lockout duration: Until admin unlocks
→ Precedence: 10

Attacks Defeated:
→ Admin account brute force
→ Targeted password attacks
→ Credential stuffing
```

### 3. LAPS (Local Admin Password Solution)
```
Control: Windows LAPS
Applied to: All OUs

Settings:
→ Password length: 20 characters
→ Password age: 30 days
→ Backup directory: Active Directory
→ Encryption: Enabled

Attacks Defeated:
→ Lateral movement via shared local admin password
→ Pass the Hash using local admin credentials
→ Credential reuse across machines
```

### 4. Protected Users Security Group
```
Control: Protected Users Group
Members: Administrator, itadmin

Enforces automatically:
→ Kerberos only — no NTLM
→ No credential caching
→ AES encryption only
→ Tickets expire in 4 hours
→ No delegation

Attacks Defeated:
→ Pass the Hash
→ NTLM relay against admin accounts
→ Pass the Ticket
→ Credential caching theft
```

### 5. LDAPS Configuration
```
Control: Secure LDAP
Port: 636 (TLS encrypted)
Certificate: SecureLabCA (5 year validity)
CA Type: Enterprise Root CA

Attacks Defeated:
→ LDAP traffic interception
→ Credential capture via LDAP
→ LDAP relay attacks
→ Directory data theft
```

### 6. SMB Signing
```
Control: SMB Signing Required
Applied: Server and client side

Settings:
→ Server signing: Always required
→ Client signing: Always required
→ SMBv1: Disabled

Attacks Defeated:
→ NTLM relay via SMB
→ Man in the middle attacks
→ EternalBlue (WannaCry)
→ SMB credential interception
```

### 7. Kerberos Hardening
```
Control: Kerberos AES Encryption
Applied: All user accounts

Settings:
→ All accounts: AES128 and AES256
→ Pre-authentication: Required on all accounts
→ Ticket lifetime: 10 hours
→ Service ticket: 600 minutes
→ Renewal lifetime: 7 days

Attacks Defeated:
→ Kerberoasting (AES uncrackable)
→ AS-REP Roasting (pre-auth required)
→ Pass the Ticket (short lifetime)
→ Golden Ticket abuse (short lifetime)
```

### 8. Advanced Audit Policy
```
Control: Advanced Audit Policy Configuration
Applied: Default Domain Controllers Policy

Categories Enabled:
→ Credential Validation: Success + Failure
→ User Account Management: Success + Failure
→ Security Group Management: Success + Failure
→ Logon: Success + Failure
→ Special Logon: Success
→ Directory Service Access: Success + Failure
→ Directory Service Changes: Success + Failure
→ Authentication Policy Change: Success

Key Event IDs Monitored:
→ 4625: Failed logon (password spray)
→ 4728: Privileged group change
→ 4720: Account created
→ 5136: AD object modified
→ 4769: Kerberos ticket requested
```

### 9. User Rights Assignment
```
Control: GPO User Rights Assignment
Applied: Default Domain Controllers Policy

Changes Made:
→ Allow log on locally: Admins only
→ Debug programs: Removed from ALL
→ Allow RDP: Admins only
→ Act as OS: No accounts

Attacks Defeated:
→ Mimikatz credential dumping
→ Unauthorized DC local access
→ Unauthorized DC RDP access
→ OS level privilege abuse
```

### 10. Security Options
```
Control: GPO Security Options
Applied: Default Domain Controllers Policy

Settings Configured:
→ LM Auth Level: 5 (NTLMv2 only)
→ SMB Signing: Always required
→ Anonymous Enumeration: Blocked
→ Hide Last Username: Enabled
→ WDigest: Disabled

Attacks Defeated:
→ NTLM relay attacks
→ Pass the Hash
→ AD reconnaissance
→ Username harvesting
→ Plaintext credential theft
```

### 11. Legacy Protocol Removal
```
Control: Protocol Hardening
Applied: Via GPO and registry

Protocols Disabled:
→ LLMNR: Disabled
→ NetBIOS: Disabled
→ SMBv1: Disabled
→ WDigest: Disabled
→ RC4 Kerberos: Restricted

Attacks Defeated:
→ Responder credential capture
→ EternalBlue exploitation
→ NTLM relay attacks
→ Kerberoasting
```

## Attack Simulations

### 1. Kerberoasting
```
Target: Service accounts with SPNs
Tool: Rubeus
Before hardening: VULNERABLE
After hardening: RESISTANT
Control that defeated it: AES256 encryption + gMSA
```

### 2. AS-REP Roasting
```
Target: Accounts with pre-auth disabled
Tool: Rubeus / Impacket
Before hardening: VULNERABLE
After hardening: BLOCKED
Control that defeated it: Pre-auth enabled on all accounts
```

### 3. Password Spray
```
Target: All domain user accounts
Tool: Custom PowerShell script
Before hardening: VULNERABLE
After hardening: DETECTED AND BLOCKED
Control that defeated it: Lockout policy + audit logging
```

### 4. NTLM Relay
```
Target: Network authentication traffic
Tool: Responder + ntlmrelayx
Before hardening: VULNERABLE
After hardening: BLOCKED
Control that defeated it: SMB signing + LLMNR disabled
```

### 5. Golden Ticket
```
Target: KRBTGT account hash
Tool: Mimikatz
Before hardening: VULNERABLE
After hardening: MITIGATED
Control that defeated it: Short ticket lifetime + AES256
```

## Challenges Faced

### Challenge 1 — RDP Lockout
```
Issue: GPO User Rights change blocked all RDP access
Impact: Complete loss of DC access
Recovery: AWS volume attachment technique
Time: 3 hours
Lesson: Always keep backup access method ready
```

### Challenge 2 — Kerberos Authentication Break
```
Issue: AES only GPO broke all domain authentication
Impact: Server completely unresponsive
Recovery: Volume attachment + registry fix
Time: 4 hours
Lesson: Never enforce AES only without testing first
```

### Challenge 3 — LAPS Configuration Issues
```
Issue: LAPS not generating passwords
Impact: Local admin passwords not managed
Recovery: Correct registry path configuration
Time: 1 hour
Lesson: DCs need special DSRM password backup setting
```

### Challenge 4 — Sensitive Data in GitHub
```
Issue: Real passwords pushed to public repository
Impact: Credentials publicly exposed
Recovery: sed replace + git push + credential rotation
Time: 30 minutes
Lesson: Always use placeholders in documentation
```

## Security Posture Improvement

### Before Hardening
```
Password policy: 8 chars minimum ❌
Account lockout: Not configured ❌
Audit logging: Not configured ❌
SMB signing: Not required ❌
NTLM: All versions allowed ❌
LAPS: Not deployed ❌
Protected Users: Empty ❌
LDAPS: Not configured ❌
Legacy protocols: All enabled ❌
Debug rights: Administrators ❌
```

### After Hardening
```
Password policy: 15 chars + complexity ✅
Account lockout: 10 attempts, 30 min ✅
Audit logging: All critical events ✅
SMB signing: Always required ✅
NTLM: Level 5 — NTLMv2 only ✅
LAPS: Deployed on all OUs ✅
Protected Users: All admins added ✅
LDAPS: Port 636 active ✅
Legacy protocols: LLMNR/SMBv1 disabled ✅
Debug rights: Removed from all ✅
```

## Recommendations

### Immediate Actions
```
1. Rotate KRBTGT password twice
2. Deploy Privileged Access Workstations
3. Enable Microsoft Defender for Identity
4. Implement Just-In-Time admin access
5. Deploy Azure AD Password Protection
```

### Long Term Actions
```
1. Regular PingCastle assessments monthly
2. SIEM integration for event log analysis
3. Regular privileged access reviews
4. Security awareness training for users
5. Incident response plan development
```

## Conclusion
This lab project successfully demonstrated the complete
process of hardening an Active Directory environment
from default insecure settings to an enterprise-grade
secure configuration. Through hands-on implementation
of 11 security controls and testing of 5 attack
techniques, we gained deep practical understanding of
both offensive and defensive AD security.

The challenges faced during the project — particularly
the RDP lockout and Kerberos break — provided invaluable
real-world experience in recovery techniques and change
management practices that are directly applicable to
production enterprise environments.

## References
```
1. Microsoft Enterprise Access Model
   https://docs.microsoft.com/en-us/security/compass/privileged-access-access-model

2. CIS Benchmarks for Windows Server
   https://www.cisecurity.org/cis-benchmarks

3. MITRE ATT&CK Framework — Active Directory
   https://attack.mitre.org/tactics/TA0006/

4. PingCastle Documentation
   https://www.pingcastle.com

5. Microsoft LAPS Documentation
   https://learn.microsoft.com/en-us/windows-server/identity/laps/laps-overview
```