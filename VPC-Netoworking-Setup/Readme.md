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
