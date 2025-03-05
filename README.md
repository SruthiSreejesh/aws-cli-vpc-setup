# 🚀 AWS CLI: VPC & EC2 Instance Setup

This guide walks you through creating a **VPC**, **subnets**, **route tables**, **Internet Gateway (IGW)**, **security groups**, **key pairs**, and **EC2 instances** using **AWS CLI** with an IAM user.
Alright, let's break down how to build a virtual network in Amazon Web Services (AWS) from scratch, using just the command line. Think of it as constructing your own private section of the internet, tailored to your needs. We'll be using the AWS Command Line Interface (CLI), which lets you talk directly to AWS services.

Now, if you're brand new to AWS, don't worry! We'll take it step-by-step. We're going to create the following:

**A Virtual Private Cloud** (VPC): This is your isolated network space within AWS.

**Subnets**: These are like smaller, segmented networks within your VPC. We'll have both public and private subnets.

**An Internet Gateway**: This is how your public subnets connect to the outside world—the internet.

**Route Tables**: These direct network traffic, telling it where to go.

**Key Pairs**: These are like digital keys, allowing you to securely access your virtual servers.

**Security Groups**: These are like virtual firewalls, controlling what traffic can enter and leave your servers.

**EC2 Instances**: These are your virtual servers, the workhorses of your AWS infrastructure.

To get started, you'll need the AWS CLI installed on your computer. It's like having a translator that lets you speak AWS's language. If you haven't set that up yet, AWS has excellent documentation online to help you. It's essential to have this working before proceeding.As we go through the commands, you'll notice that AWS assigns unique identifiers to each resource you create. These IDs are crucial for linking everything together. I strongly suggest keeping a notepad or text file handy. You'll want to jot down these IDs as you go. Here are the things you should keep track of:

The VPC's unique ID.
The IDs for your public and private subnets.
The Internet Gateway's ID.
The IDs for your public and private route tables.
The name and ID of your Security Group.
The AMI ID you will be using.
The Key pair name.
We'll be working with a lot of commands, and it can seem overwhelming at first. But don't worry, I've put together all the commands in one place. That way, you can easily refer back to them.

Essentially, by following these instructions, you'll be building the fundamental networking blocks that allow you to deploy and manage your applications on AWS. Think of this as the foundation upon which you can build anything.

---

## 📌 Prerequisites

