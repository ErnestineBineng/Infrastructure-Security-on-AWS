# Infrastructure-Security-on-AWS
This lab guides you through building and securing a two-tier architecture on AWS using:

A VPC with public and private subnets
An internet-facing Application Load Balancer (ALB)
EC2 instances in private subnets
AWS WAF in front of the ALB
AWS Systems Manager Session Manager instead of direct SSH
The end result is a secure, internet-facing application entry point with hardened backend instances and centralized web security controls.
Architecture Overview

We will deploy:
VPC: lab3-vpc with CIDR 10.0.0.0/16
Subnets:
Public: 10.0.0.0/24 (AZ a), 10.0.1.0/24 (AZ b)
Private: 10.0.3.0/24 (AZ a), 10.0.4.0/24 (AZ b)
Internet Gateway for public subnets
NAT Gateway for outbound-only internet access from private subnets
Application Load Balancer in public subnets
EC2 instances in private subnets, serving an app on port 8080
AWS WAF Web ACL attached to the ALB
Session Manager for shell access (no SSH from the internet)

A. Secure 2‑Tier VPC
1. Create the VPC
In the AWS Console, go to VPC → Create VPC.
Choose VPC only and configure:
Name: lab3-vpc
IPv4 CIDR: 10.0.0.0/16
Click Create.
2. Create Public and Private Subnets
Create four subnets in lab3-vpc:

Public subnets (for ALB):
10.0.0.0/24 in AZ a
10.0.1.0/24 in AZ b
Private subnets (for app instances):
10.0.3.0/24 in AZ a
10.0.4.0/24 in AZ b
Optional: For public subnets, enable Auto-assign public IP so resources there can receive public IPs.

3. Internet Gateway, NAT Gateway, and Route Tables
Internet Gateway

In VPC, go to Internet Gateways → Create Internet Gateway.
Attach it to lab3-vpc.
NAT Gateway

Go to NAT Gateways → Create NAT Gateway.
Place it in one of the public subnets.
Allocate and associate an Elastic IP.
Route Tables

Public route table:
Add route: 0.0.0.0/0 → Internet Gateway.
Associate it with the two public subnets.
Private route table:
Add route: 0.0.0.0/0 → NAT Gateway.
Associate it with the two private subnets.
This ensures:

Public subnets have direct internet access via IGW.
Private subnets have outbound-only access via NAT GW.

4. Create the Application Load Balancer (ALB)
In EC2 → Load Balancers → Create Load Balancer → Application Load Balancer.
Configure:
Name: lab1ALB
Scheme: Internet-facing
Listener: HTTP, port 80
VPC: lab3-vpc
Subnets: both public subnets
Create a security group mylab3securitygroup:
Inbound:
Allow HTTP (port 80) from 0.0.0.0/0
Create a target group:
Name: lab3-app-tg
Target type: Instances or IP (as preferred)
Port: 8080 (to match the app on EC2)
Link the target group to the ALB (listener → forward to lab1ALB).
You will register EC2 instances after creating them.
5. Launch EC2 Instances in Private Subnets
Launch at least one EC2 instance in each private subnet:
One in 10.0.11.0/24 (AZ a)
One in 10.0.12.0/24 (AZ b)
Create a security group app-sg:
Inbound:
Allow HTTP (port 8080) only from alb-sg
No inbound SSH from the internet.
Add user data to run a simple web app on port 8080 (for example, a basic HTTP server).
Register these instances with the lab3-app-tg target group.
Wait for targets to become healthy.
At this point, HTTP requests to the ALB on port 80 should be forwarded to the EC2 instances on port 8080 in the private subnets.





Learning Objectives
By completing this lab, you will:

Design and deploy a two-tier VPC with public and private subnets.
Configure secure routing using an Internet Gateway, NAT Gateway, and dedicated route tables.
Deploy an internet-facing ALB that fronts private EC2 instances.
Protect web traffic with AWS WAF using managed rule groups.

