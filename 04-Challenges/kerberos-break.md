# Challenge 2 — Kerberos Authentication Break

## Overview
During Kerberos hardening we enforced AES only encryption
via GPO which broke all authentication in the domain and
locked us out of the server completely. This document
covers what happened and how we recovered.

## What Happened
```
Step 1 → Configured Kerberos encryption via GPO
          Unchecked RC4 and DES
          Enabled AES128 and AES256 only

Step 2 → GPO applied to Domain Controller
          gpupdate /force ran successfully

Step 3 → Attempted to RDP back into DC

Step 4 → RDP connection failed completely

Step 5 → AWS Fleet Manager also failed

Step 6 → Instance status check failed
          Server not responding at all 💀
```

## Root Cause
```
Some Windows services and components
still required RC4 encryption

When RC4 was removed:
→ Authentication failed for all services
→ Kerberos could not issue tickets
→ Domain authentication broke completely
→ Even local services could not start
→ Server became unresponsive
```

## Error Messages Seen
```
RDP Error:
"We couldn't connect to the remote PC"
Error code: 0x704

AWS Console:
Instance status check: FAILED
Status: 0xc000000e
"A required device isn't connected"

Boot Screen:
Windows Boot Manager
"Windows failed to start"
Status: 0xc000000e
```

## Recovery Process

### Step 1 — Stop Instance
```
AWS Console
→ EC2 → Instances
→ Stop EC2 instance
→ Wait until stopped
```

### Step 2 — Attach Volume to Helper
```
AWS Console
→ EC2 → Volumes
→ Detach 50GB volume from broken DC
→ Attach to helper Windows server as D:
```

### Step 3 — Fix Kerberos Registry
```powershell
# Mount volume
Set-Disk -Number 1 -IsOffline $false
Set-Partition -DiskNumber 1 `
    -PartitionNumber 1 -NewDriveLetter D

# Load SYSTEM registry hive
reg load HKLM\BROKEN `
    "D:\Windows\System32\config\SYSTEM"

# Allow ALL Kerberos encryption types
# 2147483647 = all types allowed
reg add `
    "HKLM\BROKEN\ControlSet001\Control\Lsa\Kerberos\Parameters" `
    /v SupportedEncryptionTypes `
    /t REG_DWORD /d 2147483647 /f

# Re-enable RDP
reg add `
    "HKLM\BROKEN\ControlSet001\Control\Terminal Server" `
    /v fDenyTSConnections /t REG_DWORD /d 0 /f

reg add `
    "HKLM\BROKEN\ControlSet001\Control\Terminal Server\WinStations\RDP-Tcp" `
    /v UserAuthentication /t REG_DWORD /d 0 /f

# Unload hive
reg unload HKLM\BROKEN
Write-Host "Kerberos fixed" -ForegroundColor Green
```

### Step 4 — Fix BCD Boot Configuration
```powershell
# Restore original BCD backup
Copy-Item "C:\RecoveryBackup\BCD.before-DSRM.bak" `
    "D:\Boot\BCD" -Force

# Verify BCD is correct
bcdedit /store "D:\Boot\BCD" /enum all
```

### Step 5 — Reattach and Start
```
AWS Console
→ Take volume offline
→ Detach from helper server
→ Attach back to original DC
→ Device: /dev/xvda
→ Start DC instance
→ Wait for 2/2 status checks
→ RDP in ✅
```

## Time Taken to Recover
```
Total recovery time: ~4 hours
Attempts made: Multiple
Final solution: Volume attachment + registry fix
```

## Correct Way to Harden Kerberos
```powershell
# SAFE approach — set AES on accounts
# without removing RC4 from GPO

# Set AES encryption on all users
$users = @("jsmith", "jdoe", "itadmin", "svc-webapp")
foreach ($user in $users) {
    Set-ADUser -Identity $user `
        -KerberosEncryptionType AES128,AES256
}

# Add privileged accounts to Protected Users
# This forces AES and Kerberos only for admins
Add-ADGroupMember `
    -Identity "Protected Users" `
    -Members "Administrator", "itadmin"

# Result:
# Privileged accounts use AES only ✅
# Standard accounts still work ✅
# No service disruption ✅
```

## Lessons Learned
```
1. Never remove RC4 from GPO in production
   without extensive testing first

2. Set AES on accounts individually
   rather than removing RC4 from domain

3. Protected Users group is safer way
   to enforce AES for privileged accounts

4. ALWAYS test authentication changes
   on ONE non-critical account first

5. Keep current session open while
   testing any authentication change

6. Have AWS console ready before
   ANY Kerberos or authentication change

7. Take snapshot/backup before
   major authentication changes

8. Volume attachment is the ultimate
   recovery tool for locked Windows servers
```

## Prevention Checklist
| Check | Status |
|-------|--------|
| Test on one account first | ✅ Required |
| Keep RDP session open | ✅ Required |
| AWS console ready | ✅ Required |
| Snapshot before changes | ✅ Required |
| Rollback plan documented | ✅ Required |
| Recovery steps prepared | ✅ Required |

