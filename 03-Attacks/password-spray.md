# Password Spray Attack Simulation

## Overview
Tested password spray attack against the AD environment
to demonstrate the effectiveness of account lockout
policies and audit logging in detecting and preventing
this common attack technique.

## What is Password Spray
Password spray tries ONE common password against
MANY accounts to avoid triggering lockout on any
single account.

### Why It Is Different from Brute Force
```
Brute Force:
→ Many passwords against ONE account
→ Triggers account lockout quickly
→ Easy to detect and block

Password Spray:
→ One password against MANY accounts
→ Stays under lockout threshold
→ Harder to detect
→ Very effective against weak passwords
```

### Attack Flow
```
Step 1 → Attacker enumerates all domain usernames
Step 2 → Picks one common password
          Example: Company2024!
Step 3 → Tries that password on ALL accounts
Step 4 → Waits 30 minutes (avoids lockout)
Step 5 → Tries next common password
Step 6 → Repeats until account found ✅
```

## Common Passwords Attackers Try
```
Password1
Company2024!
Welcome1
Summer2024
Winter2024!
Monday1
January2024
[CompanyName]2024
```

## Attack Simulation

### Step 1 — Enumerate Domain Users
```powershell
# Get all enabled user accounts
Get-ADUser -Filter {Enabled -eq $true} |
Select SamAccountName |
Export-Csv "users.txt" -NoTypeInformation
```

### Step 2 — Simulate Spray Attempt
```powershell
# Simulate failed logon attempts
# This generates Event ID 4625 for detection testing
$users = @("jsmith", "jdoe", "itadmin")
$password = "Password123!"

foreach ($user in $users) {
    $secpasswd = ConvertTo-SecureString $password -AsPlainText -Force
    $cred = New-Object System.Management.Automation.PSCredential($user, $secpasswd)
    try {
        # Attempt authentication
        $null = New-PSSession -ComputerName localhost -Credential $cred -ErrorAction Stop
    } catch {
        Write-Host "Failed login for: $user" -ForegroundColor Red
    }
}
```

## Attack Results

### Before Hardening
```
Password Spray Result: VULNERABLE
Account Lockout: Not configured
Detection: No audit logging
Password Complexity: Weak (8 chars)
Risk Level: CRITICAL
```

### After Hardening
```
Password Spray Result: DETECTED AND BLOCKED
Account Lockout: 10 attempts then locked
Detection: Event ID 4625 generated
Password Complexity: 15+ chars required
Risk Level: LOW
```

## Hardening Controls That Defeated This Attack

### 1. Account Lockout Policy
```powershell
# Verify lockout policy
Get-ADDefaultDomainPasswordPolicy |
Select LockoutThreshold, LockoutDuration

LockoutThreshold : 10
LockoutDuration  : 00:30:00
```

### 2. Strong Password Policy
```powershell
# Verify password requirements
Get-ADDefaultDomainPasswordPolicy |
Select MinPasswordLength, ComplexityEnabled

MinPasswordLength : 15
ComplexityEnabled : True
```

### 3. Audit Logging Detection
```powershell
# Detect password spray via Event ID 4625
Get-WinEvent -FilterHashtable @{
    LogName = 'Security'
    Id = 4625
} -MaxEvents 20 |
Select TimeCreated,
@{N="Username";E={$_.Properties[5].Value}},
@{N="SourceIP";E={$_.Properties[19].Value}} |
Format-Table -AutoSize
```

## Detection Pattern
```
Password Spray Indicator:
→ Multiple Event ID 4625 in short time
→ DIFFERENT usernames
→ SAME source IP
→ SAME password attempt time

Brute Force Indicator:
→ Multiple Event ID 4625 in short time
→ SAME username
→ Different passwords
```

## Azure AD Password Protection
```
Additional protection:
→ Blocks commonly used passwords
→ Blocks company name variations
→ Blocks seasonal variations
→ Example: blocks "Company2024!"
→ Works for on-premises AD too
```

## Key Lessons
- Password spray is hard to detect without proper logging
- Account lockout alone is not enough
- Strong password policy defeats most spray attempts
- Monitor for multiple 4625 events from same IP
- Azure AD Password Protection adds extra layer
- Fine-Grained policies make admin accounts much harder