### 1️⃣ Install AWS CLI  
Download and install AWS CLI based on your OS:  
- **Windows**: [AWS CLI MSI Installer](https://awscli.amazonaws.com/AWSCLIV2.msi)  
- **Linux** (Ubuntu/Debian):  
  ```sh
  curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
  unzip awscliv2.zip
  sudo ./aws/install
  ```
- **MacOS**:  
  ```sh
  brew install awscli
  ```

Verify installation:  
```sh
aws --version
```

---

### 2️⃣ Create an IAM User  
1. Log in to AWS Console → **IAM** → **Users** → **Add user**.  
2. Enable **Programmatic access**.  
3. Attach the **AdministratorAccess** policy *(for full access)* or create a custom policy.  
4. Download `.csv` file with **Access Key ID** and **Secret Access Key**.  

Set up AWS CLI with IAM credentials:  
```sh
aws configure
```
Provide:  
- **Access Key ID**  
- **Secret Access Key**  
- **Region (e.g., us-east-1)**  
- **Output format (json, table, or text)**  

---

## 🔧 AWS Infrastructure Setup  

### 3️⃣ Create a VPC  
```sh
aws ec2 create-vpc --cidr-block 172.16.0.0/16 --tag-specifications 'ResourceType=vpc,Tags=[{Key=Name,Value=your-vpc-name}]'
```
* Example: aws ec2 create-vpc --cidr-block 172.16.0.0/16 --tag-specifications 'ResourceType=vpc,Tags=[{Key=Name,Value=CLI-VPC}]' 

**Note** the `VpcId` from the response.

---
### 4️⃣ ENABLE DNS HOSTNAME FOR VPC
```sh
aws ec2 modify-vpc-attribute --vpc-id <your_vpc_id> --enable-dns-hostnames "{\"Value\": true}"
```
* Example:
aws ec2 modify-vpc-attribute --vpc-id vpc-0d7643c00170ac78a --enable-dns-hostnames "{\"Value\": true}"


### 5️⃣ CREATE SUBNETS  
#### ✅ Public Subnet 1  
```sh
aws ec2 create-subnet --vpc-id <your-vpc-id> --availability-zone ap-south-1a --cidr-block 172.16.0.0/18 --tag-specifications 'ResourceType=subnet,Tags=[{Key=Name,Value=your_publicSubnet1_name}]'
```
* Example:
aws ec2 create-subnet --vpc-id vpc-0d7643c00170ac78a --availability-zone ap-south-1a --cidr-block 172.16.0.0/18 --tag-specifications 'ResourceType=subnet,Tags=[{Key=Name,Value=Subnet-Public-1A}]'

#### ✅ Public Subnet 2  
```sh
aws ec2 create-subnet --vpc-id <your-vpc-id> --availability-zone ap-south-1b --cidr-block 172.16.64.0/18 --tag-specifications 'ResourceType=subnet,Tags=[{Key=Name,Value=your_publicSubnet2_name}]'
```
* Example:
aws ec2 create-subnet --vpc-id vpc-0d7643c00170ac78a --availability-zone ap-south-1b --cidr-block 172.16.64.0/18 --tag-specifications 'ResourceType=subnet,Tags=[{Key=Name,Value=Subnet-Public-1B}]'

#### ✅ Private Subnet  
```sh
aws ec2 create-subnet --vpc-id <add-your-vpc-id> --availability-zone ap-south-1c --cidr-block 172.16.128.0/18 --tag-specifications 'ResourceType=subnet,Tags=[{Key=Name,Value=your_privateSubnet_name}]'
```
* Example:
aws ec2 create-subnet --vpc-id vpc-0d7643c00170ac78a --availability-zone ap-south-1c --cidr-block 172.16.128.0/18 --tag-specifications 'ResourceType=subnet,Tags=[{Key=Name,Value= Subnet-Private-1C }]'

**Note** the `SubnetIds` from the response.
---

### 6️⃣ ASSIGN PUBLIC IPV4 
#### ✅ for PUBLIC SUBNET 1
```sh
aws ec2 modify-subnet-attribute --subnet-id <your-public-subnet1-id> --map-public-ip-on-launch
```
* Example:
aws ec2 modify-subnet-attribute --subnet-id subnet-056347fba473db6dd --map-public-ip-on-launch

#### ✅ for PUBLIC SUBNET 2
```sh
aws ec2 modify-subnet-attribute --subnet-id <your-public-subnet2-id> --map-public-ip-on-launch
```
* Example  :
aws ec2 modify-subnet-attribute --subnet-id subnet-06a5baab94e3d42b4--map-public-ip-on-launch

### 7️⃣ CREATE THE INTERNET GATEWAY
```sh
aws ec2 create-internet-gateway --tag-specifications 'ResourceType=internet-gateway,Tags=[{Key=Name,Value=your-vpc-name}]'
```
* Example:
aws ec2 create-internet-gateway --tag-specifications 'ResourceType=internet-gateway,Tags=[{Key=Name,Value=IGW-for-CLI-VPC }]'

**Note** the `IGW Id` from the response.

### 8️⃣ ATTACH THE INTERNET GATEWAY
```sh
aws ec2 attach-internet-gateway --vpc-id <your-vpc-id> --internet-gateway-id <your-igw-id>
```
* Example:
aws ec2 attach-internet-gateway --vpc-id vpc-0d7643c00170ac78a --internet-gateway-id igw-07387ae5732acba92

### 9️⃣ FILTER THE ROUTE TABLE TO FIND THE ID OF THE VPC CREATED ROUTE TABLE
```sh
aws ec2 describe-route-tables --filter "Name=vpc-id, Values=<your-vpc-id>" --query "RouteTables[*].[RouteTableId,VpcId,Tags]"
```
* Example:
aws ec2 describe-route-tables --filter "Name= vpc-id, Values= vpc-0d7643c00170ac78a " --query "RouteTables[*].[RouteTableId,VpcId,Tags]"

### 🔟 ADD A NAME TAG TO THE PRIVATE ROUTE TABLE
```sh
aws ec2 create-tags --resources <your-privateroutetable-id> --tags Key=Name,Value=your-private-routetable-name
```
* Example:
aws ec2 create-tags --resources rtb-0db467fdca4d36e85 --tags Key=Name,Value=Private-Route-CLIVPC

### 1️⃣1️⃣ CREATE PUBLIC ROUTE TABLE 
```sh
aws ec2 create-route-table --vpc-id <your-vpc-id> --tag-specifications 'ResourceType=route-table,Tags=[{Key=Name,Value=your-public-routetable-name}]'
```
* Example:
aws ec2 create-route-table --vpc-id vpc-0d7643c00170ac78a--tag-specifications 'ResourceType=route-table,Tags=[{Key=Name,Value=Public-Route-CLIVPC }]'

**Note** the `RouteTableId`.

### 1️⃣2️⃣  ASSOCIATE THE INTERNET GATEWAY WITH THE PUBLIC ROUTE TABLE
```sh
aws ec2 create-route --route-table-id <your-public-routetable-id> --destination-cidr-block 0.0.0.0/0 --gateway-id <your-igw-id>
```
* Example:
aws ec2 create-route --route-table-id rtb-005755730bba9f85b --destination-cidr-block 0.0.0.0/0 --gateway-id igw-07387ae5732acba92

---
### 1️⃣3️⃣ ASSOCIATE THE PUBLIC SUBNET 1 WITH PUBLIC ROUTE TABLE
```sh
aws ec2 associate-route-table  --subnet-id <your-public-subnet1-id> --route-table-id <your-public-routetable-id>
```
* Example:
aws ec2 associate-route-table  --subnet-id subnet-056347fba473db6dd --route-table-id rtb-005755730bba9f85b

### 1️⃣4️⃣ ASSOCIATE THE PUBLIC SUBNET 2 WITH PUBLIC ROUTE TABLE
```sh
aws ec2 associate-route-table  --subnet-id <your-public-subnet2-id> --route-table-id <your-public-routetable-id>
```
* Example:
aws ec2 associate-route-table  --subnet-id subnet-06a5baab94e3d42b4 --route-table-id rtb-005755730bba9f85b

### 1️⃣5️⃣ ASSOCIATE THE PRIVATE SUBNET WITH PRIVATE ROUTE TABLE
```sh
aws ec2 associate-route-table  --subnet-id <your-private-subnet-id> --route-table-id <your-private-routetable-id>
```
* Example:
aws ec2 associate-route-table  --subnet-id subnet-0400769defe0dd20b --route-table-id rtb-0db467fdca4d36e85

### 1️⃣6️⃣ CREATE A KEY PAIR FOR THE EC2
```sh
aws ec2 create-key-pair --key-name <your-key-pair-name> --query 'KeyMaterial' --output text > <your-key-pair-name.pem>
```
* Example:
aws ec2 create-key-pair --key-name CLI-KEYPAIR --query 'KeyMaterial' --output text > CLI-KEYPAIR.pem

### 1️⃣7️⃣ CREATE SECURITY GROUP
```sh
aws ec2 create-security-group --group-name <your-securitygroup-name> --vpc-id <your-vpc-id> --description "<your-securitygroup-description>" --tag-specifications 'ResourceType=security-group,Tags=[{Key=Name,Value=<your-securitygroup-name-tag>}]'
```
* Example:
aws ec2 create-security-group --group-name CLI-SG --vpc-id vpc-0d7643c00170ac78a --description "SSH,HTTP-allow All" --tag-specifications 'ResourceType=security-group,Tags=[{Key=Name,Value=CLI-SG}]'

### 1️⃣8️⃣ ADD RULES SECURITY GROUP
#### ✅ SSH Rule
```sh
aws ec2 authorize-security-group-ingress --group-id <security-group-id> --protocol tcp --port 22 --cidr 0.0.0.0/0
```
* Example:
aws ec2 authorize-security-group-ingress --group-id sg-084805f43ab469ff0  --protocol tcp --port 22 --cidr 0.0.0.0/0

#### ✅ HTTP Rule 
```sh
aws ec2 authorize-security-group-ingress --group-id <security-group-id> --protocol tcp --port 80 --cidr 0.0.0.0/0
```
* Example:
aws ec2 authorize-security-group-ingress --group-id sg-084805f43ab469ff0  --protocol tcp --port 80 --cidr 0.0.0.0/0

### 1️⃣9️⃣ LAUNCH EC2 WITHOUT THE USE OF USER-DATA IN PUBLIC SUBNET 1
```sh
aws ec2 run-instances --image-id <your-ami-id> --count 1 --security-group-ids <your-security-group-id> --instance-type t2.micro --subnet-id <your-public-subnet1-id> --key-name <your-keypair-name> --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=<your-EC2name-tag>}]'
```
* Example:
aws ec2 run-instances --image-id ami-0d682f26195e9ec0f  --count 1 --security-group-ids sg-084805f43ab469ff0  --instance-type t2.micro --subnet-id subnet-056347fba473db6dd  --key-name CLI-KEYPAIR --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=instance-Public-1A}]'

