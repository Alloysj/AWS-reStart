# AWS S3 Storage Management Challenge Guide

## Step 1: Download and Unzip the Sample Files
First, download and extract the sample files to work with.

Run this command in your terminal to download the zip file:
```bash
wget https://aws-tc-largeobjects.s3.us-west-2.amazonaws.com/CUR-TF-100-RSJAWS-1-23732/183-lab-JAWS-managing-storage/s3/files.zip
```
Unzip the downloaded file:
```bash
unzip files.zip
```
This will create a folder (likely named `files` or similar). Check the contents with:
```bash
ls -l files/
```

## Step 2: Activate Versioning for Your S3 Bucket
Enable versioning on your S3 bucket using the AWS CLI:

```bash
aws s3api put-bucket-versioning --bucket my-s3-bucket --versioning-configuration Status=Enabled
```
Verify that versioning is enabled:
```bash
aws s3api get-bucket-versioning --bucket my-s3-bucket
```
Expected output:
```json
{
    "Status": "Enabled"
}
```

## Step 3: Sync the Local Folder with Your S3 Bucket
Use the `aws s3 sync` command to upload the contents of the `files` folder to the S3 bucket:
```bash
aws s3 sync files/ s3://my-s3-bucket/
```
Verify the files are uploaded:
```bash
aws s3 ls s3://my-s3-bucket/
```

## Step 4: Delete a Local File and Sync with Deletion
Delete a file locally from the `files` folder:
```bash
rm files/sample1.txt
```
Sync again with the `--delete` option to remove the deleted file from S3:
```bash
aws s3 sync files/ s3://my-s3-bucket/ --delete
```
Verify the file is deleted:
```bash
aws s3 ls s3://my-s3-bucket/
```

## Step 5: Recover the Deleted File Using Versioning
Since versioning is enabled, the deleted file is still recoverable.

List all object versions in the bucket:
```bash
aws s3api list-object-versions --bucket my-s3-bucket
```
Look for the deleted file (`sample1.txt`) in the output:
```json
{
    "DeleteMarkers": [
        {
            "Key": "sample1.txt",
            "VersionId": "some-delete-version-id",
            "IsLatest": true
        }
    ],
    "Versions": [
        {
            "Key": "sample1.txt",
            "VersionId": "some-previous-version-id",
            "IsLatest": false
        }
    ]
}
```
The **DeleteMarkers** entry indicates the deletion event.  
The **Versions** entry contains the previous version of the file.

Restore the previous version by running:
```bash
aws s3api get-object --bucket my-s3-bucket --key sample1.txt --version-id some-previous-version-id files/sample1.txt
```
Sync the restored file back to S3:
```bash
aws s3 sync files/ s3://my-s3-bucket/
```
Verify the file is back in S3:
```bash
aws s3 ls s3://my-s3-bucket/
```

## Final Notes
- Replace `my-s3-bucket` and file names (`sample1.txt`) with actual values.
- If you encounter errors, ensure your AWS CLI is configured with the correct permissions (`s3:PutObject`, `s3:GetObject`, `s3:ListBucket`, etc.).
- The `--delete` option in **Step 4** ensures S3 mirrors the local folder, removing files that no longer exist locally.

---
```
