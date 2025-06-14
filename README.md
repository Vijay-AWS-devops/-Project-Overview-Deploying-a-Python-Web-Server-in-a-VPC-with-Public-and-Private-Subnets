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

# 🌐 AWS VPC Architecture Overview

This document provides a high-level overview of the VPC (Virtual Private Cloud) architecture for the project `AWS_Project_Prod`.

---

## 1. **VPC**

- **Name**: `AWS_Project_Prod-vpc`
- **Purpose**: A secure and isolated network environment for deploying cloud resources.

---

## 2. **Subnets (4)**

The VPC includes four subnets across two Availability Zones (`us-east-1a` and `us-east-1b`), divided into public and private subnets:

| AZ             | Public Subnet                              | Purpose                                      |
|----------------|--------------------------------------------|----------------------------------------------|
| us-east-1a     | `AWS_Project_Prod-subnet-public1-us-east-1a` | For internet-facing resources                |
| us-east-1a     | `AWS_Project_Prod-subnet-private1-us-east-1a`| For internal-only resources                  |
| us-east-1b     | `AWS_Project_Prod-subnet-public2-us-east-1b` | Same as above                                |
| us-east-1b     | `AWS_Project_Prod-subnet-private2-us-east-1b`| Same as above                                |

---

## 3. **Route Tables (3)**

| Route Table Name                          | Associated Subnets                                       | Gateway/Resource Used         | Purpose                                           |
|------------------------------------------|-----------------------------------------------------------|-------------------------------|---------------------------------------------------|
| `AWS_Project_Prod-rtb-public`            | `AWS_Project_Prod-subnet-public1-us-east-1a`,<br>`AWS_Project_Prod-subnet-public2-us-east-1b` | Internet Gateway              | Routes traffic to the internet                    |
| `AWS_Project_Prod-rtb-private1-us-east-1a`| `AWS_Project_Prod-subnet-private1-us-east-1a`             | NAT Gateway 1                 | Enables outbound access from private subnet       |
| `AWS_Project_Prod-rtb-private2-us-east-1b`| `AWS_Project_Prod-subnet-private2-us-east-1b`             | NAT Gateway 2                 | Same purpose as above                             |

---

## 4. **Network Connections (3)**

| Resource Name                            | Purpose                                                   |
|------------------------------------------|------------------------------------------------------------|
| `AWS_Project_Prod-igw`                   | Provides internet access to public subnets                 |
| `AWS_Project_Prod-nat-public1-us-east-1a`| Allows private subnets in us-east-1a to access the internet|
| `AWS_Project_Prod-nat-public2-us-east-1b`| Same as above but for us-east-1b                           |

---

## 🏁 Key Takeaways

✅ **High Availability**  
Resources are distributed across two Availability Zones for fault tolerance.

✅ **Security**  
Public and private subnets help isolate sensitive workloads from direct internet exposure.

✅ **Internet Access**  
- Public subnets use an **Internet Gateway**.
- Private subnets use **NAT Gateways** for outbound-only internet access.


Sure! Here's the **updated README.md** with the steps renumbered accordingly — specifically, **Step 1 is now Step 2**, and the rest are adjusted to reflect that change.

---

# 🚀 AWS Auto Scaling Group Setup (Private Subnet)

This steps to create an **Auto Scaling Group (ASG)** in a **private subnet**, using a **Launch Template** with an Ubuntu T2.micro EC2 instance and a custom **Security Group** that allows SSH and traffic on port 8000.

---

## 🛠️ Prerequisites

- An existing VPC named: `AWS_Project_Prod-vpc`
- Two private subnets in Availability Zones:
  - `us-east-1a`: `AWS_Project_Prod-subnet-private1-us-east-1a`
  - `us-east-1b`: `AWS_Project_Prod-subnet-private2-us-east-1b`

---

## 🧱 Step 1: Create a Launch Template

| Field               | Value |
|--------------------|-------|
| Name               | `AWS_Project_Prod-launch-template` |
| Description        | Launch template for Ubuntu T2.micro instances |
| AMI                | Latest Ubuntu Server |
| Instance Type      | `t2.micro` |
| Network            | `AWS_Project_Prod-vpc` |
| Security Group     | `AWS_Project_Prod-sec-group` |
| Tags               | `Name: AWS_Project_Prod-instance` |

---

## 🔐 Step 2: Create a Security Group

| Field             | Value |
|------------------|-------|
| Name             | `AWS_Project_Prod-sec-group` |
| Description      | Security group for production environment |
| VPC              | `AWS_Project_Prod-vpc` |
| Inbound Rules    | - SSH (Port 22) from your IP or `0.0.0.0/0`<br>- Custom TCP (Port 8000) from your IP or `0.0.0.0/0` |

