# AWS CloudFormation Serverless ETL Deployment Guide

This guide documents the complete deployment of the **Day 50 - Infrastructure as Code (IaC) Serverless ETL Project** using both:

1. **AWS CLI (Recommended for DevOps & Automation)**
2. **AWS Management Console (GUI Approach)**

---

# Project Overview

This project deploys a complete serverless ETL pipeline using AWS CloudFormation.

The CloudFormation template provisions:

- Amazon S3 (Raw Data Bucket)
- Amazon S3 (Processed Data Bucket)
- AWS Lambda Function
- IAM Role
- AWS Secrets Manager Secret
- CloudWatch Log Group
- S3 Event Trigger

Repository Structure

```
day50-iac-serverless-wrapup
│
├── data
│   └── orders.csv
│
├── infra
│   └── cloudformation
│       └── etl-stack.yaml
│
├── lambda
│   └── etl_orders
│       ├── lambda_function.py
│       ├── db_config.py
│       └── transform.py
│
├── scripts
└── README.md
```

---

# Prerequisites

- AWS Account
- AWS CLI v2 Installed
- Git Installed
- Repository Cloned
- PowerShell / Command Prompt

Clone repository

```bash
git clone https://github.com/manangupta12/aws-data-engineering-course.git

cd aws-data-engineering-course/day50-iac-serverless-wrapup
```

---

# Method 1 — Deploy Using AWS CLI (Recommended)

## Step 1 — Verify AWS CLI

```bash
aws --version
```

Example

```
aws-cli/2.x.x
```

---

## Step 2 — Configure AWS CLI

Create an Access Key from

```
AWS Console
↓
IAM
↓
Security Credentials
↓
Create Access Key
↓
Command Line Interface (CLI)
```

Configure CLI

```bash
aws configure
```

Example

```
AWS Access Key ID:
****************

AWS Secret Access Key:
**************************************

Default region:
ap-south-1

Default output:
json
```

Verify

```bash
aws sts get-caller-identity
```

Expected

```json
{
    "UserId": "...",
    "Account": "...",
    "Arn": "arn:aws:iam::<ACCOUNT_ID>:root"
}
```

---

## Step 3 — Package Lambda Function

Navigate to Lambda source

```powershell
cd lambda\etl_orders
```

Create deployment package

```powershell
Compress-Archive -Path * -DestinationPath ..\..\function.zip -Force
```

Return to project

```powershell
cd ..\..
```

Verify

```powershell
dir function.zip
```

---

## Step 4 — Create S3 Artifact Bucket

Create bucket

```bash
aws s3 mb s3://day50-serverless-artifacts-<ACCOUNT_ID> --region ap-south-1
```

Verify

```bash
aws s3 ls
```

---

## Step 5 — Upload Lambda Package

```bash
aws s3 cp function.zip s3://day50-serverless-artifacts-<ACCOUNT_ID>/function.zip
```

Verify upload

```bash
aws s3 ls s3://day50-serverless-artifacts-<ACCOUNT_ID>
```

Expected

```
function.zip
```

---

## Step 6 — Validate CloudFormation Template

```bash
aws cloudformation validate-template --template-body file://infra/cloudformation/etl-stack.yaml
```

If valid

```
Parameters
Capabilities
Description
```

---

## Step 7 — Deploy CloudFormation Stack

```bash
aws cloudformation deploy \
--template-file infra/cloudformation/etl-stack.yaml \
--stack-name day50-serverless-etl \
--parameter-overrides \
ProjectPrefix=day50demo \
LambdaArtifactBucket=day50-serverless-artifacts-<ACCOUNT_ID> \
LambdaArtifactKey=function.zip \
--capabilities CAPABILITY_NAMED_IAM
```

Expected

```
Waiting for changeset...

Waiting for stack create/update...

Successfully created/updated stack
```

---

## Step 8 — Verify CloudFormation Stack

```bash
aws cloudformation describe-stacks \
--stack-name day50-serverless-etl
```

Status

```
CREATE_COMPLETE
```

---

## Step 9 — View Stack Outputs

```bash
aws cloudformation describe-stacks \
--stack-name day50-serverless-etl \
--query "Stacks[0].Outputs"
```

Outputs include

- Raw Bucket
- Processed Bucket
- Lambda Function
- Secret ARN

---

## Step 10 — Verify Resources

### S3

```bash
aws s3 ls
```

Expected

