# 🔒 Active Directory Hardening Lab

## 🎯 Project Overview
A hands-on cybersecurity lab project documenting the complete 
process of building and hardening an enterprise Active Directory 
environment on AWS EC2. This project covers real-world security 
controls, attack simulations, challenges faced, and lessons learned.

## 🏗️ Infrastructure
- Cloud: Amazon Web Services (AWS)
- Server: Windows Server 2019 Datacenter
- Service: Active Directory Domain Services
- Domain: YOUR-DOMAIN.local
- Instance: EC2 t2.medium

## 🔒 Security Controls Implemented
| Control | Status |
|---------|--------|
| Password Policy (15+ chars) | ✅ Implemented |
| Fine-Grained Password Policies | ✅ Implemented |
| LAPS Deployment | ✅ Implemented |
| Protected Users Group | ✅ Implemented |
| LDAPS (port 636) | ✅ Implemented |
| SMB Signing | ✅ Implemented |
| Kerberos AES256 Hardening | ✅ Implemented |
| Advanced Audit Policy | ✅ Implemented |
| User Rights Assignment | ✅ Implemented |
| Security Options | ✅ Implemented |
| Legacy Protocol Removal | ✅ Implemented |

## ⚔️ Attack Simulations
| Attack | Result |
|--------|--------|
| Kerberoasting | Tested |
| Password Spray | Tested |
| NTLM Relay | Tested |

## 🚧 Challenges Faced
- RDP lockout after GPO User Rights Assignment changes
- Kerberos authentication break after AES enforcement
- LAPS configuration issues on Windows Server 2019
- Server recovery using AWS volume attachment technique

## 📊 Results
- PingCastle score before hardening: TBD
- PingCastle score after hardening: TBD

## 📚 Lessons Learned
- Always test GPO changes in lab before production
- Keep backup access method ready before any auth changes
- Document every configuration change made
- AWS console is your best friend for recovery

## 🛠️ Tools Used
- PingCastle — AD security assessment
- Purple Knight — Attack indicator scanning
- Microsoft Security Compliance Toolkit
- Windows LAPS
- Active Directory Users and Computers
- Group Policy Management Console

## 📁 Repository Structure
- 01-Setup — AWS and AD DS installation steps
- 02-Hardening — Security controls implementation
- 03-Attacks — Attack simulation results
- 04-Challenges — Issues faced and resolutions
- 05-Verification — PingCastle reports
- 06-Report — Final security report
