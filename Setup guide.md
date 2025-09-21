Step 1: Create VPC and Network Infrastructure
1.1 Create VPC

Console Steps:

Go to VPC Dashboard → Create VPC Name:Moodle VPC-vpc: 10.0.0.0/16 IPv6 CIDR: No IPv6 Tenancy: Default
<img width="1892" height="750" alt="image" src="https://github.com/user-attachments/assets/8de8d2e1-7dc3-4c32-b6a9-5bf9ff674ff1" />

1.2 Create Subnets (4 subnets total)

Public Subnets:

Public Subnet1:- 10.0.0.0/20 (ap-south-1a) Public Subnet2:- 10.0.16.0/20 (ap-south-1b)

Private Subnets:

Private Subnet1:- 10.0.128.0/20 (ap-south-1a) Private Subnet2:- 10.0.144.0/20 (ap-south-1b)
<img width="1912" height="748" alt="image" src="https://github.com/user-attachments/assets/e5cdb031-959a-4425-aadb-7e74c838736e" />


1.3 Create Internet Gateway

Create Internet Gateway → Name: Moodle-igw Attach to Moodle VPC-vpc

<img width="1905" height="758" alt="image" src="https://github.com/user-attachments/assets/a9f00945-4c90-4018-ab1d-d0b3e9e275ed" />


1.4 Create NAT Gateways

Create NAT Gateway in each public subnet NAT-AZ-A in Public Subnet AZ-A NAT-AZ-B in Public Subnet AZ-B Allocate Elastic IPs for each
<img width="1892" height="750" alt="image" src="https://github.com/user-attachments/assets/460a9ce0-5e85-4279-8ba2-6adbbb82b468" />


1.5 Configure Route Tables

Public Route Table:

Route: 0.0.0.0/0 → Internet Gateway Associate with public subnets

Private Route Tables (create 2):

Private RT AZ-A: 0.0.0.0/0 → NAT Gateway AZ-A Private RT AZ-B: 0.0.0.0/0 → NAT Gateway AZ-B
<img width="1907" height="760" alt="image" src="https://github.com/user-attachments/assets/454ec50f-1697-403a-979c-86d99c1147a0" />


Step 2: Create Security Groups
2.1 ALB Security Group

Name:moodle-alb-sg

Inbound Rules:

HTTP (80) - 0.0.0.0/0

HTTPS (443) - 0.0.0.0/0

2.2 EC2 Security Group

Name: moodle-ec2-sg

Inbound Rules: HTTP (80) - ALB Security Group

SSH (22) - Your IP (for management)

2.3 Database Security Group

Name: moodle-db-sg Inbound Rules:

MySQL (3306) - EC2 Security Group

2.4 EFS Security Group

Name:moodle-efs-sg

Inbound Rules:

NFS (2049) - EC2 Security Group
<img width="1898" height="756" alt="image" src="https://github.com/user-attachments/assets/f83a6ca1-c74e-469b-9a86-4847d9b6a496" />


Step 3: MySql Databas:-
=
3.1 Create DB Subnet Group

Go to RDS → Subnet Groups → Create

Name: moodle-db-subnet-group

VPC: Select your VPC

Add both private subnets

3.2 Create MySql

RDS → Create Database

Engine: MySql

Version: Latest

Templates: Production

Cluster identifier:HA-MySql

Master username: admin

Master password: [secure-password]

Instance class: db.t3.medium (start small, can scale)

Multi-AZ: Yes

VPC: Your VPC

Subnet group: HA-db-subnet-group

Security group: HA-db-sg
<img width="1901" height="761" alt="image" src="https://github.com/user-attachments/assets/64327637-ba59-4fe4-bd1f-71d5b33de0cb" />

<img width="1900" height="760" alt="image" src="https://github.com/user-attachments/assets/76c1997f-75bc-46b4-ab78-519dfac72085" />


Step 4: Create EFS File System
=
4.1 Create EFS

EFS → Create File System

Name: moodle-efs

VPC: Moodle VPC-vpc

Availability and Durability: Regional

Performance mode: General Purpose

Throughput mode: Provisioned

4.2 Create Mount Targets

Create mount targets in both private subnets


<img width="1893" height="757" alt="image" src="https://github.com/user-attachments/assets/29b9f557-abce-4470-a27c-caf116f0d676" />


Step 5: Create Launch Template and Auto Scaling
=
5.1 Create Launch Template

bash# Name: moodle-launch-template

AMI: Amazon Linux 2023

Instance type: t2.micro

Key pair: eth1

Security group: Moodle-ec2-sg

User Data Script:

#!/bin/bash
yum install -y httpd
echo "Test" > /var/www/html/index.html
systemctl enable httpd
systemctl start httpd

<img width="1898" height="755" alt="image" src="https://github.com/user-attachments/assets/3e2eff90-d2ad-4b67-a826-90f65528a190" />


5.2 Create Auto Scaling Group
=

EC2 → Auto Scaling Groups → Create

Name: moodle-asg

Launch template: Moodle-launch-template

VPC: Moodle VPC-vpc

Subnets: Both private subnets

Desired: 3, Min: 3, Max: 6

Health check: ELB

Health check grace period: 300 seconds
<img width="1896" height="757" alt="image" src="https://github.com/user-attachments/assets/e63c2236-018b-4273-a1b2-4ebc83929479" />


Step 6: Create Application Load Balancer
=
6.1 Create ALB

EC2 → Load Balancers → Create ALB

Name: moodle-alb

Scheme: Internet-facing

VPC: Moodle VPC-vpc

Subnets: Both public subnets

Security group: moodle-alb-sg

<img width="1908" height="751" alt="image" src="https://github.com/user-attachments/assets/5e42e0c5-e7cb-45fa-b6de-f1230103a6d2" />


6.2 Create Target Group

Name: moodle-targets

Protocol: HTTP

Port: 80

VPC: Moodle VPC-vpc

Health check path: /moodle/
<img width="1913" height="757" alt="image" src="https://github.com/user-attachments/assets/f1860b85-dc77-4650-9222-858a5ef7daa4" />





