### 2️⃣0️⃣ LAUNCH EC2 WITH THE USE OF USER-DATA IN PUBLIC SUBNET 2
```sh
aws ec2 run-instances --image-id <your-ami-id> --count 1 --security-group-ids <your-security-group-id> --instance-type t2.micro --subnet-id <your-public-subnet2-id> --key-name <your-keypair-name> --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=<your-name-tag>}]' --user-data <your-user-data>
```
* Example:
aws ec2 run-instances --image-id ami-0d682f26195e9ec0f   --count 1   --security-group-ids sg-084805f43ab469ff0   --instance-type t2.micro   --subnet-id subnet-06a5baab94e3d42b4   --key-name CLI-KEYPAIR   --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=instance-Public-1B}]'   --user-data '#!/bin/bash
  sudo yum update -y
  sudo yum install -y httpd php git
  sudo systemctl start httpd.service
  sudo systemctl enable httpd.service
  cd /tmp
  git clone https://github.com/XxxxYyyy/aws-cli-site.git
  sudo cp -r aws-elb-site/* /var/www/html/
  sudo chown -R apache:apache /var/www/html/*
  sudo systemctl restart httpd.service'
  
### 2️⃣1️⃣ LAUNCH EC2 WITHOUT THE USE OF USER-DATA IN PRIVATE SUBNET 1

```sh
aws ec2 run-instances --image-id <your-ami-id> --count 1 --security-group-ids <your-security-group-id> --instance-type t3.micro --subnet-id <your-private-subnet-id> --key-name <your-keypair-name> --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=<your-name-tag>}]'
```
* Example:
aws ec2 run-instances --image-id ami-0d682f26195e9ec0f  --count 1 --security-group-ids sg-084805f43ab469ff0  --instance-type t3.micro --subnet-id subnet-0400769defe0dd20b --key-name CLI-KEYPAIR --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=instance-Private-1C}]'

### 2️⃣2️⃣ GET STATUS OF EC2 INSTANCE IN PUBLIC SUBNET1
```sh
aws ec2 describe-instance-status --instance-id <your-instance-id>
```
* Example:
aws ec2 describe-instance-status --instance-id i-0f628c52ceabe0b83

### 2️⃣3️⃣ GET PUBLIC IP ADDRESS OF EC2 INSTANCE IN PUBLIC SUBNET1
```sh
aws ec2 describe-instances --filters Name=instance-id,Values=<your-publicsubnet-instance-id> --query 'Reservations[*].Instances[*].[PublicIpAddress]' --output table
```
* Example:
aws ec2 describe-instances --filters "Name=instance-id,Values=i-0f628c52ceabe0b83" --query 'Reservations[*].Instances[*].[PublicIpAddress]' --output table

### 2️⃣4️⃣ GET PUBLIC IP ADDRESS OF EC2 INSTANCE IN PRIVATE SUBNET 
```sh
aws ec2 describe-instances --filters Name=instance-id,Values=<your-private-instance-id> --query 'Reservations[*].Instances[*].[PublicIpAddress]' --output table
```
* Example:
aws ec2 describe-instances --filters "Name=instance-id,Values=i-0640ada575e77ea5c" --query 'Reservations[*].Instances[*].[PublicIpAddress]' --output table

**NOTE** (NONE WILL BE DISPLAYED AS RESPONSE SINCE IT IS A PRIVATE SUBNET )

### 2️⃣5️⃣ VIEWING THE INSTANCE IN PUBLIC SUBNET1
```sh
aws ec2 describe-instances --instance-id <your-instance-id>
```
* Example:
aws ec2 describe-instances --instance-id i-0f628c52ceabe0b83


## ✅ **Verification Steps**
### 1️⃣ List All Resources  
```sh
aws ec2 describe-vpcs
aws ec2 describe-subnets
aws ec2 describe-route-tables
aws ec2 describe-internet-gateways
aws ec2 describe-security-groups
aws ec2 describe-instances
```

### 2️⃣ SSH TO THE INSTANCE
```sh
chmod 400 your-key-pair-name.pem
```
* Example:
chmod 400 CLI-KEYPAIR.pem

```sh
ssh -i your-key-pair-name.pem ec2-user@your-instance-id
```
* Example:
ssh -i CLI-KEYPAIR.pem ec2-user@15.207.117.251

## 🎯 **Conclusion**  
You have successfully set up a **VPC with Public and Private subnets, route tables, an Internet Gateway, security groups, key pairs, and EC2 instances** using **AWS CLI**.

