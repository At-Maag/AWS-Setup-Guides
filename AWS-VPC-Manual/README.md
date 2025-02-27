# AWS VPC Lab - Challenge Lab Manual

## **Overview**
This guide documents the process of configuring an AWS Virtual Private Cloud (VPC) with both public and private subnets, setting up routing, and using a **bastion server** to access a private instance. Additionally, this manual explains **IP addresses**, why they are important, and how they function within this setup.

## **Why Do We Use IP Addresses?**
An **IP (Internet Protocol) address** is a unique identifier assigned to devices within a network, allowing them to communicate with each other. There are two types of IP addresses used in this lab:

- **Public IP Address:** Assigned to internet-facing resources (e.g., the bastion server) so they can be accessed over the internet.
- **Private IP Address:** Used within the private network for internal communication (e.g., private EC2 instances that should not be directly accessible from the internet).

### **IP Addressing in This Lab**
- The **public subnet** uses a **public IP** for the **bastion server**, allowing SSH access from the internet.
- The **private subnet** uses **private IPs**, making instances only accessible within the VPC.
- The **NAT Gateway** (in the public subnet) enables instances in the private subnet to access the internet **without exposing them to incoming traffic**.

## **Lab Tasks**
### **Task 1: Creating a VPC**
1. Navigate to the **AWS VPC Management Console**.
2. Choose **Your VPCs** > **Create VPC**.
3. Configure the VPC:
   - **Name tag:** `Lab VPC`
   - **IPv4 CIDR Block:** `10.0.0.0/16`
   - **Enable DNS Hostnames**: Yes
4. Click **Create VPC**.

### **Task 2: Creating Subnets**
#### **Creating the Public Subnet**
1. In the **Subnets** section, click **Create subnet**.
2. Configure:
   - **VPC ID:** `Lab VPC`
   - **Subnet Name:** `Public Subnet`
   - **IPv4 CIDR Block:** `10.0.0.0/24`
3. Click **Create subnet**.
4. Enable auto-assignment of public IP addresses:
   - Click **Actions** > **Edit subnet settings**.
   - Enable **auto-assign public IPv4**.
   - Click **Save**.

#### **Creating the Private Subnet**
1. Repeat the same steps but configure:
   - **Subnet Name:** `Private Subnet`
   - **IPv4 CIDR Block:** `10.0.2.0/23`
2. Click **Create subnet**.

### **Task 3: Creating an Internet Gateway**
1. Go to **Internet Gateways**.
2. Click **Create Internet Gateway**.
3. Name it `Lab IGW` and attach it to `Lab VPC`.

### **Task 4: Configuring Route Tables**
1. Go to **Route Tables**.
2. Rename the default route table as `Private Route Table`.
3. Create a new route table and name it `Public Route Table`.
4. Configure **routes for the Public Route Table**:
   - **Destination:** `0.0.0.0/0`
   - **Target:** `Internet Gateway (Lab IGW)`
5. Click **Save changes**.
6. Associate the **Public Subnet** with the **Public Route Table**.
7. Verify the **Private Route Table** only has `10.0.0.0/16` (local traffic).

### **Task 5: Launching a Bastion Server**
1. Go to **EC2 > Instances** > **Launch Instance**.
2. Configure:
   - **Name:** `Bastion Server`
   - **AMI:** Amazon Linux 2023
   - **Instance Type:** `t3.micro`
   - **Subnet:** `Public Subnet`
   - **Auto-assign Public IP:** Enabled
   - **Security Group:** Allow SSH from anywhere
3. Click **Launch**.

### **Task 6: Creating a NAT Gateway**
1. Go to **NAT Gateways**.
2. Click **Create NAT Gateway**.
3. Configure:
   - **Name:** `Lab NAT Gateway`
   - **Subnet:** `Public Subnet`
   - **Elastic IP:** Allocate a new Elastic IP
4. Click **Create NAT Gateway**.
5. Modify the **Private Route Table**:
   - **Destination:** `0.0.0.0/0`
   - **Target:** `NAT Gateway`
6. Click **Save changes**.

### **Task 7: Testing the Private Subnet**
1. Launch a new **EC2 instance** in the **Private Subnet**.
2. SSH into the **Bastion Server** (using its **public IP**):
   ```bash
   ssh ec2-user@<BASTION-PUBLIC-IP>
   ```
3. From the bastion server, SSH into the **Private Instance** (using its **private IP**):
   ```bash
   ssh ec2-user@<PRIVATE-IP>
   ```
4. Confirm internet access by running:
   ```bash
   ping -c 3 amazon.com
   ```

### **Final Notes**
- **Public IPs** allow instances to communicate directly with the internet.
- **Private IPs** ensure internal communication but require a NAT gateway for outgoing internet access.
- The **bastion server** acts as a **secure bridge** to the private instance.

This manual provides a structured approach to setting up a VPC with proper networking rules. Now, you can push this to GitHub for documentation! 🚀

