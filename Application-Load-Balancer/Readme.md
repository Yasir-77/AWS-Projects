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

- Go to VPC → Your VPCs → Create VPC
- Choose: VPC Only
- Name: Project
- IPv4 CIDR: 10.0.0.0/16
- Click Create VPC

<img width="1511" height="1084" alt="image" src="https://github.com/user-attachments/assets/ea05fdae-9df3-4260-b38b-a1891ad9242e" />


---

### 2. Create Subnets

Create four subnets across two Availability Zones:

**Public Subnets**
- PublicSub-A: `10.0.0.0/24`
- PublicSub-B: `10.0.1.0/24`

**Private Subnets**
- PrivateSub-A: `10.0.2.0/24`
- PrivateSub-B: `10.0.3.0/24`

<img width="1707" height="477" alt="image" src="https://github.com/user-attachments/assets/82c73b80-16b1-465d-a1b0-de8079cdc6b5" />

#### Subnets are created across two Availability Zones to ensure high availability and fault tolerance. Public subnets are used for internet-facing resources, while private subnets are used to host EC2 instances that should not be directly accessible from the internet.
---

### 3. Internet Gateway 

Create an Internet Gateway

- VPC → Internet Gateways → Create
- Name: project-igw
- Create → Attach to VPC → choose my-vpc
  
<img width="1701" height="441" alt="image" src="https://github.com/user-attachments/assets/63b99d2c-a0ef-4e76-b154-84590e4963a3" />

#### An Internet Gateway is attached to the VPC to allow resources in public subnets (such as the Application Load Balancer) to communicate with the internet. This gateway is not used by private subnets, maintaining their isolation.

---

### 4.Route Tables

#### Public Route Table

- VPC → Route Tables → Create Route Table
- Name: public-rt
- VPC: my-vpc
- Create

Edit Routes:
- Destination: 0.0.0.0/0
- Target: Internet Gateway (my-igw)

Edit Subnet Associations:
- Select:
  - public-subnet-a
  - public-subnet-b

Now these subnets are officially public.

<img width="1691" height="523" alt="image" src="https://github.com/user-attachments/assets/5a8536f6-669f-45e5-b829-cd496e5f90ab" />

<img width="1690" height="412" alt="image" src="https://github.com/user-attachments/assets/8de1072d-96a5-4362-b753-783c3deccaf8" />

<img width="1701" height="556" alt="image" src="https://github.com/user-attachments/assets/fc18cfdd-fe1c-425b-9775-771f8f2aa688" />

#### Private Route Table

- Create route table: private-rt
- Associate:
  - private-subnet-a
  - private-subnet-b

DO NOT add an internet gateway route.
The private subnets will have no internet path unless we add NAT, which we do NOT need for this assignment.

<img width="1694" height="512" alt="image" src="https://github.com/user-attachments/assets/24df2f1b-bb79-4e48-8efc-276aa526a48c" />

<img width="1700" height="503" alt="image" src="https://github.com/user-attachments/assets/044d8e1b-0cda-4977-9337-ed72de8890fd" />

#### Route tables define how traffic flows within the VPC and to external networks. Public subnets require a route to the Internet Gateway, while private subnets intentionally do not allow direct internet access. This separation ensures that only explicitly public resources are internet-facing.

---

### 5. CREATE NAT GATEWAYS 

### Create NAT Gateway A (AZ-A)

- Go to VPC → NAT Gateways → Create NAT Gateway

- Name: NATGW-A
- Subnet: public-subnet-a
- Elastic IP: Allocate EIP
- Click Create NAT Gateway

Wait until status shows Available.

### Create NAT Gateway B (AZ-B)

- Create another NAT Gateway
- Name: NATGW-B
- Subnet: public-subnet-b
- Elastic IP: Allocate EIP
- Click Create NAT Gateway

Wait until status shows Available.

<img width="1696" height="276" alt="image" src="https://github.com/user-attachments/assets/07200e10-4c01-4a9d-8831-c61316ad4b0e" />

#### Because EC2 instances in private subnets must install software (e.g. Apache via `yum`), they require outbound internet access. NAT Gateways provide this access without exposing the instances to inbound internet traffic. To ensure high availability and avoid cross-AZ traffic, one NAT Gateway is deployed per Availability Zone.
---

### 6.UPDATE PRIVATE ROUTE TABLES

### Private Route Table (AZ-A)

- Go to VPC → Route Tables
- Select the route table associated with private-subnet-a
- Edit Routes → Add route:
  - Destination: 0.0.0.0/0
  - Target: NATGW-A

Save routes

