# 🚀 AWS CLI: VPC & EC2 Instance Setup

This guide walks you through creating a **VPC**, **subnets**, **route tables**, **Internet Gateway (IGW)**, **security groups**, **key pairs**, and **EC2 instances** using **AWS CLI** with an IAM user.
Alright, let's break down how to build a virtual network in Amazon Web Services (AWS) from scratch, using just the command line. Think of it as constructing your own private section of the internet, tailored to your needs. We'll be using the AWS Command Line Interface (CLI), which lets you talk directly to AWS services.

Now, if you're brand new to AWS, don't worry! We'll take it step-by-step. We're going to create the following:

A Virtual Private Cloud (VPC): This is your isolated network space within AWS.
Subnets: These are like smaller, segmented networks within your VPC. We'll have both public and private subnets.
An Internet Gateway: This is how your public subnets connect to the outside world—the internet.
Route Tables: These direct network traffic, telling it where to go.
Key Pairs: These are like digital keys, allowing you to securely access your virtual servers.
Security Groups: These are like virtual firewalls, controlling what traffic can enter and leave your servers.
EC2 Instances: These are your virtual servers, the workhorses of your AWS infrastructure.
To get started, you'll need the AWS CLI installed on your computer. It's like having a translator that lets you speak AWS's language. If you haven't set that up yet, AWS has excellent documentation online to help you. It's essential to have this working before proceeding.

As we go through the commands, you'll notice that AWS assigns unique identifiers to each resource you create. These IDs are crucial for linking everything together. I strongly suggest keeping a notepad or text file handy. You'll want to jot down these IDs as you go. Here are the things you should keep track of:

The VPC's unique ID.
The IDs for your public and private subnets.
The Internet Gateway's ID.
The IDs for your public and private route tables.
The name and ID of your Security Group.
The AMI ID you will be using.
The Key pair name.
We'll be working with a lot of commands, and it can seem overwhelming at first. But don't worry, I've put together a handy cheat sheet at the end with all the commands in one place. That way, you can easily refer back to them.

Essentially, by following these instructions, you'll be building the fundamental networking blocks that allow you to deploy and manage your applications on AWS. Think of this as the foundation upon which you can build anything.

---

## 📌 Prerequisites

### 1️⃣ Install AWS CLI  
Download and install AWS CLI based on your OS:  
- **Windows**: [AWS CLI MSI Installer](https://awscli.amazonaws.com/AWSCLIV2.msi)  
- **Linux** (Ubuntu/Debian):  
  curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
  unzip awscliv2.zip
  sudo ./aws/install
 
- **MacOS**:  
  brew install awscli
  
Verify installation:  
 aws --version


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
First things first, we need to create the VPC itself. We're building a Virtual Private Cloud (VPC) right from the command line. Think of it as setting up your own little corner of the AWS cloud. Here's the command you'll use:

aws ec2 create-vpc --cidr-block 172.16.0.0/16 --tag-specifications 'ResourceType=vpc,Tags=[{Key=Name,Value=YourVPCName}]'
Example: aws ec2 create-vpc --cidr-block 172.16.0.0/16 --tag-specifications 'ResourceType=vpc,Tags=[{Key=Name,Value=CLI-VPC}]'

**Note** the `VpcId` from the response.

---

### 4️⃣ Create Subnets  
#### ✅ Public Subnet 1  

aws ec2 create-subnet --vpc-id <VpcId> --cidr-block 10.0.1.0/24 --availability-zone us-east-1a
```
#### ✅ Public Subnet 2  
```sh
aws ec2 create-subnet --vpc-id <VpcId> --cidr-block 10.0.2.0/24 --availability-zone us-east-1b
```
#### ✅ Private Subnet  
```sh
aws ec2 create-subnet --vpc-id <VpcId> --cidr-block 10.0.3.0/24 --availability-zone us-east-1c
```

---

### 5️⃣ Create Route Tables  
#### ✅ Public Route Table  
```sh
aws ec2 create-route-table --vpc-id <VpcId>
```
Save the `RouteTableId`.

#### ✅ Private Route Table  
```sh
aws ec2 create-route-table --vpc-id <VpcId>
```
Save the `RouteTableId`.

---

### 6️⃣ Attach Subnets to Route Tables  
#### ✅ Associate Public Subnet 1 & 2 with Public Route Table  
```sh
aws ec2 associate-route-table --subnet-id <PublicSubnet1Id> --route-table-id <PublicRouteTableId>
aws ec2 associate-route-table --subnet-id <PublicSubnet2Id> --route-table-id <PublicRouteTableId>
```
#### ✅ Associate Private Subnet with Private Route Table  
```sh
aws ec2 associate-route-table --subnet-id <PrivateSubnetId> --route-table-id <PrivateRouteTableId>
```

---

### 7️⃣ Attach Internet Gateway (IGW)  
#### ✅ Create and Attach IGW to VPC  
```sh
aws ec2 create-internet-gateway
aws ec2 attach-internet-gateway --internet-gateway-id <IGWId> --vpc-id <VpcId>
```
#### ✅ Add a Route in Public Route Table for IGW  
```sh
aws ec2 create-route --route-table-id <PublicRouteTableId> --destination-cidr-block 0.0.0.0/0 --gateway-id <IGWId>
```

---

### 8️⃣ Create Security Group  
```sh
aws ec2 create-security-group --group-name MySecurityGroup --description "Allow SSH and HTTP" --vpc-id <VpcId>
```
Save `GroupId`.

#### ✅ Allow SSH & HTTP  
```sh
aws ec2 authorize-security-group-ingress --group-id <GroupId> --protocol tcp --port 22 --cidr 0.0.0.0/0
aws ec2 authorize-security-group-ingress --group-id <GroupId> --protocol tcp --port 80 --cidr 0.0.0.0/0
```

---

### 9️⃣ Create a Key Pair  
```sh
aws ec2 create-key-pair --key-name MyKeyPair --query 'KeyMaterial' --output text > MyKeyPair.pem
```
Change permissions:  
```sh
chmod 400 MyKeyPair.pem
```

---

### 🔠 Launch EC2 Instances  
#### ✅ Public Instance  
```sh
aws ec2 run-instances --image-id ami-12345678 --count 1 --instance-type t2.micro --key-name MyKeyPair --security-group-ids <GroupId> --subnet-id <PublicSubnet1Id> --associate-public-ip-address
```
#### ✅ Private Instance  
```sh
aws ec2 run-instances --image-id ami-12345678 --count 1 --instance-type t2.micro --key-name MyKeyPair --security-group-ids <GroupId> --subnet-id <PrivateSubnetId>
```

---

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

### 2️⃣ SSH into Public Instance  
```sh
ssh -i MyKeyPair.pem ec2-user@<PublicInstanceIP>
```

---

## 🎯 **Conclusion**  
Congratulations! 🎉 You have successfully set up a **VPC with subnets, route tables, an Internet Gateway, security groups, key pairs, and EC2 instances** using **AWS CLI**.

