# AWS Lambda Word Count Function

## Overview
This project demonstrates how to create an AWS Lambda function that automatically counts the number of words in a text file uploaded to an Amazon S3 bucket. The function sends the word count result via Amazon Simple Notification Service (SNS) to an email recipient.

## Features
- Uses **AWS Lambda** to process text files.
- Automatically triggers when a file is uploaded to **Amazon S3**.
- Sends word count results via **Amazon SNS**.
- Fully serverless and event-driven architecture.

---

## Step-by-Step Instructions

### **1. Launch Your AWS Lab Environment**
- Open AWS Console.
- Navigate to **S3**, **Lambda**, **SNS**, and **IAM**.

### **2. Create an S3 Bucket**
- Navigate to **Amazon S3**.
- Click **Create bucket**.
- Enter a **unique bucket name** (e.g., `lambda-wordcount-at12345`).
- Select a region and leave other settings as default.
- Click **Create bucket**.

### **3. Create an SNS Topic**
- Navigate to **Amazon SNS**.
- Click **Create topic** → **Standard**.
- Set a name (e.g., `WordCountTopic`).
- Click **Create topic**.
- Copy the **Topic ARN**.

### **4. Create an SNS Subscription**
- Navigate to **Subscriptions**.
- Click **Create subscription**.
- Choose **Protocol: Email**.
- Enter your email and select the SNS topic created earlier.
- Click **Create subscription**.
- Confirm your subscription from the email sent by AWS.

### **5. Create an IAM Role for Lambda**
- Navigate to **IAM → Roles**.
- Click **Create role** → Choose **AWS Service → Lambda**.
- Attach the following policies:
  - `AWSLambdaBasicExecutionRole`
  - `AmazonS3FullAccess`
  - `AmazonSNSFullAccess`
- Name the role (e.g., `LambdaAccessRole`) and **Create role**.
- Copy the **Role ARN**.

### **6. Create the AWS Lambda Function**
- Navigate to **AWS Lambda** → **Create function**.
- Choose **Author from scratch**.
- Name: `WordCountLambda`.
- Runtime: `Python 3.9`.
- Permissions: Use existing role (`LambdaAccessRole`).
- Click **Create function**.

### **7. Add Code to the Lambda Function**
Replace the default Lambda code with the following:

```python
import boto3
import json

def lambda_handler(event, context):
    s3 = boto3.client('s3')
    sns = boto3.client('sns')

    # Extract bucket and file name from event
    bucket_name = event['Records'][0]['s3']['bucket']['name']
    file_name = event['Records'][0]['s3']['object']['key']
    
    # Retrieve file from S3
    response = s3.get_object(Bucket=bucket_name, Key=file_name)
    text_content = response['Body'].read().decode('utf-8')
    
    # Count words
    word_count = len(text_content.split())
    
    # Send SNS Notification
    message = f"The word count in the {file_name} file is {word_count}."
    sns_topic_arn = "arn:aws:sns:REGION:ACCOUNT_ID:WordCountTopic"  # Replace with actual ARN
    sns.publish(TopicArn=sns_topic_arn, Message=message, Subject="Word Count Result")
    
    return {
        'statusCode': 200,
        'body': json.dumps({'message': message})
    }
```
- Replace `sns_topic_arn` with the **SNS Topic ARN** from Step 3.
- Click **Deploy**.

### **8. Set Up an S3 Event Trigger for Lambda**
- Go to the **Lambda function**.
- Click **Add trigger**.
- Select **S3** as the source.
- Choose your S3 bucket.
- Select **All object create events**.
- Click **Add**.

### **9. Test the Function**
- Create a text file (`test1.txt`) with sample text.
- Upload it to the **S3 bucket**.
- Check **Lambda logs** in CloudWatch.
- Check your **email** for the SNS notification.

### **10. Validate and Submit**
- Upload multiple text files with different content.
- Confirm SNS notifications are received.
- Take screenshots of the Lambda function and email notification.

---

## **Conclusion**
By following this guide, you successfully:
- Created a Lambda function to count words in a text file.
- Configured S3 to trigger the function.
- Sent the word count results via SNS email notifications.

This project demonstrates **serverless computing with AWS Lambda** and how to integrate multiple AWS services. 🚀

