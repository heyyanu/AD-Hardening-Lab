# Kerberos Hardening

## Overview
Hardened Kerberos authentication by enforcing AES256
encryption and enabling pre-authentication on all accounts.
This defeats Kerberoasting and AS-REP Roasting attacks
which rely on weak RC4 encryption and missing pre-auth.

## Why Kerberos Hardening Matters
Default Kerberos uses RC4 encryption which is weak and
crackable. Two major attacks exploit this:

### Kerberoasting
- Targets service accounts with SPNs registered
- Attacker requests Kerberos ticket
- Takes ticket offline and cracks password
- RC4 tickets crack in hours
- AES256 tickets take millions of years

### AS-REP Roasting
- Targets accounts with pre-authentication disabled
- Attacker gets password hash WITHOUT knowing password
- Cracks hash offline
- Fix: Enable pre-auth on ALL accounts

## Settings Hardened

### AES256 Encryption
| Setting | Before | After |
|---------|--------|-------|
| Encryption type | RC4 allowed | AES256 only |
| Ticket strength | Weak | Very strong |
| Kerberoasting | Easy | Practically impossible |

### Kerberos Pre-Authentication
| Setting | Before | After |
|---------|--------|-------|
| Pre-auth required | Not enforced | Required on all accounts |
| AS-REP Roasting | Possible | Blocked |

### Ticket Lifetime
| Setting | Value |
|---------|-------|
| User ticket lifetime | 10 hours |
| Service ticket lifetime | 600 minutes |
| Renewal lifetime | 7 days |
| Clock skew tolerance | 5 minutes |

## Implementation

### Force AES Encryption via GPO
```text
Group Policy Management
→ Default Domain Policy → Edit
→ Computer Configuration
→ Policies → Windows Settings
→ Security Settings → Local Policies
→ Security Options
→ Network security: Configure encryption
  types allowed for Kerberos
→ Check ONLY:
   ✅ AES128_HMAC_SHA1
   ✅ AES256_HMAC_SHA1
   ❌ DES_CBC_CRC — uncheck
   ❌ DES_CBC_MD5 — uncheck
   ❌ RC4_HMAC_MD5 — uncheck
```

### Set AES on All User Accounts
```powershell
# Set AES on all users
$users = @("jsmith", "jdoe", "itadmin", "svc-webapp")
foreach ($user in $users) {
    Set-ADUser -Identity $user `
    -KerberosEncryptionType AES128,AES256
    Write-Host "AES set for: $user"
}

# Set AES on admin accounts
Get-ADUser -Filter {AdminCount -eq 1} |
ForEach-Object {
    Set-ADUser -Identity $_.SamAccountName `
    -KerberosEncryptionType AES128,AES256
}
```

### Enable Pre-Authentication on All Accounts
```powershell
# Find accounts with pre-auth disabled
Get-ADUser -Filter {DoesNotRequirePreAuth -eq $true} `
    -Properties DoesNotRequirePreAuth |
Select-Object Name, SamAccountName

# Fix all accounts
Get-ADUser -Filter {DoesNotRequirePreAuth -eq $true} |
ForEach-Object {
    Set-ADAccountControl `
    -Identity $_.SamAccountName `
    -DoesNotRequirePreAuth $false
}
```

### Set Ticket Lifetime via GPO
```text
Default Domain Policy → Edit
→ Computer Configuration
→ Windows Settings → Security Settings
→ Account Policies → Kerberos Policy
→ Maximum lifetime for user ticket: 10 hours
→ Maximum lifetime for service ticket: 600 minutes
→ Maximum lifetime for user ticket renewal: 7 days
→ Maximum tolerance for clock sync: 5 minutes
```

## Verification
```powershell
# Verify AES on all users
Get-ADUser -Filter * -Properties KerberosEncryptionType |
Select-Object Name, KerberosEncryptionType |
Format-Table -AutoSize
```

Example output (as documented):
```text
Name          KerberosEncryptionType
----          ----------------------
Administrator {AES128, AES256}
John Smith    {AES128, AES256}
Jane Doe      {AES128, AES256}
IT Admin      {AES128, AES256}
svc-webapp    {AES128, AES256}

# Verify no pre-auth disabled accounts
Get-ADUser -Filter {DoesNotRequirePreAuth -eq $true} |
Select-Object Name
# Should return nothing ✅
```

## ⚠️ Challenge Faced — Critical Warning
Enforcing AES only via GPO broke authentication
and locked us out of the server completely.

### What Happened
- Set Kerberos to AES only via GPO
- Some services still required RC4
- Authentication failed for all users
- Could not RDP into server
- Had to recover via AWS volume attachment

### Recovery Process
```powershell
# Fix via attached volume on helper server
reg load HKLM\BROKEN "D:\Windows\System32\config\SYSTEM"

# Allow ALL encryption types
reg add "HKLM\BROKEN\ControlSet001\Control\Lsa\Kerberos\Parameters" `
    /v SupportedEncryptionTypes /t REG_DWORD /d 2147483647 /f

reg unload HKLM\BROKEN
```

### Lesson Learned
- Never enforce AES only without testing first
- Always keep AWS console access ready
- Test on ONE account before applying to all
- Keep RDP session open while testing auth changes
- Have rollback plan ready before any auth changes

## Security Impact
| Attack | Before | After |
|--------|--------|-------|
| Kerberoasting | Easy with RC4 | Very hard with AES256 |
| AS-REP Roasting | Possible | Blocked |
| Ticket forgery | Easier | Harder |
| Pass the Ticket | Tickets valid longer | Tickets expire faster |