<img width="1700" height="494" alt="image" src="https://github.com/user-attachments/assets/45b07e67-5f05-48e3-889a-48a89fefc1eb" />

### Private Route Table (AZ-B)

- Select the route table associated with private-subnet-b
- Edit Routes → Add route:
  - Destination: 0.0.0.0/0
  - Target: NATGW-B

Save routes

<img width="1704" height="475" alt="image" src="https://github.com/user-attachments/assets/692cd11a-e056-4168-81d5-40c3cda08115" />

---

### 7. Security Groups

#### ALB Security Group
- Inbound:
  - HTTP (80) from `0.0.0.0/0`
  - HTTPS (443) from `0.0.0.0/0`
- Outbound:
  - Allow all
 <img width="1675" height="1211" alt="image" src="https://github.com/user-attachments/assets/35c35f6d-bbea-4b7b-b946-52535c5843c1" />


#### EC2 Security Group
- Inbound:
  - HTTP (80) from ALB security group only
- Outbound:
  - Allow all

<img width="1683" height="1103" alt="image" src="https://github.com/user-attachments/assets/0315af17-2928-4258-a665-f37cd7d54856" />

#### Security groups are used to strictly control traffic between components. This ensures EC2 instances can only receive traffic from the load balancer and are not directly reachable from the internet.

---

### 8. Launch EC2 Instances

#### Launch Instance 1

- EC2 → Launch Instance
- Name: web1
- AMI: Amazon Linux 2
- Instance type: t2.micro
- Key pair: web1
- Network:
  - VPC: project-vpc
  - Subnet: private-subnet-a
  - Auto-assign public IP: Disable
  - Security group: sg-ec2

User-data for web1
```
#!/bin/bash
yum update -y
yum install -y httpd
systemctl enable httpd
systemctl start httpd
echo "<h1>Hello from WEB SERVER 1</h1>" > /var/www/html/index.html
```
Launch.

### Launch Instance 2

Same as instance 1, except:
- Subnet: private-subnet-b

User-data:
```
#!/bin/bash
yum update -y
yum install -y httpd
systemctl enable httpd
systemctl start httpd
echo "<h1>Hello from WEB SERVER 2</h1>" > /var/www/html/index.html
```
Launch.

<img width="1704" height="277" alt="image" src="https://github.com/user-attachments/assets/271c89dd-71d3-407f-8344-f69d1d63ddc9" />

#### Two EC2 instances are launched in separate Availability Zones to provide redundancy. They are placed in private subnets with no public IP addresses, enforcing backend isolation.
Each instance uses user-data to install and start Apache and serve a simple HTML page identifying the instance.
---

### 9. CREATE TARGET GROUP

- EC2 → Target Groups → Create
- Target type: Instances
- Name: tg-web
- Protocol: HTTP
- Port: 80
- VPC: project-vpc
- Health check path: /

Register targets
- Select both EC2 instances → Add → Create.

<img width="1691" height="869" alt="image" src="https://github.com/user-attachments/assets/85b79ee0-3d20-4730-92de-beb9a1e48eda" />

#### A target group is created to define how the Application Load Balancer routes traffic to backend instances. It also performs health checks to ensure traffic is only sent to healthy EC2 instances.
---

### 10. CREATE THE APPLICATION LOAD BALANCER

- EC2 → Load Balancers → Create Load Balancer
- Choose: Application Load Balancer
- Name: my-alb
- Scheme: Internet-facing
- IP type: IPv4

Network mapping:
- VPC: my-vpc
- Subnets:
  - public-subnet-a
  - public-subnet-b
- Security group:
  - Select: sg-alb
- Listener:
  - HTTP : 80 → forward to tg-web

Create ALB.

<img width="1680" height="929" alt="image" src="https://github.com/user-attachments/assets/84b864c0-8828-454c-a511-fdb4980e8bb5" />

An internet-facing Application Load Balancer is created across both public subnets. It serves as the single entry point for all inbound traffic and distributes requests across healthy backend instances.

---

### 11.TEST

#### Get the ALB DNS name

- Go to ALB → Description → copy
- Test in browser
- Refresh multiple times:
- You should see:
  - "Hello from WEB SERVER 1"
  - "Hello from WEB SERVER 2"

<img width="1371" height="244" alt="image" src="https://github.com/user-attachments/assets/6cfd8530-cd4a-4323-9cd0-0c6b0a7f9517" />

<img width="1318" height="297" alt="image" src="https://github.com/user-attachments/assets/e9aec5dc-5d1a-4f13-a48f-349d22e33de9" />

#### Check health statuses

