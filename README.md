# **AWS 3-Tier Architecture Deployment**

![3TierArch](https://github.com/user-attachments/assets/b7e351ac-eb8b-49fa-9116-fbc45ca63084)


## **Introduction**
When building a cloud-based application, the architecture is just as critical as the application itself. The AWS 3-Tier Architecture is designed to address key challenges such as:

- **Scalability**: Ability to scale up or down without micromanagement.
- **Availability**: Ensures minimal downtime and fault tolerance.
- **Security**: Segregates tiers with security groups and isolates resources to limit vulnerabilities.

This project demonstrates deploying a web application using a 3-tier architecture on AWS for **Brainiac**, a tech startup. Each tier—Web, Application, and Database—serves specific functions and operates independently to enhance scalability, security, and availability.

---

## **Why 3-Tier Architecture?**
1. **Web/Presentation Tier** :
   - Handles user interaction.
   - Hosts the frontend interface using web servers.

2. **Application Tier** :
   - Processes data and executes core business logic.
   - Connects the web and database tiers.

3. **Data Tier** :
   - Manages and stores application data.
   - Often uses relational databases such as MySQL.

This separation ensures fault isolation, efficient resource management, and secure access between tiers.

![image](https://github.com/user-attachments/assets/89b9586a-9268-4700-8911-abd9998a6b59)

---

## **Prerequisites**
- An AWS account with IAM user access.
- Knowledge of:
  - VPC networks, EC2 instances, Auto Scaling Groups, and Security Groups.
  - Linux commands, SSH, and scripting.
  - AWS CLI and the AWS Management Console.

---

## **Project Overview**
This project involves creating a robust infrastructure:

1. **Networking Setup** :
   - A VPC with two public and four private subnets spread across two availability zones.
   - Route tables, Internet Gateway, and NAT Gateway configurations.

2. **Tier 1: Web Tier** :
   - EC2 launch template for web servers.
   - Auto Scaling Group (ASG) to manage server instances.
   - Application Load Balancer (ALB) for traffic distribution.

3. **Tier 2: Application Tier**:
   - EC2 launch template for application servers.
   - ASG for dynamic scaling.
   - ALB to route traffic between Web and Application tiers.
   - Bastion host for secure access.

4. **Tier 3: Database Tier**:
   - Relational Database Service (RDS) with MySQL.
   - Database Subnet Group for proper placement in private subnets.
   - Security Group to manage inbound/outbound access.

---

## **Steps to Deploy**

### **1. Networking Setup**
- Create a VPC named `brainiac-webApp` with a CIDR block of `10.0.0.0/16`.
- Configure subnets:
  - Two public subnets for the Web Tier.
  - Four private subnets (two for Application Tier, two for Database Tier).
- Set up:
  - Public Route Table linked to an Internet Gateway.
  - Private Route Table linked to a NAT Gateway for outbound Internet access from private subnets.

### **2. Web Tier (Frontend)**
- **Launch Template**:
  - Create a launch template named `brainiac-webServer`.
  - AMI: Amazon Linux 2, Instance Type: t2.micro (Free Tier eligible).
  - Security Group: Allow inbound HTTP, HTTPS, and SSH traffic.

- **Auto Scaling Group (ASG)**:
  - Configure minimum: 2, maximum: 5 instances.
  - Use the public subnets.

- **Application Load Balancer (ALB)**:
  - Create an Internet-facing ALB named `brainiac-webServer-alb`.
  - Target Group: Web server instances.

### **3. Application Tier (Backend)**
- **Launch Template**:
  - Create a template named `brainiac-appServer-template`.
  - Security Group: Restrict access to the Web Tier security group.

- **Auto Scaling Group (ASG)**:
  - Configure with a minimum of 2 instances in private subnets.

- **Application Load Balancer (ALB)**:
  - Create an internal ALB named `brainiac-appServer-alb`.
  - Route traffic from the Web Tier to the Application Tier.

- **Bastion Host**:
  - Launch an EC2 instance in the Web Tier for secure access to Application Tier instances.

### **4. Database Tier (Data Storage)**
- **Database Security Group**:
  - Allow inbound MySQL traffic from the Application Tier security group.

- **DB Subnet Group**:
  - Associate with private subnets for high availability.

- **RDS Instance**:
  - Create a MySQL database with Multi-AZ deployment enabled.
  - Provide the database endpoint for Application Tier connectivity.

---

## **Validation**
1. Verify Web Tier functionality by accessing the public DNS of the ALB.
2. Test Application Tier connectivity by SSHing into the Bastion Host and pinging the private IP of app servers.
3. Connect to the database using the RDS endpoint and confirm data accessibility.

---

## **Architecture Diagram**
Include an architecture diagram showcasing the 3-tier setup with components such as subnets, ALBs, ASGs, EC2 instances, and the RDS database.

---

## **Key Features**
- **High Availability**: Spread resources across multiple availability zones.
- **Scalability**: ASGs dynamically adjust resources based on demand.
- **Security**: Segregated tiers with specific security group rules.
- **Fault Tolerance**: NAT Gateway and Multi-AZ RDS for redundancy.
---

## **Technologies Used**
- **AWS Services**: VPC, EC2, RDS, ALB, Auto Scaling Groups, NAT Gateway.
- **Database**: MySQL.
- **Instance OS**: Amazon Linux 2.
- **Tools**: AWS CLI, Management Console.
