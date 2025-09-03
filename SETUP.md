# Setup Guide – Configure a VPC with Private Subnets and a NAT Gateway

This guide provides the detailed step-by-step process to build a **VPC with public and private subnets** and enable outbound internet connectivity for private instances using a **NAT Gateway**.

---

## Prerequisites
- An **AWS Account** with sufficient permissions to create VPC, Subnets, Route Tables, IGW, NAT Gateway, and EC2.  
- **IAM user/role** with administrator or networking privileges.  
- Access to the **AWS Management Console** or **AWS CLI** configured locally.  
- Basic understanding of **VPC networking** concepts (CIDR, subnets, route tables).  
- A **key pair** created in your region to SSH into EC2 instances.  

---

## 1. Create a VPC
1. Open the **VPC Console** → *Create VPC*.  
2. Choose **VPC only** and set:  
   - Name: `MyVPC`  
   - IPv4 CIDR block: `10.0.0.0/16`  
3. Click **Create VPC**.  

---

## 2. Create Subnets
1. Go to **Subnets** → *Create subnet*.  
2. Select your VPC (`MyVPC`).  
3. Add:  
   - **Public Subnet** (e.g., `10.0.1.0/24`) in AZ-1a.  
   - **Private Subnet** (e.g., `10.0.2.0/24`) in AZ-1a.  
4. Enable **Auto-assign public IPv4** for the public subnet only.  

---

## 3. Attach an Internet Gateway (IGW)
1. In **Internet Gateways**, click *Create internet gateway*.  
   - Name: `MyIGW`.  
2. Attach it to your VPC (`MyVPC`).  

---

## 4. Configure Route Tables
1. Go to **Route Tables** → *Create route table*.  
   - Name: `Public-RT`.  
   - Associate with `MyVPC`.  
2. Add a route:  
   - Destination: `0.0.0.0/0` → Target: **IGW**.  
   - Associate `Public-RT` with the **public subnet**.  
3. Create another route table: `Private-RT`.  
   - Associate with `MyVPC`.  
   - Associate `Private-RT` with the **private subnet**.  

---

## 5. Launch EC2 Instances
1. Create an **EC2 instance** in the **Public Subnet** (Bastion host).  
   - Assign public IP.  
   - Security Group: allow SSH from your IP.  
2. Create an **EC2 instance** in the **Private Subnet** (Application server).  
   - No public IP.  
   - Security Group: allow SSH only from the Bastion host’s Security Group.  

> **Important:** To SSH from the Public instance (Bastion) into the Private instance,  
> - Download the private key (`.pem` file) to your local machine.  
> - When connecting to the Bastion host, **upload the key file to it** (e.g., using `scp`).  
> - Use that key on the Bastion to log in to the Private EC2 instance.  

---

## 6. Create a NAT Gateway
1. Go to **NAT Gateways** → *Create NAT gateway*.  
2. Select:  
   - Subnet: **Public Subnet** (the one connected to IGW; can be the same as the Bastion host).  
   - Elastic IP: Allocate new Elastic IP.  
3. Create the NAT Gateway.  

---

## 7. Update Private Route Table
1. Edit `Private-RT`.  
2. Add route:  
   - Destination: `0.0.0.0/0` → Target: **NAT Gateway**.  

---

## 8. Test Connectivity
- SSH into the **Bastion host** from your local machine.  
- From Bastion, SSH into the **Private instance** using the key file.  
- On the private instance, run:  
  ```bash
  ping google.com
  sudo yum update -y
- You should see outbound internet working via the NAT Gateway. 
