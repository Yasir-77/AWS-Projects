# AWS Networking Assignment: VPC and Network Setup

## Objective:
Create a custom VPC with both public and private subnets, configure internet access, and deploy EC2 instances with secure segmentation. 

## Requirements: 
1- VPC & Subnets: Create a custom VPC with at least one public and one private subnet. 
2- Internet & NAT Gateway: Attach an Internet Gateway (IGW). Deploy a NAT Gateway (NGW) in the public subnet. 
3- EC2 Instances: Launch one EC2 instance in the public subnet (with internet access). Launch another EC2 instance in the private subnet (no direct internet access). 
4- Routing & Security: Configure route tables for correct routing. Restrict direct access to the private instance. 
5- Bonus Challenge (Optional): Set up a Bastion Host (Jump Box)

## Project Summary:
This project implements a secure network architecture on AWS using a custom VPC, public and private subnets, a bastion host, and controlled routing. It demonstrates how to safely expose public resources while isolating private compute instances from direct internet access.
