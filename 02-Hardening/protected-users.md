# Protected Users Security Group

## Overview
Added all privileged accounts to the Protected Users
security group to automatically enforce strong
authentication restrictions. This group removes entire
categories of credential theft attacks for high-value accounts.

## What is Protected Users
A special built-in AD security group that automatically
enforces strict security controls on any account added to it.
Think of it like an automatic armour upgrade for admin accounts.

## What it Automatically Enforces
| Protection | Effect | Attack Defeated |
|-----------|--------|----------------|
| Kerberos only | No NTLM authentication | NTLM relay |
| No credential caching | Hash not stored on machines | Pass the Hash |
| AES encryption only | No RC4 tickets | Kerberoasting |
| Ticket expires in 4 hours | Stolen tickets useless quickly | Pass the Ticket |
| No delegation | Cannot be impersonated | Delegation abuse |

## Who to Add
```text
✅ Add these accounts:
→ Domain Admins
→ Enterprise Admins
→ All Tier 0 admin accounts
→ All Tier 1 admin accounts

❌ Never add these:
→ Regular users (breaks daily work)
→ Service accounts (breaks services)
→ Computer accounts (breaks auth)
```

## What Happens if Regular Users Added
- Cannot connect to some network resources
- WiFi may stop working
- Cannot log in offline (no credential caching)
- Gets kicked out of apps every 4 hours
- Some VPNs stop working

## Implementation

### Via PowerShell
```powershell
# Add admin accounts to Protected Users
Add-ADGroupMember -Identity "Protected Users" `
    -Members "Administrator"

Add-ADGroupMember -Identity "Protected Users" `
    -Members "itadmin"

# Verify members
Get-ADGroupMember -Identity "Protected Users" |
Select-Object Name, SamAccountName
```

### Via GUI
```text
Active Directory Users and Computers
→ Find "Protected Users" group
→ Right click → Properties
→ Members tab → Add
→ Type admin username → OK
→ Apply → OK
```

## Verification
```powershell
Get-ADGroupMember -Identity "Protected Users" |
Select-Object Name, SamAccountName
```

Example membership output (as documented):
```text
Name          SamAccountName
----          --------------
Administrator Administrator
IT Admin      itadmin
```

## Testing Protection Works
```powershell
# Try NTLM auth with Protected Users account
# Should fail with:
# "NTLM authentication is not allowed for this account"

# Try credential caching
# runas /savecred should fail

# Check Kerberos ticket lifetime
# Should show 4 hour expiry maximum
klist
```

## Security Impact
| Before Protected Users | After Protected Users |
|----------------------|---------------------|
| NTLM auth allowed | Kerberos only |
| Credentials cached | No caching |
| RC4 tickets | AES only |
| Tickets last 10 hours | Tickets expire 4 hours |
| Delegation allowed | No delegation |

## Key Lessons
- Only add privileged accounts — never regular users
- Service accounts will break if added
- Computer accounts must never be added
- Test with one admin account before adding all
- Check klist to verify ticket lifetime changed

