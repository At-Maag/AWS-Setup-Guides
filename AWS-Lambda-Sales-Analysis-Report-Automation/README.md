# AWS Lambda: Sales Analysis Report Automation

## Overview
This document provides step-by-step instructions for deploying an **AWS Lambda function** that automates a **sales analysis report** by pulling data from an **EC2-hosted MySQL database** and emailing the results via **Amazon SNS**. It also covers troubleshooting **inbound rules** and configuring **cron expressions** for scheduled execution.

---

## 1. Architecture Overview
### Components:
- **AWS Lambda Functions**:
  - `salesAnalysisReport`: Calls another Lambda function and sends reports.
  - `salesAnalysisReportDataExtractor`: Extracts data from a MySQL database.
- **Amazon EC2**: Hosts the **MySQL database** (LAMP stack).
- **Amazon SNS**: Sends the report via email.
- **Amazon CloudWatch**: Schedules the function using a cron expression.

---

## 2. Prerequisites
Ensure you have:
- **AWS account with IAM permissions**
- **An EC2 instance running MySQL**
- **AWS CLI installed and configured (`aws configure`)**

---

## 3. Setting Up IAM Roles
### 3.1. Creating IAM Role for Lambda
1. Navigate to **AWS IAM Console** → **Roles** → **Create role**.
2. Select **AWS Service** → Choose **Lambda**.
3. Attach policies:
   - `AmazonSNSFullAccess`
   - `AmazonSSMReadOnlyAccess`
   - `AWSLambdaBasicExecutionRole`
   - `AWSLambdaRole`
4. Name it **salesAnalysisReportRole** and save.

### 3.2. Creating IAM Role for Data Extraction Lambda
1. Repeat the above steps but attach:
   - `AWSLambdaBasicExecutionRole`
   - `AWSLambdaVPCAccessExecutionRole`
2. Name it **salesAnalysisReportDERole** and save.

---

## 4. Creating the Lambda Functions
### 4.1. Creating a Lambda Layer
1. Run:
   ```bash
   pip install pymysql -t python/
   zip -r pymysql.zip python
   ```
2. Go to **AWS Lambda Console** → **Layers** → **Create Layer**.
3. Upload `pymysql.zip`, set runtime to **Python 3.9**, and save as **pymysqlLibrary**.

### 4.2. Creating `salesAnalysisReportDataExtractor`
1. Navigate to **AWS Lambda Console** → **Create Function**.
2. Select **Python 3.9** and use **salesAnalysisReportDERole**.
3. Add **pymysqlLibrary** as a layer.
4. Upload the function code (`salesAnalysisReportDataExtractor.py`).
5. Set **Environment Variables**:
   ```
   dbUrl = <your RDS endpoint>
   dbName = cafe_db
   dbUser = root
   dbPassword = <your password>
   ```
6. Click **Save**.

### 4.3. Creating `salesAnalysisReport`
1. Repeat steps **1-4**, but name it **salesAnalysisReport**.
2. Use **salesAnalysisReportRole**.
3. Set **Environment Variables**:
   ```
   topicARN = <your SNS topic ARN>
   ```
4. Click **Save**.

---

## 5. Configuring Database Inbound Rules
### 5.1. Fixing Lambda Timeout Issue (Caused by Inbound Rules)
1. Go to **AWS EC2 Console** → **Security Groups**.
2. Locate the **EC2 instance's security group**.
3. Edit **Inbound Rules**:
   - Add a rule:
     - **Type**: MySQL/Aurora
     - **Protocol**: TCP
     - **Port Range**: `3306`
     - **Source**: **Lambda Security Group**
4. Click **Save Rules**.

---

## 6. Testing the Lambda Functions
### 6.1. Testing `salesAnalysisReportDataExtractor`
1. Create a test event (`SARExtractorTest`):
   ```json
   {
     "dbUrl": "<your-rds-endpoint>",
     "dbName": "cafe_db",
     "dbUser": "root",
     "dbPassword": "<your-password>"
   }
   ```
2. Click **Test**.

### 6.2. Testing `salesAnalysisReport`
1. Create a test event (`SARTestEvent`):
   ```json
   {
     "topicARN": "<your-sns-topic-arn>"
   }
   ```
2. Click **Test**.

If successful, check your **email inbox**.

---

## 7. Scheduling Execution with Cron Expressions
### AWS Cron Expression Format
```
cron(Minutes Hours Day-of-month Month Day-of-week Year)
```
- **Minutes**: `0-59`
- **Hours**: `0-23` (UTC)
- **Day-of-month**: `1-31`
- **Month**: `1-12`
- **Day-of-week**: `SUN-SAT` or `?`
- **Year**: `2024, 2025, * (any)`

### Example Cron Expressions
| Time Zone  | Run Time  | UTC Equivalent | Cron Expression |
|------------|----------|---------------|----------------|
| **PST** (UTC-8) | 8:00 PM | 4:00 AM UTC | `cron(0 4 ? * MON-SAT *)` |
| **PDT** (UTC-7) | 8:00 PM | 3:00 AM UTC | `cron(0 3 ? * MON-SAT *)` |
| **EST** (UTC-5) | 8:00 PM | 1:00 AM UTC | `cron(0 1 ? * MON-SAT *)` |

### Setting Up a CloudWatch Schedule
1. Go to **AWS EventBridge (CloudWatch Events)**.
2. Click **Create Rule** → Name it `salesAnalysisReportTrigger`.
3. Choose **Schedule expression** → Enter:
   ```cron
   cron(0 4 ? * MON-SAT *)
   ```
   *(for 8 PM PST)*
4. Set target as **salesAnalysisReport Lambda**.
5. Click **Create Rule**.

---

## 8. Conclusion
This guide covered:
✅ Deploying **AWS Lambda**
✅ Configuring **Security Groups and inbound rules**
✅ Testing using **AWS Lambda Console**
✅ Scheduling execution using **AWS Cron Expressions**

🚀 **Push this to GitHub for future reference!** 🚀

