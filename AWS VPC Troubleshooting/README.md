### **AWS VPC Troubleshooting Manual**
#### *Identifying and Resolving Connectivity Issues Using Flow Logs and AWS CLI*

---

## **Lab Overview**
In this lab, we troubleshoot connectivity issues in an AWS Virtual Private Cloud (VPC) using **AWS CLI** and **Flow Logs**. We analyze access to a web server instance and resolve blocked SSH and HTTP connections.

---

## **Objectives**
By the end of this lab, you will be able to:
- Identify connectivity issues using **AWS Flow Logs**.
- Analyze **security groups, route tables, and network ACLs**.
- Modify network rules to resolve **SSH and HTTP access problems**.
- Use AWS CLI commands to **filter, query, and troubleshoot** networking configurations.

---

## **Scenario**
A **web server instance** is running, but the webpage does not load, and SSH connections fail. Using AWS CLI, you will diagnose and resolve these issues.

---

## **Troubleshooting Challenge 1: HTTP Access Blocked**
### **Issue**
The web server instance is running, but the webpage is not accessible.

### **Investigation Steps**
1. Check the **security groups** assigned to the web server:
   ```sh
   aws ec2 describe-security-groups
   ```
2. Verify the **route table association** of the public subnet:
   ```sh
   aws ec2 describe-route-tables --filters "Name=association.subnet-id,Values=<SUBNET_ID>"
   ```
3. Confirm that the **public route table** has an entry to the **internet gateway**:
   ```sh
   aws ec2 create-route --route-table-id <ROUTE_TABLE_ID> --gateway-id <INTERNET_GATEWAY_ID> --destination-cidr-block 0.0.0.0/0
   ```

### **Solution (Click to reveal)**
<details>
  <summary>Solution</summary>
  After checking the security groups and routes, we found that the **public route table** was missing a rule directing traffic to the internet gateway. Adding a route to `0.0.0.0/0` resolved the issue, allowing HTTP traffic to reach the web server.
</details>

---

## **Troubleshooting Challenge 2: SSH Connection Failing**
### **Issue**
The SSH connection to the web server fails despite port 22 being allowed in **security groups**.

### **Investigation Steps**
1. Verify **network ACLs** assigned to the subnet:
   ```sh
   aws ec2 describe-network-acls --filter "Name=association.subnet-id,Values=<SUBNET_ID>"
   ```
2. Identify **deny rules** that could be blocking SSH (port 22):
   ```sh
   aws ec2 describe-network-acls --query "NetworkAcls[*].Entries"
   ```
3. If a **deny rule** exists for port 22, remove it:
   ```sh
   aws ec2 delete-network-acl-entry --network-acl-id <ACL_ID> --ingress --rule-number 40
   ```

### **How We Identified Rule 40 as the Issue**
- The **network ACL list** showed Rule 40 as **DENY** for TCP traffic on **port 22**.
- This rule took precedence over the **ALLOW** rule due to AWS **network ACL evaluation order**.
- After deleting Rule 40, SSH connections were successfully established.

### **Solution (Click to reveal)**
<details>
  <summary>Solution</summary>
  Deleting **Rule 40** from the **network ACL** allowed SSH traffic through. AWS evaluates **network ACLs in ascending order**; Rule 40 was processed before the **ALLOW** rule, causing the connection to be rejected.
</details>

---

## **Troubleshooting Challenge 3: Analyzing Flow Logs**
### **Issue**
We need to verify whether failed SSH connection attempts were captured in **VPC Flow Logs**.

### **Investigation Steps**
1. **List stored flow logs in the S3 bucket**:
   ```sh
   aws s3 ls s3://<FLOW_LOG_BUCKET>/ --recursive
   ```
2. **Extract log files**:
   ```sh
   gunzip *.gz
   ```
3. **Check logs for rejected SSH connections (port 22)**:
   ```sh
   grep -rn 22 . | grep REJECT | grep <YOUR_IP>
   ```

### **Solution (Click to reveal)**
<details>
  <summary>Solution</summary>
  By searching the flow logs, we confirmed that SSH connection attempts were being **REJECTED** due to the **network ACL deny rule (Rule 40)**. After removing the rule, logs showed **ACCEPTED** connections.
</details>

---

## **Key Takeaways**
- **Security groups are stateful**, meaning an inbound rule automatically allows the response traffic.
- **Network ACLs are stateless**, meaning inbound and outbound rules must be explicitly allowed.
- AWS evaluates **network ACLs in ascending order**, so a lower-numbered **DENY** rule can override a later **ALLOW** rule.
- **Flow Logs help diagnose connectivity issues** by capturing rejected or allowed traffic.

---

## **Additional Questions**
1. **Why did the network ACL block SSH even though the security group allowed it?**  
   - *Network ACLs apply to the entire subnet and override security groups.*
   
2. **Why did we add a route to the internet gateway?**  
   - *Without a route, traffic from the public subnet cannot reach the internet.*

3. **How can we confirm SSH is working after fixing Rule 40?**  
   - *Use `ssh -i <KEY> ec2-user@<INSTANCE_PUBLIC_IP>` to verify connectivity.*

---

## **Conclusion**
You have successfully completed the troubleshooting lab! 🎉  
You diagnosed and resolved **HTTP and SSH connection issues** using **AWS CLI and Flow Logs**.

---
This manual is designed for **GitHub documentation** with hidden solutions. Let me know if you'd like any formatting adjustments! 🚀

