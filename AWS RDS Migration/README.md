# AWS RDS Migration Lab - Step-by-Step Guide

This document provides a detailed step-by-step guide for migrating a MySQL/MariaDB database from an EC2 instance to Amazon RDS. It also includes troubleshooting steps for common issues encountered during the process.

---

## **1. Lab Overview**

In this lab, we migrated a local database running on an EC2 instance to a fully managed Amazon RDS instance. The process included:

- Generating data in the local database.
- Creating an Amazon RDS MariaDB instance.
- Migrating data from the EC2 database to RDS.
- Configuring the application to use RDS instead of the local database.
- Monitoring the RDS instance using AWS CloudWatch.

---

## **2. Creating an Amazon RDS Instance**

### **2.1 Prerequisite Components**

We created the following AWS resources:

- **Security Group for RDS** (`CafeDatabaseSG`)
- **Private Subnets for RDS** (`CafeDB Private Subnet 1` & `CafeDB Private Subnet 2`)
- **Database Subnet Group** (`CafeDBSubnetGroup`)

### **2.2 Creating the Security Group**

```sh
aws ec2 create-security-group \
--group-name CafeDatabaseSG \
--description "Security group for cafe database" \
--vpc-id <CafeInstance VPC ID>
```

### **2.3 Configuring Inbound Rules**

Allow only EC2 instances to connect to the RDS instance:

```sh
aws ec2 authorize-security-group-ingress \
--group-id <CafeDatabaseSG Group ID> \
--protocol tcp --port 3306 \
--source-group <CafeSecurityGroup ID>
```

### **2.4 Creating Subnets**

```sh
aws ec2 create-subnet \
--vpc-id <CafeInstance VPC ID> \
--cidr-block 10.200.2.0/23 \
--availability-zone us-west-2a
```

```sh
aws ec2 create-subnet \
--vpc-id <CafeInstance VPC ID> \
--cidr-block 10.200.10.0/23 \
--availability-zone us-west-2b
```

### **2.5 Creating the DB Subnet Group**

```sh
aws rds create-db-subnet-group \
--db-subnet-group-name "CafeDBSubnetGroup" \
--db-subnet-group-description "DB subnet group for cafe" \
--subnet-ids <CafeDB Private Subnet 1 ID> <CafeDB Private Subnet 2 ID> \
--tags "Key=Name,Value=CafeDatabaseSubnetGroup"
```

### **2.6 Creating the RDS Instance**

```sh
aws rds create-db-instance \
--db-instance-identifier CafeDBInstance \
--engine mariadb \
--engine-version 10.5.20 \
--db-instance-class db.t3.micro \
--allocated-storage 20 \
--availability-zone us-west-2a \
--db-subnet-group-name "CafeDBSubnetGroup" \
--vpc-security-group-ids <CafeDatabaseSG Group ID> \
--no-publicly-accessible \
--master-username root --master-user-password 'Re:Start!9'
```

---

## **3. Migrating Data to RDS**

### **3.1 Backup the Local Database**

```sh
mysqldump --host=localhost \
--user=root --password='Re:Start!9' \
--databases cafe_db --add-drop-database > cafedb-backup.sql
```

### **3.2 Restore the Backup to RDS**

```sh
mysql --host=<RDS Instance Endpoint> \
--user=root --password='Re:Start!9' < cafedb-backup.sql
```

---

## **4. Configuring the Application**

To switch the application to use the RDS database, update the `Parameter Store` in **AWS Systems Manager**:

1. Open **AWS Systems Manager**.
2. Navigate to **Parameter Store**.
3. Find `/cafe/dbUrl` and update it with the **RDS Instance Endpoint**.

---

## **5. Troubleshooting Issues**

### **Issue 1: AWS CLI - "Unknown options: tcp, protocol"**

**Fix:** Ensure `--protocol tcp` comes **before** `--port` in the command.

### **Issue 2: EC2 Instance Cannot Connect to RDS**

**Fix:** Ensure the RDS security group allows inbound **MySQL (3306)** traffic **from the EC2 security group.**

```sh
aws ec2 authorize-security-group-ingress \
--group-id <RDS Security Group ID> \
--protocol tcp --port 3306 \
--source-group <EC2 Security Group ID>
```

### **Issue 3: RDS Security Group is Missing Rules**

**Fix:** Verify using:

```sh
aws ec2 describe-security-groups --group-ids <RDS Security Group ID> --query "SecurityGroups[*].IpPermissions"
```

### **Issue 4: ************************************`mysqldump`************************************ Command Not Found**

**Fix:** Install MySQL client:

```sh
sudo yum install -y mysql
```

### **Issue 5: Cannot Connect to RDS (Timeout Error 110)**

**Fix:** Ensure RDS **security group allows connections** from EC2.

### **Issue 6: Database Does Not Exist (********`Unknown database 'cafe_db'`********\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*)**

**Fix:** Log into MySQL and manually create the database:

```sql
CREATE DATABASE cafe_db;
```

### **Issue 7: RDS Instance Not Accessible**

**Fix:** Ensure the instance is in the correct **availability zone** and **subnet group**.

```sh
aws rds describe-db-instances --db-instance-identifier CafeDBInstance
```

---

## **6. Monitoring RDS Performance**

### **6.1 Check RDS Metrics in AWS CloudWatch**

1. Open the **AWS Management Console**.
2. Navigate to **RDS > Databases > CafeDBInstance**.
3. Click **Monitoring** to view:
   - **CPU Utilization**
   - **Database Connections**
   - **Read/Write IOPS**
   - **Freeable Memory**

---

## **Conclusion**

This guide provides a structured approach to migrating a MySQL/MariaDB database from an EC2 instance to Amazon RDS. It includes:

- Creating RDS infrastructure using AWS CLI.
- Configuring security groups and subnets.
- Troubleshooting common issues.
- Monitoring RDS performance in AWS CloudWatch.

**Author :** At Maag **Date: 2025/02/26**

