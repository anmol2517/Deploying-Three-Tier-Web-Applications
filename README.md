![3TierArch](https://github.com/user-attachments/assets/b7e351ac-eb8b-49fa-9116-fbc45ca63084)



## Intro
When building a cloud-based application, the underlying architecture and environment are just as critical as the application itself. There are many considerations when deciding on the proper architecture of your app:
> Scalability: How easily and/or frequently does the app need to scale up or down? How much value do you put into not having to constantly micro-manage and monitor resource usage?
> Availability: How readily available is your app? How important is being able to go through long periods of time without failures? If failure does occur in a part of your app, how vulnerable is the rest?
> Security: How secure is your app? How does your app handle security permissions for different parts of your app? If an attack happens in one part of your app, how vulnerable is the rest?

## Enter the AWS 3-tier architecture
> Why 3-tier? This form of architecture addresses all the issues stated above. it provides increased scalability, availability, and security by spreading the application into multiple Availability Zones and separating it into three layers that serve different functions, independent of each other. If an AZ does down for some reason, the application has the ability to automatically scale resources to another AZ, without affecting the rest of the application tiers. Each tier has its own security group that only allows the inbound/outbound traffic needed to perform specific tasks.

1. Web/Presentation Tier: Houses the user-facing elements of the application, such as web servers and the interface/frontend.
2. Application Tier: Houses the backend and application source code needed to process data and run functions.
3. Data Tier: Houses and manages the application data. Often where the databases are stored.

So what does this mean for you? You’re a Dev Ops Engineer for Brainiac, a new, rapidly growing tech startup. The Product team wants to build a new web app. You’re tasked with planning and building the architecture the application will be run on.

## Prerequisites
1. An AWS account with IAM user access.
2. Familiarity with the AWS Management Console.
3. Familiarity with VPC network structures, EC2 instances, Auto scaling groups, and security groups.
4. Familiarity with Linux commands, scripting, and SSH.
5. Access to a command line tool.

## Prior reading
This topic can get pretty dense and detailed, so if you’re unfamiliar with some of the topics such as Auto Scaling Groups, EC2 instances, or the AWS CLI, I highly recommend going through some of my prior articles as a refresher.

>> Automating an Apache web server with an Amazon EC2 instance: A step-by-step guide
>> Using the AWS CLI to launch an Amazon EC2 instance with an Apache web server
>> Leveraging high availability by creating an AWS Auto scaling group & application load balancer

## The underlying network architecture
Like any overused cliché, you can’t build a house without a solid foundation. To make things a bit easier down the road, we’re going to create the base environment upon which our 3-tier application architecture will be built.

This base network consists of:

- A VPC.
- Two (2) public subnets spread across two availability zones (Web Tier).
- Two (2) private subnets spread across two availability zones (Application Tier).
- Two (2) private subnets spread across two availability zones (Database Tier).
- One (1) public route table that connects the public subnets to an internet gateway.
- One (1) private route table that will connect the Application Tier private subnets and a NAT gateway.

![image](https://github.com/user-attachments/assets/4e8c2553-0f5d-4eab-a949-3d7a1120ce7b)

This seems daunting, but if we take it one step at a time, the path will become clearer and clearer.

Luckily, we can do most of this in one step with the AWS Management Console (minus some extraneous cleanup).

In the VPC console, let’s create a new VPC. We’ll select the ‘VPC and more’ option and name our project ‘brainiac-webApp’ with a CIDR block of 10.0.0.0/16.

To increase the availability of our Brainiac application, we’ll use two AZs (us-east-1a and us-east-1b), two public subnets, and four private subnets (We’ll add a NAT gateway later when we’re ready to build out the Application tier).

![image](https://github.com/user-attachments/assets/e183072c-95a2-4764-89f5-5fe19130d4db)

![image](https://github.com/user-attachments/assets/95b5e82f-e34e-4404-a2ab-de2be65b2f6c)

![image](https://github.com/user-attachments/assets/39987965-469f-4ae6-95a9-0886bad0bc1a)

## Enable auto-assign IPv4
Once all the assets have been created, we need to make sure we ‘Enable auto-assign public IPv4 address’ for BOTH public subnets so we can access its resources via the Internet.

![image](https://github.com/user-attachments/assets/bbea2d81-5a4c-48b9-a8e7-800941472692)

## Create a NAT Gateway
>> A NAT gateway allows instances from the private subnets to connect to resources outside of the VPC and the Internet (for necessary services such as patches or package updates).
>> It’s best practice to maintain high availability and deploy two NAT gateways in our public subnets (one in each AZ); however, for now, we will just deploy one.
>> Navigate to ‘NAT Gateways’ and create a new gateway called public-NAT-1. Select one of the public subnets, allocate an elastic IP, and create the gateway.

## Configure private route tables
As we can see, a route table has been created for each private subnet (4) by default. However, we only need one private route table (for the Application Tier subnets). This is where clear naming conventions can immensely help guard against confusion, so let’s navigate to our route tables and clean things up a bit.

>> Select any one of the private route tables and adjust the name to something like ‘brainiac-webApp-rtb-private1.’ This will be our private route table. Now we can associate this table with all four private subnets (-subnet-private1, -subnet-private2, -subnet-private-3, -subnet-private4)

## Tier 1: Web tier (Frontend)
The Web Tier, also known as the ‘Presentation’ tier, is the environment where our application will be delivered for users to interact with. For Brainiac, this is where we will launch our web servers that will host the frontend of our application.

What we’ll build:
>>> A web server launch template to define what kind of EC2 instances will be provisioned for the application.
>>> An Auto Scaling Group (ASG) that will dynamically provision EC2 instances.
>>> An Application Load Balancer (ALB) to help route incoming traffic to the proper targets.

![image](https://github.com/user-attachments/assets/4d84f6f9-5d88-4564-b331-66194c8ecdfa)

## 1. Create a web server launch template
It’s time to create a template that will be used by our ASG to dynamically launch EC2 instances in our public subnets.
In the EC2 console, navigate to ‘Launch templates’ under the ‘Instances’ sidebar menu. We’re going to create a new template called ‘brainiac-webServer’ with the following provisions:

1. AMI: Amazon 2 Linux
2. Instance type: t2.micro (1GB – Free Tier)
3. A new or existing key pair
We’re not going to specify subnets, but we will create a new security group with inbound SSH, HTTP, and HTTPS rules. Make sure the proper brainiac VPC is selected.

## 3. Application load balancer (ALB)
We’ll need an ALB to distribute incoming HTTP traffic to the proper targets (our EC2s). The ALB will be named, ‘brainiac-webServer-alb.’ We want this ALB to be ‘Internet-facing,’ so it can listen for HTTP/S requests.

## Group size
We want to set a minimum and maximum number of instances the ASG can provision:
>> Desired capacity: 2
>> Minimum capacity: 2
>> Maximum capacity: 5

Review the ASG settings and create the group!
Once the ASG is fully initialized, we can go to our EC2 dashboard and see that two EC2 instances have been deployed.
To see if our ALB is properly routing traffic, let’s go to its public DNS. We should be able to access the website we implemented when creating our EC2 launch template.


## Tier 2: Application tier (Backend)
>> The Application Tier is essentially where the heart of our Brainiac app lives. This is where the source code and core operations send/retrieve data to/from the Web and Database tiers.
>> The structure is very similar to the Web Tier but with some minor additions and considerations.

What we will build:
A launch template to define the type of EC2 instances.
An Auto Scaling Group (ASG) to dynamically provision EC2 instances.
An Application Load Balancer (ALB) to route traffic from the Web tier.
A Bastion host to securely connect to our application servers.

![image](https://github.com/user-attachments/assets/1c080b82-54d4-4f08-81cd-1d2d506af225)

## 1. Create an application server launch template
This template will define what kind of EC2 instances our backend services will use, so let’s create a new template called, ‘brainiac-appServer-template.’
We will use the same settings as the brainiac-webServer-template (Amazon 2 Linux, t2.micro-1GB, same key pair).
Our security group settings are where things will differ. Remember, this is a private subnet, where all of our application source code will live. We need to take precautions so it cannot be accessible from the outside.
We want to allow ICMP–IPv4 from the brainiac-webServer-sg, which allows us to ping the application server from our web server.

## 2. Create an Auto Scaling Group (ASG)
Similar to the Web Tier, we’ll create an ASG from the brainiac-appServer-template called, ‘brainiac-appServer-asg.’

## 3. Application Load Balancer (ALB)
Now we’ll create another ALB that routes traffic from the Web Tier to the Application Tier. We’ll name it ‘brainiac-appServer-alb.’

## Confirm connectivity from the Web Tier
Our application servers are up and running. Let’s verify connectivity by pinging the application server from one of the web servers.

>> SSH into the web server EC2 and ping the private IP address of one of the app server EC2s.

"ssh -i "webServer_key.pem" ec2-user@ec2-54-209-250-120.compute-1.amazonaws.com"

## 4. Create a Bastion host
A bastion host is a dedicated server used to securely access a private network from a public network. We want to protect our Application Tier from potential outside access points, so we will create an EC2 instance in the Web Tier, outside of the ASG. This is the only server that will be used as a gateway to our app servers.
In the EC2 console, launch a new instance called, ‘brainiac-bastionHost.’ We’ll use the same provisions as before (Amazon Linux2, t2.micro). Make sure the brainiac-vpc is selected, as well as one of the public subnets.

## Tier 3: Database tier (Data storage & retrieval)
Almost there! Now it's time to build the last tier of our Brainiac application architecture: the Database. Every application needs a way to store important data, such as user login info, session data, transactions, application content, etc… Our application servers need to be able to read and write to databases to perform necessary tasks and deliver proper content/services to the Web Tier and users.

>> We are going to use a Relational Database Service (RDS) that uses MySQL.
What we’ll build:
1. A database security group that allows outbound and inbound mySQL requests to and from our app servers.
2. A DB subnet group to ensure the database is created in the proper subnets.
3. An RDS database with MySql.

## 1. Create a database security group
Our application servers need a way to access the database, so let’s first create a security group that allows inbound traffic from the application servers.
Create a new security group called, ‘brainiac-db-sg.’ Make sure the Brainiac vpc is selected.

## 2. Create a DB subnet group
In the RDS console, under the ‘Subnet groups’ sidebar menu, create a new subnet group called, ‘brainiac-db-subnetGroup.’ Make sure the brainiac-vpc is selected.

## 3. Create an RDS database
Under the RDS console and the ‘Databases’ sidebar menu, create a new database with a MySQL engine.

## Connect to the Database
After the DB has been created, we’ll need the database endpoint to establish a connection from the app server.

![image](https://github.com/user-attachments/assets/0708310e-8c6b-47fc-ac88-888e6ff0cdae)
