# AWS Networking Assignment: VPC and Network Setup

## Objective:
Create a custom VPC with both public and private subnets, configure internet access, and deploy EC2 instances with secure segmentation. 

## Requirements: 
- VPC & Subnets: Create a custom VPC with at least one public and one private subnet. 
- Internet & NAT Gateway: Attach an Internet Gateway (IGW). Deploy a NAT Gateway (NGW) in the public subnet. 
- EC2 Instances: Launch one EC2 instance in the public subnet (with internet access). Launch another EC2 instance in the private subnet (no direct internet access). 
- Routing & Security: Configure route tables for correct routing. Restrict direct access to the private instance. 
- Bonus Challenge (Optional): Set up a Bastion Host (Jump Box)

## Project Summary:
This project implements a secure network architecture on AWS using a custom VPC, public and private subnets, a bastion host, and controlled routing. It demonstrates how to safely expose public resources while isolating private compute instances from direct internet access.

## Project Architecture
<img width="1228" height="826" alt="image" src="https://github.com/user-attachments/assets/5930f023-d778-458e-97fb-a77aee487f0f" />

## Architecture Description

This project implements a secure, segmented network environment using a custom Amazon Virtual Private Cloud (VPC). The design separates public-facing resources from internal resources using public and private subnets, a bastion host, route tables, and controlled security groups.

## Step by Step:

### 1. VPC & Subnet Design

- A custom VPC was created with the CIDR block 10.0.0.0/16.
- Inside the VPC, two subnets were defined:
  - Public Subnet (10.0.1.0/24) - Hosts resources that require direct internet access, such as the bastion host and NAT Gateway.
  - Private Subnet (10.0.2.0/24) - Hosts internal EC2 instances that should not be directly reachable from the internet.

This segmentation ensures that only approved entry points exist and internal servers remain isolated.

### 2. Internet Connectivity

To provide internet access, two components are used:

- Internet Gateway (IGW) - Attached to the VPC and used by the public subnet to enable inbound and outbound internet traffic.

- NAT Gateway (NGW) - Deployed in the public subnet, allowing instances in the private subnet to make outbound-only internet requests (for software updates, package downloads, etc.) while still preventing any inbound connections from the internet.

### 3. EC2 Instances

Two EC2 instances were deployed:

- Public EC2 Instance: Bastion Host

Located in the public subnet and assigned a public IP address. This instance is used as a secure jump box to reach the private instance. Only trusted administrator IP addresses are allowed to SSH into it.

- Private EC2 Instance

Located in the private subnet with no public IP. This instance cannot be accessed directly from the internet. It can only be reached via SSH from the bastion host using an internal (private) IP address.

### 4. Routing & Security Controls

- Public Route Table - Associated with the public subnet, default route: 0.0.0.0/0 → Internet Gateway

- Private Route Table - Associated with the private subnet, default route: 0.0.0.0/0 → NAT Gateway

This ensures that private subnet instances can access the internet only through the NAT.

- Security Groups

Two security groups were configured:

1- Bastion SG
- Inbound: SSH (22) allowed only from the administrator's IP

- Outbound: All traffic allowed

2- Private SG
- Inbound: SSH (22) allowed only from the bastion host’s security group

- Outbound: All traffic allowed

This configuration ensures the private instance has no direct exposure to the internet or external networks.

### 5. Bastion Host – Bonus Feature

A bastion host (jump box) was implemented as part of the optional bonus challenge.
This server provides a secure entry point into the VPC for administrative SSH access and must be used to reach the private EC2 instance.