![image](https://github.com/user-attachments/assets/afb4f314-cc64-45d0-a34f-ea3a3afba2be)


---

## 🔄 Step 3: Create an Auto Scaling Group

| Field                  | Value |
|-----------------------|-------|
| Name                  | `AWS_Project_Prod-asg` |
| Launch Template       | `AWS_Project_Prod-launch-template` |
| VPC                   | `AWS_Project_Prod-vpc` |
| Subnets               | Both private subnets (`AWS_Project_Prod-subnet-private1-us-east-1a`, `AWS_Project_Prod-subnet-private2-us-east-1b`) |
| Desired Capacity      | `2` |
| Minimum Capacity      | `1` |
| Maximum Capacity      | `4` |
| Health Check Type     | `EC2` (default) |

![image](https://github.com/user-attachments/assets/d3e00621-f8fc-40d5-89e4-0809f6b24f55)

---

## ✅ Key Takeaways

- **High Availability**: Instances are spread across two AZs for fault tolerance.
- **Scalability**: Automatically scales between 1 and 4 instances based on demand.
- **Security**: Only SSH and port 8000 are open for access.
- **Private Deployment**: All resources are deployed in private subnets for enhanced security.


# 🔐 Setting Up a Bastion Host to Access Private EC2 Instances

This guide will walk you through setting up a **Bastion (Jump) Host** in AWS to securely access EC2 instances in a **private subnet**. You'll also learn how to connect to the private instances using SSH and deploy a simple Python web server on them.

---

## 🛠️ Prerequisites

- An existing VPC: `AWS_Project_Prod-vpc`
- A public subnet within that VPC
- A key pair (.pem or .ppk)
- WinSCP and PuTTY tools (if on Windows)

---

## 🧱 Step 1: Create the Bastion Host

1. **Launch an EC2 Instance**
   - **AMI**: Ubuntu Server (latest LTS)
   - **Instance Type**: `t2.micro`
   - **VPC**: `AWS_Project_Prod-vpc`
   - **Subnet**: Select a **public subnet** (e.g., `AWS_Project_Prod-subnet-public1-us-east-1a`)
   - **Public IP**: Enable "Auto-assign Public IP"
   - **Security Group**: Allow SSH from your IP address or `0.0.0.0/0` (for testing)

2. **Name the instance** something like: `AWS_Project_Bastion`

---

## 🔐 Step 2: Prepare the Key Pair

1. If your key is in `.ppk` format:
   - Use **PuTTYgen** to convert it to `.pem`:
     - Open PuTTYgen
     - Load your `.ppk` file
     - Click **Save private key** and choose `.pem` format

2. Transfer the `.pem` file to the Bastion host:
   - Use **WinSCP** to copy the `.pem` file to the Bastion host (e.g., `/home/ubuntu/your-key.pem`)

3. Change the file permissions:
   ```bash
   chmod 600 /home/ubuntu/your-key.pem
   ```

---

## 🔌 Step 3: Connect to Private EC2 Instance via Bastion Host

From the Bastion host terminal:

```bash
ssh -i /home/ubuntu/your-key.pem ubuntu@<Private_EC2_Instance_IP>
```

> ✅ Example:
```bash
ssh -i /home/ubuntu/AWS_prod.pem ubuntu@172.1.1.1
```
![image](https://github.com/user-attachments/assets/79e40cd4-e973-4cd1-9a75-f2a120de9ca2)

---

## 🖥️ Step 4: Deploy a Simple Python Web Server

Once connected to the private EC2 instance, run the following commands:

### Option 1: Serve a Static HTML File

1. Create an HTML file with the following content:

```html
<!DOCTYPE html>
<html>
<head>
  <title>Page Title</title>
</head>
<body>

<h1>AWS Project in private server 1</h1>
<p>Successful.</p>

</body>
</html>
```

You can create this file using the command line:

```bash
vim index.html
```

Paste the above code and save the file (`Ctrl+O`, `Enter`, then `Ctrl+X`).

2. Start a Python HTTP server:
   ```bash
   python3 -m http.server 8000
   ```

3. The server will now be running on port 8000.

---

## 🔄 Step 5: Repeat on the Second EC2 Instance

1. Connect to the second private EC2 instance using its IP.
2. Create a different HTML file (you can change the title and message slightly):
   Absolutely! Here's a **clean and user-friendly HTML file** that you can use for the **second EC2 instance**, with a slightly different title and message to distinguish it from the first one.

You can copy this content into an `index.html` file on your second EC2 instance using a text editor like `nano` or `vim`.

---

### ✅ HTML File for Second Private Server (Instance 2)

```html
<!DOCTYPE html>
<html>
<head>
  <title>AWS Project - Private Server 2</title>
</head>
<body>

<h1>Welcome to AWS Project - Private Server 2</h1>
<p>This is the second server in the private subnet.</p>

</body>
</html>
```

3. Start the server again:
   ```bash
   python3 -m http.server 8000
   ```

Now both servers are running and can be accessed individually. You can later test **load balancing** between these two endpoints.

---

## ✅ Key Takeaways

- The **Bastion Host** acts as a secure gateway to private resources.
- You can only reach private EC2 instances **through** the Bastion Host.
- Using **different HTML files** helps distinguish traffic when testing with a load balancer.

---

Here's a **clean and well-structured README.md** file for your GitHub repository, explaining how to set up an **Application Load Balancer (ALB)** in AWS and test it with Python web servers running on EC2 instances in a private subnet.

---

# 🌐 AWS Application Load Balancer Setup

This guide walks you through setting up an **Application Load Balancer (ALB)** in your existing VPC (`AWS_Project_Prod`) and testing it by routing traffic to Python HTTP servers running on two EC2 instances in a **private subnet**.

---

## 🛠️ Prerequisites

- A VPC named `AWS_Project_Prod` with:
  - Two **public subnets**, each in a different Availability Zone.
- Two **EC2 instances** in the **private subnet** of the same VPC.
- A **Bastion Host** in the public subnet to securely access the private instances.
- Python 3 installed on both private EC2 instances.
- An HTML file already created on both instances 

---

## 🔄 Step 1: Create an Application Load Balancer (ALB)

1. **Navigate to the AWS Console**
   - Go to **EC2 Dashboard**
   - In the left sidebar, click **Load Balancers**

2. **Create a New Load Balancer**
   - Choose **Application Load Balancer**
   - Click **Create**

3. **Configure the Load Balancer**
   - **Name**: `AWS_Project_ALB`
   - **Scheme**: Internet-facing
   - **VPC**: Select `AWS_Project_Prod`
   - **Subnets**: Select the **two public subnets** in different AZs
   - **Security Group**: Allow HTTP (Port 80) from `0.0.0.0/0`

![image](https://github.com/user-attachments/assets/4aba3230-35fd-4244-b17c-d50771d3bf46)


4. **Create a Target Group**
   - **Target Type**: IP addresses or instances (choose based on your preference)
   - **Protocol**: HTTP
   - **Port**: 8000
   - **Health Check Path**: `/` or any endpoint that returns 200 OK
   - Add both private EC2 instance IPs or register them as targets

5. **Add Listeners**
   - **HTTP Port 80 → Forward to your target group**

6. **Review and Create**
   - Click **Create Load Balancer**

7. **Copy the DNS Name**
   - After creation, copy the **DNS name** of the ALB (e.g., `aws-project-alb-123456789.us-east-1.elb.amazonaws.com`)

---

## 🔧 Step 2: Start Python Web Server on Private Instances

From the **Bastion Host**, SSH into one of the private EC2 instances:

```bash
ssh -i /home/ubuntu/AWS_prod.pem ubuntu@<Private_EC2_Instance_IP>
```

Once connected, start the Python web server:

```bash
python3 -m http.server 8000
```

Repeat this step for the **second private EC2 instance** using its IP address.

> Make sure each instance is serving a unique HTML file so you can see which one the load balancer routes to.

---

## 🌐 Step 3: Test the Load Balancer

1. Open a browser on your local machine.
2. Enter the **ALB’s DNS name** in the address bar:
   ```
   http://aws-project-alb-123456789.us-east-1.elb.amazonaws.com
   ```

3. You should see the HTML page served by one of the private EC2 instances.
4. Refresh the page — depending on your ALB settings, it may route to either instance.

![image](https://github.com/user-attachments/assets/8fbf3573-2ca6-4db8-b717-03ea988ef9b6)


---

## ✅ Key Takeaways

- The **Application Load Balancer** distributes incoming traffic across multiple private EC2 instances.
- Traffic is routed via the **public DNS name** of the ALB.
- Both instances are in a **private subnet**, but still accessible via the ALB.
- This setup improves **availability** and provides a scalable way to serve web content.

Sure! Here's a **simple and clear conclusion** you can add to your README file or project documentation:

---

## ✅ Conclusion

In this project, we set up a secure and scalable cloud environment using AWS. We created a VPC with public and private subnets across two Availability Zones, deployed EC2 instances in the private subnet, and used a Bastion Host to securely access them. We also set up an Application Load Balancer to distribute traffic between two Python web servers running on the private instances.

This setup demonstrates how to build a resilient, high-availability architecture while keeping sensitive resources protected in a private network. It’s a great foundation for deploying real-world applications in the cloud.

--- 







