**Setting Up an AWS CLI-Based EC2 LAMP Stack and Troubleshooting Guide**

---

## **Overview**
This guide provides step-by-step instructions for deploying a LAMP stack (Linux, Apache, MySQL, PHP) on an EC2 instance using AWS CLI. It also includes a troubleshooting section to resolve common issues.

---

## **Step 1: Connecting to the CLI Host Instance**
1. Log in to the **AWS Management Console**.
2. Navigate to **EC2 Dashboard** and find the **CLI Host** instance.
3. Connect to the instance using **EC2 Instance Connect**.
4. Verify your connection by running basic Linux commands:
   ```bash
   uname -a
   ```

---

## **Step 2: Configuring AWS CLI**
1. Run the following command to configure AWS CLI with the provided credentials:
   ```bash
   aws configure
   ```
2. Enter the following information when prompted:
   - **AWS Access Key ID**
   - **AWS Secret Access Key**
   - **Default region name** (e.g., `us-west-2`)
   - **Default output format:** `json`

---

## **Step 3: Running the LAMP Deployment Script**
1. Navigate to the directory containing the deployment script:
   ```bash
   cd ~/sysops-activity-files/starters
   ```
2. Create a backup of the script:
   ```bash
   cp create-lamp-instance-v2.sh create-lamp-instance.backup
   ```
3. Edit the script to ensure the correct AMI ID:
   ```bash
   vi create-lamp-instance-v2.sh
   ```
   - Update the AMI ID with a valid Amazon Linux 2 AMI for your region:
     ```bash
     imageId="ami-065212a60f30e3798"
     ```
   - Ensure the region in the `aws ec2 run-instances` command matches your configured region:
     ```bash
     --region us-west-2
     ```
4. Save the changes and run the script:
   ```bash
   ./create-lamp-instance-v2.sh
   ```

---

## **Step 4: Troubleshooting Common Issues**

### **Issue 1: Incorrect Region in `aws ec2 run-instances` Command**
**Cause:** The `aws ec2 run-instances` command was using `--region us-east-1` instead of the correct region (`us-west-2`).  
**Solution:**
1. Edit the `create-lamp-instance-v2.sh` script and ensure the correct region is specified in the `aws ec2 run-instances` command:
   ```bash
   --region us-west-2
   ```
2. Save and re-run the script.

### **Issue 2: HTTPD Service Not Found**
**Cause:** The user data script did not install Apache (`httpd`).  
**Solution:**
1. Check the user data log to see if Apache was installed:
   ```bash
   sudo tail -f /var/log/cloud-init-output.log
   ```
2. If Apache is missing, install and start it manually:
   ```bash
   sudo yum update -y
   sudo yum install httpd -y
   sudo systemctl start httpd
   sudo systemctl enable httpd
   ```
3. Verify that Apache is running:
   ```bash
   sudo systemctl status httpd
   ```

### **Issue 3: Webpage Not Loading**
**Cause:** Security group rules are not configured to allow traffic on port 80.  
**Solution:**
1. Go to the **AWS Management Console** and find the security group for the LAMP instance.
2. Add an inbound rule for **HTTP (port 80)** with source `0.0.0.0/0`.
3. Save the rule and refresh the webpage.

---

## **Step 5: Verifying the Deployment**
1. Open a browser and navigate to:
   ```
   http://<public-ip>/cafe/
   ```
   Replace `<public-ip>` with your instance's public IP.
2. You should see the web application for the cafe.

---

## **Conclusion**
By following this guide, you have successfully deployed a LAMP stack on an EC2 instance using AWS CLI. If any issues arise, use the troubleshooting section to resolve them.

---

