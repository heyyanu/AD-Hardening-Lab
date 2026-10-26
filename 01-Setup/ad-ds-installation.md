# Active Directory Domain Services Installation

## Prerequisites
- Windows Server 2019 installed
- Static IP configured
- Administrator access

## Installation Steps

### Step 1 — Install AD DS Role
Server Manager
→ Add Roles and Features
→ Role-based installation
→ Active Directory Domain Services
→ Add Features
→ Install

### Step 2 — Promote to Domain Controller
Server Manager
→ Yellow notification flag
→ Promote this server to DC
→ Add new forest
→ Domain name: secure.local
→ Forest level: Windows Server 2016
→ Set DSRM password
→ Install
→ Server reboots automatically

### Step 3 — Verify Installation
\`\`\`powershell
# Verify domain is working
Get-ADDomain

# Verify DC is working
Get-ADDomainController

# Verify services running
Get-Service adws,kdc,netlogon,dns |
Select Name, Status
\`\`\`

## Verification Results
\`\`\`
Name      Status
----      ------
adws     Running
dns      Running
kdc      Running
Netlogon Running
\`\`\`

## Structure Created
\`\`\`
secure.local
├── OU=IT_Department
├── OU=HR_Department
├── OU=Servers
├── OU=Workstations
└── OU=ServiceAccounts
\`\`\`

## Users Created
| Name | Username | Department |
|------|----------|-----------|
| John Smith | jsmith | IT |
| Jane Doe | jdoe | HR |
| IT Admin | itadmin | IT |
| svc-webapp | svc-webapp | Service Account |