- Target groups → tg-web → Targets
- Both should be healthy.
<img width="1431" height="785" alt="image" src="https://github.com/user-attachments/assets/43bfe130-35a4-42dd-a1a3-d0cc5379717d" />

The ALB DNS name is accessed in a browser to verify functionality. Refreshing the page confirms traffic is distributed across both EC2 instances. Target group health checks confirm that both instances are healthy and serving traffic correctly.

---

### BONUS 1 - Add a Route53 DNS name and point it to the ALB DNS name via ALIAS record type.

#### 1.Create a Public Hosted Zone in Route 53

- Go to Route 53 → Hosted Zones
- Click Create hosted zone
- Domain name: yasirahmed.co.uk
- Type: Public Hosted Zone
- Create
- Route 53 will give you 4 name servers like:
  - ns-123.awsdns-45.com
  - ns-678.awsdns-90.net

<img width="1701" height="496" alt="image" src="https://github.com/user-attachments/assets/c12db874-42df-4bc0-a8af-1c475b18dfc5" />

### 2.Update Name Servers in GoDaddy

- Log in to GoDaddy
- Go to My Products → Domains
- Click your domain → DNS → Nameservers
- Choose Change → Custom
- Replace all GoDaddy name servers with the 4 Route 53 name servers
- Save

Note: DNS propagation can take a few minutes up to 48 hours (usually much faster).

<img width="1698" height="1164" alt="image" src="https://github.com/user-attachments/assets/22d1ac1f-367c-493b-9354-8adb0b4717a1" />


### 3.Create the ALIAS Record in Route 53

- Once propagation starts: Route 53 → Hosted Zones → yourdomain.com
- Click Create record
  - Record name: app
  - Record type: A
  - Alias: Yes
  - Alias target
  - Application Load Balancer
  - Select your region
  - Select your ALB
- Create record

<img width="1690" height="697" alt="image" src="https://github.com/user-attachments/assets/2aa1e2c6-ce26-458a-8be8-edd3529bf07a" />

### 4. Enable HTTPS with ACM and Attach Certificate to the ALB

Once the domain is successfully resolving to the Application Load Balancer via Route 53, HTTPS must be enabled using an SSL/TLS certificate that matches the domain name. This ensures encrypted communication and prevents browser security warnings.

Because the domain is served by an AWS Application Load Balancer, certificates must be issued by **AWS Certificate Manager (ACM)** and must exactly match the requested domain.

#### 4.1 - Verify the Certificate Currently Attached to the ALB

1. Navigate to **EC2 → Load Balancers**
2. Select the Application Load Balancer
3. Open the **Listeners** tab
4. Select the **HTTPS : 443** listener
5. Check the **Default SSL certificate** and note the domain name

If the certificate does not include `test.yasirahmed.co.uk`, a new certificate must be issued.

#### 4.2 - Request a New ACM Certificate

1. Go to **AWS Certificate Manager (ACM)**
2. Click **Request certificate**
3. Choose **Public certificate**
4. Add the following domain option: test.yasirahmed.co.uk
5. Select **DNS validation**
6. Request the certificate
7. Click **Create records in Route 53** to automatically add the required validation records
8. Wait until the certificate status changes to **Issued**

> The certificate must be created in the same AWS region as the Application Load Balancer.

#### 4.3 - Attach the Certificate to the ALB HTTPS Listener

1. Return to **EC2 → Load Balancers**
2. Select the Application Load Balancer
3. Open the **Listeners** tab
4. Edit the **HTTPS : 443** listener
5. Select the newly issued ACM certificate
6. Save changes

After a short propagation period, HTTPS will be active and the domain can be accessed securely without browser warnings.

<img width="1437" height="313" alt="image" src="https://github.com/user-attachments/assets/1f946dcd-2c7e-4a01-b674-905154d17ff0" />
<img width="1645" height="519" alt="image" src="https://github.com/user-attachments/assets/15abf921-ce8c-4edf-bd24-7b44e131009b" />



### 5.Test

After propagation:
```
http://test.yasirahmed.co.uk
```

If HTTPS is enabled:
```
https://test.yasirahmed.co.uk
```

This is the best-practice AWS solution. The custom domain now resolves securely over HTTPS using a valid ACM certificate, completing the Route 53 and SSL integration while keeping GoDaddy as the domain registrar.

<img width="1684" height="282" alt="image" src="https://github.com/user-attachments/assets/8ac8b31b-0de4-4616-a679-68f5b14264e2" />

<img width="1694" height="204" alt="image" src="https://github.com/user-attachments/assets/d38a7624-21d7-4a68-b3b8-7ca41f325657" />












