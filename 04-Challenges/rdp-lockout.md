# Challenge 1 — RDP Lockout After GPO Changes

## Overview
During the hardening process we accidentally locked
ourselves out of the Domain Controller by misconfiguring
User Rights Assignment in Group Policy. This document
covers what happened, why it happened, and how we
recovered.

## What Happened
```
Step 1 → Configured User Rights Assignment
          via Default Domain Controllers Policy

Step 2 → Restricted "Deny log on through
          Remote Desktop Services"

Step 3 → GPO applied to Domain Controller

Step 4 → Attempted to RDP back into DC

Step 5 → Error received:
          "To sign in remotely you need the right
           to sign in through Remote Desktop Services"

Step 6 → ALL RDP access blocked 💀
          Could not get back in
```

## Root Cause
```
GPO change restricted RDP access
before verifying alternative access
method was available.

The change applied immediately via
gpupdate and blocked our own session.

Lesson: NEVER change authentication
settings without backup access ready.
```

## Error Messages Seen
```
Error 1:
"To sign in remotely, you need the right to sign in
through Remote Desktop Services"

Error 2:
"Unable to establish Remote Desktop connection.
Error code: 0x704"

Error 3:
Instance status check failed
Status: 0xc000000e
"A required device isn't connected or can't be accessed"
```

## Recovery Process

### Attempt 1 — AWS Fleet Manager
```
AWS Console → EC2 → Connect
→ Fleet Manager Remote Desktop
→ Failed — same GPO restriction applied
```

### Attempt 2 — User Data Script
```
AWS Console → Stop instance
→ Edit User Data
→ Added PowerShell script to fix RDP
→ Started instance
→ Script did not fully apply
```

### Attempt 3 — Volume Attachment (SUCCESS)
```
Step 1 → Stop broken DC instance
Step 2 → Detach 50GB root volume from DC
Step 3 → Attach volume to helper Windows server
Step 4 → Mount volume as drive D:
Step 5 → Edit registry directly on mounted volume
Step 6 → Fix GPO files in SYSVOL
Step 7 → Clear security database
Step 8 → Detach from helper server
Step 9 → Reattach to original DC
Step 10 → Start DC instance
Step 11 → RDP access restored ✅
```

### Fix Commands Used on Helper Server
```powershell
# Fix GPO files — remove deny RDP setting
Get-ChildItem "D:\Windows\SYSVOL" `
    -Recurse -Filter "GptTmpl.inf" |
    ForEach-Object {
        $content = Get-Content $_.FullName
        $content -replace `
            "SeDenyRemoteInteractiveLogonRight.*", "" |
        Set-Content $_.FullName
    }

# Delete security database — forces rebuild
Remove-Item "D:\Windows\security\database\secedit.sdb" `
    -Force -ErrorAction SilentlyContinue

# Fix registry — enable RDP
reg load HKLM\BROKEN `
    "D:\Windows\System32\config\SYSTEM"

reg add "HKLM\BROKEN\ControlSet001\Control\Terminal Server" `
    /v fDenyTSConnections /t REG_DWORD /d 0 /f

reg add "HKLM\BROKEN\ControlSet001\Control\Terminal Server\WinStations\RDP-Tcp" `
    /v UserAuthentication /t REG_DWORD /d 0 /f

reg unload HKLM\BROKEN

# Clear GPO cache
reg load HKLM\BROKENSW `
    "D:\Windows\System32\config\SOFTWARE"

reg delete "HKLM\BROKENSW\Microsoft\Windows\CurrentVersion\Group Policy\State\Machine" `
    /f 2>$null

reg unload HKLM\BROKENSW
```

## Time Taken to Recover
```
Total recovery time: ~3 hours
Attempts made: 5+
Final solution: Volume attachment technique
```

## Lessons Learned
```
1. ALWAYS keep AWS console access ready
   before making any GPO auth changes

2. NEVER change RDP rights without
   verifying alternative access first

3. Test GPO changes on ONE machine first
   before applying to Domain Controllers

4. Keep current RDP session OPEN while
   testing GPO changes — verify before closing

5. Have rollback plan ready for every
   authentication change

6. Volume attachment is powerful recovery tool
   for locked out Windows servers on AWS

7. Document every change made so recovery
   is faster if something breaks
```

## Prevention Checklist
| Check | Description |
|-------|-------------|
| AWS console access | Verified before changes |
| Alternative access | Fleet Manager tested |
| Test environment | Changes tested in lab first |
| Rollback plan | Documented before changes |
| Session kept open | RDP session maintained during changes |
