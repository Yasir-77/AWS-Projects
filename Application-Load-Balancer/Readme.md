# Application Load Balancer with Private EC2 Instances

## Project Overview

This project demonstrates a common DevOps and cloud architecture pattern using AWS:

- Multiple EC2 instances deployed across different Availability Zones
- An Application Load Balancer (ALB) handling all inbound traffic
- EC2 instances placed in private subnets with no direct internet access
- Secure traffic flow using properly scoped security groups
- Optional enhancements including HTTPS, Route 53 DNS, and Auto Scaling

The goal is to ensure that all external traffic flows through the load balancer while backend instances remain isolated and secure.

---

## Architecture Diagram (Logical)

Internet  
↓  
Application Load Balancer (Public Subnets)  
↓  
Target Group  
↓  
EC2 Instances (Private Subnets, Multiple AZs)

---

## Services Used

- Amazon VPC
- Amazon EC2
- Application Load Balancer (ALB)
- Target Groups
- NAT Gateway
- AWS Certificate Manager (ACM)
- Amazon Route 53
- Auto Scaling Group (optional)
- AWS CloudShell / SSH (for access)

---

## Objectives

- Deploy two EC2 instances behind an ALB
- Ensure EC2 instances are not directly accessible from the internet
- Configure health checks and load balancing
- Validate traffic distribution across instances
- Secure traffic using HTTPS
- (Optional) Enable DNS, Auto Scaling, and high availability

---

## Step-by-Step Implementation

### 1. Create a VPC

- Create a new VPC with CIDR block `10.0.0.0/16`
- Enable DNS hostnames and DNS resolution

---

### 2. Create Subnets

Create four subnets across two Availability Zones:

**Public Subnets**
- PublicSub-A: `10.0.1.0/24`
- PublicSub-B: `10.0.2.0/24`

**Private Subnets**
- PrivateSub-A: `10.0.3.0/24`
- PrivateSub-B: `10.0.4.0/24`

---

### 3. Internet Gateway and Routing

- Create and attach an Internet Gateway to the VPC
- Create a public route table:
  - Route `0.0.0.0/0` → Internet Gateway
  - Associate with both public subnets
- Create a private route table:
  - Associate with both private subnets

---

### 4. NAT Gateways (Outbound Internet for Private EC2)

- Allocate two Elastic IPs
- Create one NAT Gateway per Availability Zone:
  - NATGW-A in PublicSub-A
  - NATGW-B in PublicSub-B
- Update private route tables:
  - PrivateSub-A → NATGW-A
  - PrivateSub-B → NATGW-B

This allows private instances to download packages without being publicly accessible.

---

### 5. Security Groups

#### ALB Security Group
- Inbound:
  - HTTP (80) from `0.0.0.0/0`
  - HTTPS (443) from `0.0.0.0/0`
- Outbound:
  - Allow all

#### EC2 Security Group
- Inbound:
  - HTTP (80) from ALB security group only
  - SSH (22) from Bastion security group (optional)
- Outbound:
  - Allow all

---

### 6. Launch EC2 Instances

- Launch two Amazon Linux 2 instances
- Place each instance in a different private subnet
- Disable public IPv4 assignment
- Attach the EC2 security group

#### User Data Script

Each instance runs Apache and returns unique content:

```bash
#!/bin/bash
yum update -y
yum install -y httpd
systemctl enable httpd
systemctl start httpd
echo "<h1>Hello from Web Server 1</h1>" > /var/www/html/index.html

