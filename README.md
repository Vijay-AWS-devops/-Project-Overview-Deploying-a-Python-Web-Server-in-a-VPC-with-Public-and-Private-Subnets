# -Project-Overview-Deploying-a-Python-Web-Server-in-a-VPC-with-Public-and-Private-Subnets
In this project, we’ll walk through how to deploy a small Python web server on EC2 instances placed in a private subnet.
We start by creating a VPC with one public subnet and one private subnet, each in two different Availability Zones (AZs). The public subnet hosts a bastion host, which allows secure access to the private subnet.
In the private subnet, we deploy two servers using an Auto Scaling Group, and we place them behind a Load Balancer. The load balancer itself is located in the public subnet, making it accessible from the internet. This setup helps distribute traffic across both AZs and improves availability and fault tolerance.
To connect to the EC2 instances in the private subnet, we use the bastion host as a jump box.
Since the private subnet doesn’t have direct internet access, we deploy a NAT Gateway in the public subnet of each AZ. This allows the private instances to reach the internet for updates or external dependencies while remaining secure and hidden from public exposure.