```
day50demo-orders-raw

day50demo-orders-processed

day50-serverless-artifacts-<ACCOUNT_ID>
```

---

### Lambda

```bash
aws lambda list-functions
```

---

### Secrets Manager

```bash
aws secretsmanager list-secrets
```

---

### CloudWatch

```bash
aws logs describe-log-groups
```

---

## Step 11 — Upload Sample CSV

```bash
aws s3 cp data/orders.csv s3://day50demo-orders-raw/incoming/orders.csv
```

---

## Step 12 — Verify Processed Files

```bash
aws s3 ls s3://day50demo-orders-processed --recursive
```

---

## Step 13 — View Lambda Logs

```bash
aws logs tail /aws/lambda/day50demo-etl-orders --follow
```

---

# Method 2 — Deploy Using AWS Management Console

## Step 1 — Upload Lambda Package

Open

```
Amazon S3
```

Create bucket

```
day50-serverless-artifacts-<ACCOUNT_ID>
```

Open bucket

Click

```
Upload
```

Upload

```
function.zip
```

---

## Step 2 — Open CloudFormation

```
AWS Console

↓

CloudFormation

↓

Create Stack

↓

With new resources (standard)
```

---

## Step 3 — Upload Template

Choose

```
Upload template file
```

Upload

```
infra/cloudformation/etl-stack.yaml
```

Click

```
Next
```

---

## Step 4 — Configure Stack

Stack Name

```
day50-serverless-etl
```

Parameters

```
ProjectPrefix

day50demo
```

```
LambdaArtifactBucket

day50-serverless-artifacts-<ACCOUNT_ID>
```

```
LambdaArtifactKey

function.zip
```

Click

```
Next
```

---

## Step 5 — Configure Options

Leave default settings.

Click

```
Next
```

---

## Step 6 — Review

Enable

```
I acknowledge that AWS CloudFormation might create IAM resources.
```

Click

```
Submit
```

Deployment starts.

---

## Step 7 — Monitor Stack

Open

```
CloudFormation

↓

Stacks

↓

day50-serverless-etl
```

Wait until

```
CREATE_COMPLETE
```

---

## Step 8 — View Resources

CloudFormation

```
Resources
```

Displays

- Lambda
- IAM Role
- S3 Buckets
- Secrets Manager
- CloudWatch

---

## Step 9 — View Outputs

CloudFormation

```
Outputs
```

Displays

- Raw Bucket
- Processed Bucket
- Lambda Name
- Secret ARN

---

## Step 10 — Upload Sample CSV

Open

```
Amazon S3

↓

Raw Bucket

↓

Upload
```

Upload

```
orders.csv
```

Destination

```
incoming/orders.csv
```

---

## Step 11 — Verify Processed Output

Open

```
Processed Bucket
```

Verify processed files are generated.

---

## Step 12 — View Logs

Open

```
CloudWatch

↓

Log Groups

↓

/aws/lambda/day50demo-etl-orders
```

Review Lambda execution logs.

---

# Services Used

- AWS CloudFormation
- Amazon S3
- AWS Lambda
- AWS IAM
- AWS Secrets Manager
- Amazon CloudWatch
- AWS CLI

---

# Deployment Workflow

```
Lambda Source Code
        │
        ▼
Create function.zip
        │
        ▼
Upload ZIP to S3
        │
        ▼
Deploy CloudFormation Stack
        │
        ▼
CloudFormation Creates
        │
 ┌──────┼───────────────┐
 │      │               │
 ▼      ▼               ▼
S3    Lambda          IAM
 │
 ▼
Upload CSV
 │
 ▼
S3 Event Trigger
 │
 ▼
Lambda Executes
 │
 ▼
Transform Data
 │
 ▼
Processed Bucket
 │
 ▼
CloudWatch Logs
```

---

# Cleanup

Delete the CloudFormation Stack

CLI

```bash
aws cloudformation delete-stack --stack-name day50-serverless-etl
```

or

```
AWS Console

↓

CloudFormation

↓

day50-serverless-etl

↓

Delete
```

CloudFormation automatically removes all associated resources created by the stack.

---

# Outcome

After successful deployment:

- CloudFormation Stack Created
- Lambda Function Deployed
- IAM Role Created
- Raw S3 Bucket Created
- Processed S3 Bucket Created
- Secrets Manager Secret Created
- CloudWatch Log Group Created
- ETL Pipeline Ready for CSV Processing
- Infrastructure Managed as Code
