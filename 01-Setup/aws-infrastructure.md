# AWS Infrastructure Setup

## EC2 Instance Configuration

### Instance Details
- AMI: Windows Server 2019 Datacenter
- Instance Type: t2.medium (2 vCPU, 4GB RAM)
- Storage: 50GB SSD (gp2)
- Region: Asia Pacific (Sydney)
- Security Group: Allow RDP (3389) from my IP

### Steps to Launch Instance
1. Login to AWS Console
2. Navigate to EC2 → Launch Instance
3. Select Windows Server 2019 Base AMI
4. Choose t2.medium instance type
5. Configure 50GB storage
6. Create security group allowing port 3389
7. Launch with key pair
8. Get Windows password from AWS Console

### Initial Connection
- Download RDP file from AWS Console
- Use Administrator credentials
- Connect via Remote Desktop

## Challenges Faced During Setup
- Initial password retrieval required key pair
- Security group needed RDP port opened
- AWS assigns new IP after every restart

## Screenshots
![EC2 Instance](screenshots/ec2-instance.png)
![Security Group](screenshots/security-group.png)
