# **AWS S3 and Snapshot Management Lab Manual**

## **Lab Overview**
This lab focuses on managing AWS storage using Amazon S3 and Amazon EBS. You will create an S3 bucket, manage snapshots, set up automated backups, and handle file versioning and recovery.

## **Objectives**
- Create and configure an S3 bucket.
- Take snapshots of an EBS volume using AWS CLI.
- Automate snapshot creation using cron jobs.
- Enable S3 versioning and recover deleted files.
- Sync and manage files between an EBS volume and S3.

---

## **Task 1: Creating and Configuring Resources**
### **1.1 Create an S3 Bucket**
1. Open the AWS Management Console.
2. In the **Search bar**, type `S3` and select **S3 Management Console**.
3. Click **Create bucket**.
4. Enter a unique bucket name (e.g., `my-example-s3-bucket`).
5. Leave the region as default and click **Create bucket**.

### **1.2 Attach Instance Profile to Processor**
1. Open the **EC2 Management Console**.
2. In the **Search bar**, type `EC2` and select **Instances**.
3. Select the **Processor** instance.
4. Click **Actions** > **Security** > **Modify IAM Role**.
5. Choose the `S3BucketAccess` role and click **Update IAM role**.

---

## **Task 2: Taking Snapshots of Your Instance**
### **2.1 Connecting to the Command Host EC2 Instance**
1. Open the **EC2 Management Console**.
2. Select the **Command Host** instance and click **Connect**.
3. Use **EC2 Instance Connect** to open a terminal window.

### **2.2 Taking an Initial Snapshot**
1. Find the **EBS volume ID** attached to the Processor:
   ```bash
   aws ec2 describe-instances --filters 'Name=tag:Name,Values=Processor' --query 'Reservations[0].Instances[0].BlockDeviceMappings[0].Ebs.VolumeId'
   ```
2. Retrieve the **Instance ID**:
   ```bash
   aws ec2 describe-instances --filters 'Name=tag:Name,Values=Processor' --query 'Reservations[0].Instances[0].InstanceId'
   ```
3. Stop the **Processor** instance:
   ```bash
   aws ec2 stop-instances --instance-ids INSTANCE-ID
   ```
4. Wait until the instance is fully stopped:
   ```bash
   aws ec2 wait instance-stopped --instance-id INSTANCE-ID
   ```
5. Create a **snapshot** of the volume:
   ```bash
   aws ec2 create-snapshot --volume-id VOLUME-ID
   ```
6. Wait for snapshot completion:
   ```bash
   aws ec2 wait snapshot-completed --snapshot-id SNAPSHOT-ID
   ```
7. Restart the **Processor** instance:
   ```bash
   aws ec2 start-instances --instance-ids INSTANCE-ID
   ```

### **2.3 Scheduling the Creation of Subsequent Snapshots**
<details>
<summary>Steps</summary>

1. Create a cron job to take snapshots every minute:
   ```bash
   echo "* * * * * aws ec2 create-snapshot --volume-id VOLUME-ID 2>&1 >> /tmp/cronlog" > cronjob
   crontab cronjob
   ```
2. Verify snapshots are being created:
   ```bash
   aws ec2 describe-snapshots --filters "Name=volume-id,Values=VOLUME-ID"
   ```
</details>

<details>
<summary>Troubleshooting</summary>

- **Issue:** `boto3` module not found when running Python script.
  - **Solution:** Install `boto3`:
    ```bash
    pip install boto3
    ```
</details>

### **2.4 Retaining the Last Two Snapshots**
1. Stop the cron job:
   ```bash
   crontab -r
   ```
2. Run the snapshot management script:
   ```bash
   python3 snapshooter_v2.py
   ```

---

## **Task 3: Challenge - Synchronizing Files with Amazon S3**
<details>
<summary>Steps</summary>

1. Download and unzip sample files:
   ```bash
   wget https://aws-tc-largeobjects.s3.us-west-2.amazonaws.com/CUR-TF-100-RSJAWS-3-23732/183-lab-JAWS-managing-storage/s3/files.zip
   unzip files.zip
   ```
2. Enable S3 versioning:
   ```bash
   aws s3api put-bucket-versioning --bucket my-example-s3-bucket --versioning-configuration Status=Enabled
   ```
3. Sync files to S3:
   ```bash
   aws s3 sync files/ s3://my-example-s3-bucket/files/
   ```
4. Delete a file locally and sync the deletion:
   ```bash
   rm files/file1.txt
   aws s3 sync files/ s3://my-example-s3-bucket/files/ --delete
   ```
5. Recover the deleted file from S3:
   ```bash
   aws s3api list-object-versions --bucket my-example-s3-bucket --prefix files/file1.txt
   aws s3api get-object --bucket my-example-s3-bucket --key files/file1.txt --version-id VERSION-ID files/file1.txt
   ```
6. Re-sync the restored file to S3:
   ```bash
   aws s3 sync files/ s3://my-example-s3-bucket/files/
   ```
</details>

<details>
<summary>Troubleshooting</summary>

- **Issue:** `Invalid version id specified` when restoring a deleted file.
  - **Solution:** Ensure the correct `VersionId` is used and is enclosed in double quotes:
    ```bash
    aws s3api get-object --bucket my-example-s3-bucket --key files/file1.txt --version-id "VERSION-ID" files/file1.txt
    ```
</details>

---

## **Conclusion**
- Created and configured an **S3 bucket**.
- Managed **EBS snapshots** with AWS CLI.
- Automated snapshots with **cron jobs**.
- Used **S3 versioning** to recover deleted files.

---

