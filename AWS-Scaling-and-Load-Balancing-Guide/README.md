# AWS Scaling and Load Balancing Manual

This manual serves as a comprehensive guide for deploying a scalable and highly available architecture on AWS using **Elastic Load Balancing (ELB)** and **Amazon EC2 Auto Scaling**.

## Overview  
Elastic Load Balancing (ELB) and Auto Scaling work together to distribute incoming traffic across multiple EC2 instances and scale resources automatically based on demand. This ensures high availability and fault tolerance for your applications.

### Key Components  
- **Elastic Load Balancer (ELB):** Distributes traffic across multiple EC2 instances in multiple Availability Zones.
- **Auto Scaling Group:** Automatically adjusts the number of EC2 instances to handle the load.
- **Amazon Machine Image (AMI):** A snapshot of an EC2 instance used to launch new instances with the same configuration.
- **Amazon CloudWatch:** Monitors your infrastructure and triggers scaling actions based on metrics.

---

## Step-by-Step Guide

### Step 1: Creating an AMI
Creating an AMI ensures that new instances launched by the Auto Scaling group have the same configuration.

1. Open the **Amazon EC2 Console**.
2. Select an existing EC2 instance (`Web Server 1`).
3. Click **Actions > Image > Create Image**.
4. Configure the image settings and create the AMI.

---

### Step 2: Creating a Load Balancer
A load balancer helps distribute traffic evenly across multiple instances.

1. Open the **Amazon EC2 Console** and go to **Load Balancers**.
2. Click **Create Load Balancer** and select **Application Load Balancer**.
3. Configure the following:
   - **Network mapping:** Select the desired Availability Zones.
   - **Security Groups:** Attach appropriate security groups.
   - **Target Group:** Create a target group for your EC2 instances.
4. Review and create the load balancer.

---

### Step 3: Creating a Launch Template
A launch template specifies the configuration for instances launched by the Auto Scaling group.

1. In the **EC2 Console**, go to **Launch Templates** and click **Create Launch Template**.
2. Provide a name and description.
3. Configure the following:
   - **AMI:** Select the AMI created in **Step 1**.
   - **Instance type:** Choose the appropriate instance type.
   - **Security group:** Use the same group configured for the load balancer.
4. Save the launch template.

---

### Step 4: Creating an Auto Scaling Group
An Auto Scaling group ensures that the right number of instances are running to handle traffic.

1. In the **EC2 Console**, go to **Auto Scaling Groups** and create a new group.
2. Use the launch template created in **Step 3**.
3. Configure the network and load balancer settings:
   - Attach the load balancer created in **Step 2**.
   - Specify the **Availability Zones**.
4. Set scaling policies based on CPU utilization.
5. Review and create the Auto Scaling group.

---

### Step 5: Verifying Load Balancing
Ensure that the load balancer is functioning correctly.

1. Open the **EC2 Console** and go to **Instances**.
2. Check the health status of instances in the Auto Scaling group.
3. Verify that traffic is distributed evenly.
4. Use **Amazon CloudWatch** to monitor performance.

---

### Step 6: Testing Auto Scaling
Simulate high traffic to test your scaling policies.

1. Open the **CloudWatch Console** and create an alarm for CPU utilization.
2. Simulate increased load on your instances.
3. Verify that the Auto Scaling group launches new instances when the threshold is reached.
4. Check the CloudWatch alarm for activity.

---

### Step 7: Terminating Unneeded Instances
To reduce costs and clean up resources:

1. In the **EC2 Console**, select the instances you want to terminate.
2. Choose **Actions > Instance State > Terminate Instance**.

---

## Optional Challenge: Creating an AMI using AWS CLI
> **Note:** The solution for this challenge is hidden. Expand it only if you want to see the steps.

---

## Conclusion
By following this guide, you have:
- Created an AMI.
- Configured a load balancer.
- Set up a launch template and an Auto Scaling group.
- Verified and tested load balancing and auto-scaling.
- Used CloudWatch to monitor and scale your infrastructure.

This manual equips you with the knowledge to deploy a robust and scalable architecture on AWS.

