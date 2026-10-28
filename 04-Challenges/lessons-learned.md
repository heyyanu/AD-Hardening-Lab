# Lessons Learned

## Overview
Key lessons learned throughout the Active Directory
hardening lab project. These lessons come from real
mistakes made during implementation and are valuable
for anyone planning to harden AD in a real environment.

## Top 10 Lessons Learned

### 1. Always Keep Backup Access Ready
```
What happened:
→ Changed GPO settings
→ Locked ourselves out of DC
→ No backup access method ready

Lesson:
→ ALWAYS verify AWS console access
   before making ANY auth changes
→ Test Fleet Manager BEFORE
   making any GPO changes
→ Keep alternative access method
   ready at ALL times
```

### 2. Never Change Auth Settings Without Testing
```
What happened:
→ Enforced AES only Kerberos via GPO
→ Broke all domain authentication
→ Server became completely unresponsive

Lesson:
→ Test EVERY auth change on
   ONE non-critical account first
→ Keep RDP session OPEN while testing
→ Verify login works before closing session
→ Never apply domain-wide auth changes
   without extensive lab testing first
```

### 3. GPO Changes Apply Immediately
```
What happened:
→ Made GPO change thinking we had time
→ gpupdate applied change instantly
→ Got locked out before realising

Lesson:
→ GPO changes can apply immediately
→ Have rollback plan READY before
   making any security GPO change
→ Test in isolated lab OU first
→ Use GPO links to control scope
```

### 4. Volume Attachment is Powerful Recovery Tool
```
What we learned:
→ When all else fails on AWS
→ Stop instance
→ Detach root volume
→ Attach to helper server
→ Edit registry and files directly
→ Reattach and restart
→ Powerful recovery technique ✅

Skills gained:
→ Registry editing on offline volumes
→ BCD boot configuration repair
→ SYSVOL GPO file editing
→ AWS volume management
```

### 5. Document Every Change Made
```
What happened:
→ Made multiple changes during recovery
→ Lost track of what was changed
→ Hard to know what fixed the issue

Lesson:
→ Document EVERY change as you make it
→ Note exact command used
→ Note time of change
→ Note result of change
→ Makes troubleshooting much faster
```

### 6. LAPS Configuration is Complex
```
What happened:
→ LAPS installed but not generating passwords
→ Multiple registry paths to configure
→ DC required special DSRM setting
→ Took many attempts to get working

Lesson:
→ Windows LAPS and Legacy LAPS
   use different registry paths
→ DCs need BackupDSRMPassword enabled
→ Check LAPS event log for errors
   Event ID 10004 = Success
   Event ID 10005 = Failure
→ Link GPO to Domain Controllers OU
   separately from other OUs
```

### 7. Never Push Sensitive Data to GitHub
```
What happened:
→ Accidentally pushed real passwords
→ Real computer names
→ Real IP addresses
→ All visible publicly on GitHub

Lesson:
→ ALWAYS use placeholders in documentation
   YOUR-DC-HOSTNAME not real hostname
   **REDACTED** not real password
   YOUR-IP not real IP address
→ Check files before git push
→ Use git grep to find sensitive data
→ Rotate any exposed credentials immediately
```

### 8. Test Tools Before Relying on Them
```
What happened:
→ Purple Knight required registration
→ Could not download directly
→ PingCastle version was expired
→ Had to find and download latest version

Lesson:
→ Download and test all tools
   before starting the lab
→ Check tool versions are current
→ Have offline copies where possible
→ Know alternative tools if primary fails
```

### 9. AWS Changes IP After Restart
```
What happened:
→ Restarted EC2 instance
→ Could not RDP with saved connection
→ IP address had changed

Lesson:
→ AWS assigns NEW public IP after restart
→ Always check current IP in AWS console
→ Use Elastic IP for permanent IP address
→ Update RDP connection file after restart
```

### 10. Security Hardening is a Process Not a Task
```
What we learned:
→ Cannot harden everything at once
→ Each control can break something
→ Must test and verify each step
→ Takes time and multiple iterations
→ Real environments take months/years

Lesson:
→ Plan changes carefully
→ Test each control individually
→ Verify before moving to next
→ Document everything
→ Have rollback plan always ready
```

## Summary of Recovery Techniques Used

| Issue | Recovery Method | Time |
|-------|----------------|------|
| RDP lockout | Volume attachment + GPO fix | 3 hours |
| Kerberos break | Volume attachment + registry fix | 4 hours |
| LAPS not working | Registry path correction | 1 hour |
| BCD boot failure | BCD backup restoration | 1 hour |
| Password pushed to GitHub | sed replace + git push | 30 mins |

## Skills Gained Through Challenges

### Technical Skills
```
✅ AWS EC2 volume management
✅ Windows registry editing
✅ BCD boot configuration repair
✅ SYSVOL GPO file editing
✅ Offline Windows recovery
✅ Git sensitive data removal
✅ PowerShell troubleshooting
```

### Security Skills
```
✅ Understanding GPO impact
✅ Authentication troubleshooting
✅ Kerberos encryption management
✅ LAPS deployment and configuration
✅ AD security hardening
✅ Recovery planning
✅ Change management importance
```

## Final Advice
```
For anyone planning to harden AD:

1. Build a lab first — never test in production
2. Take snapshots before every major change
3. Keep backup access method ready always
4. Test every change before applying domain-wide
5. Document everything as you go
6. Have rollback plan for every change
7. Learn from mistakes — they are the best teacher
8. Breaking things in a lab is GOOD experience
   Better to break it here than in production
```
