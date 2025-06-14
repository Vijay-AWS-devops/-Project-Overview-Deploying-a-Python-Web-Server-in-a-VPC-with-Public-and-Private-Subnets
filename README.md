# -Project-Overview-Deploying-a-Python-Web-Server-in-a-VPC-with-Public-and-Private-Subnets
In this project, we’ll walk through how to deploy a small Python web server on EC2 instances placed in a private subnet.

We start by creating a VPC with one public subnet and one private subnet, each in two different Availability Zones (AZs). The public subnet hosts a bastion host, which allows secure access to the private subnet.
In the private subnet, we deploy two servers using an Auto Scaling Group, and we place them behind a Load Balancer. 

The load balancer itself is located in the public subnet, making it accessible from the internet. This setup helps distribute traffic across both AZs and improves availability and fault tolerance.
To connect to the EC2 instances in the private subnet, we use the bastion host as a jump box.

Since the private subnet doesn’t have direct internet access, we deploy a NAT Gateway in the public subnet of each AZ. This allows the private instances to reach the internet for updates or external dependencies while remaining secure and hidden from public exposure.

VPC Workflow diagram:

![VPC Workflow dia](https://github.com/user-attachments/assets/708164ab-6cc7-486e-af1c-3a40ee8765b0)


CREATE VPC
![image](https://github.com/user-attachments/assets/dfb1007b-2637-450a-a528-84738f302798)

**1. VPC**
**Name** : AWS_Project_Prod-vpc
**Description** : This is the main virtual network for the project.
**Purpose** : Provides a secure and isolated environment for deploying resources.

**2. Subnets (4)**
The VPC contains four subnets , distributed across two Availability Zones (us-east-1a and us-east-1b). These subnets are categorized into public and private subnets:

Availability Zone: us-east-1a
Public Subnet : AWS_Project_Prod-subnet-public1-us-east-1a
Purpose: Hosts resources that need direct internet access.

Private Subnet : AWS_Project_Prod-subnet-private1-us-east-1a
Purpose: Hosts resources that should not be exposed to the public internet.

Availability Zone: us-east-1b
Public Subnet : AWS_Project_Prod-subnet-public2-us-east-1b
Purpose: Similar to the public subnet in us-east-1a.

Private Subnet : AWS_Project_Prod-subnet-private2-us-east-1b
Purpose: Similar to the private subnet in us-east-1a.

**3. Route Tables (3)**
Route tables define how traffic flows within the VPC. The image shows three route tables:

Public Route Table : AWS_Project_Prod-rtb-public
Associated with:
AWS_Project_Prod-subnet-public1-us-east-1a
AWS_Project_Prod-subnet-public2-us-east-1b
Purpose: Routes traffic from public subnets to the internet via an Internet Gateway (AWS_Project_Prod-igw).

Private Route Table 1 : AWS_Project_Prod-rtb-private1-us-east-1a
Associated with:
AWS_Project_Prod-subnet-private1-us-east-1a
Purpose: Routes outbound traffic from private subnets in us-east-1a to a NAT Gateway (AWS_Project_Prod-nat-public1-us-east-1a).

Private Route Table 2 : AWS_Project_Prod-rtb-private2-us-east-1b
Associated with:
AWS_Project_Prod-subnet-private2-us-east-1b
Purpose: Routes outbound traffic from private subnets in us-east-1b to a NAT Gateway (AWS_Project_Prod-nat-public2-us-east-1b).

**4. Network Connections (3)**
This section shows the external network connections associated with the VPC:
Internet Gateway : AWS_Project_Prod-igw
Purpose: Provides internet connectivity to public subnets.

NAT Gateway 1 : AWS_Project_Prod-nat-public1-us-east-1a
Purpose: Enables private subnets in us-east-1a to access the internet.

NAT Gateway 2 : AWS_Project_Prod-nat-public2-us-east-1b
Purpose: Enables private subnets in us-east-1b to access the internet.

**🏁 Key Takeaways**
High Availability: Resources are spread across two AZs (us-east-1a and us-east-1b) to ensure fault tolerance.
Security: Public and private subnets separate exposed resources from internal ones.
Internet Connectivity: Public subnets use an Internet Gateway for direct internet access. Private subnets use NAT Gateways for outbound internet access while remaining hidden from the public internet.








