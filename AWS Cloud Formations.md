# AWS CloudFormation

## What is AWS CloudFormation?

AWS CloudFormation is an Infrastructure-as-Code (IaC) service that lets you define and provision AWS infrastructure using templates (YAML or JSON).

Instead of creating resources manually in the AWS console, you describe them in a template, and CloudFormation automatically creates, updates, and deletes resources in a consistent and repeatable way.

### Key Benefits:

- ✅ Automates infrastructure setup
- ✅ Version control for infrastructure
- ✅ Reduces manual errors
- ✅ Easy to replicate environments (dev, test, prod)
- ✅ Supports dependencies between resources

---

## Step-by-Step Guide: Create a Lambda Function using CloudFormation

Below is a simple guide to create an AWS Lambda function using a CloudFormation template.

### Step 1: Prepare Your Lambda Code

Create a simple Lambda function (Python example):

```python
def lambda_handler(event, context):
    return "Hello from Lambda using CloudFormation!"
```

Zip this file:

```bash
zip function.zip lambda_function.py
```

Upload the ZIP file to an S3 bucket:

```bash
aws s3 cp function.zip s3://your-bucket-name/function.zip
```

---

### Step 2: Create CloudFormation Template

Create a file called `template.yaml`:

```yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: AWS Lambda using CloudFormation

Resources:
  MyLambdaExecutionRole:
    Type: AWS::IAM::Role
    Properties:
      RoleName: lambda-execution-role
      AssumeRolePolicyDocument:
        Version: '2012-10-17'
        Statement:
          - Effect: Allow
            Principal:
              Service:
                - lambda.amazonaws.com
            Action:
              - sts:AssumeRole
      ManagedPolicyArns:
        - arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole

  MyLambdaFunction:
    Type: AWS::Lambda::Function
    Properties:
      FunctionName: MyCloudFormationLambda
      Runtime: python3.9
      Handler: lambda_function.lambda_handler
      Role: !GetAtt MyLambdaExecutionRole.Arn
      Code:
        S3Bucket: your-bucket-name
        S3Key: function.zip
      Timeout: 10
```

---

### Step 3: Deploy the CloudFormation Stack

Use AWS CLI to create the stack:

```bash
aws cloudformation create-stack \
  --stack-name MyLambdaStack \
  --template-body file://template.yaml \
  --capabilities CAPABILITY_NAMED_IAM
```

---

### Step 4: Monitor Stack Creation

Check stack status:

```bash
aws cloudformation describe-stacks --stack-name MyLambdaStack
```

Wait until status becomes: **CREATE_COMPLETE**

---

### Step 5: Test the Lambda Function

Invoke your Lambda:

```bash
aws lambda invoke \
  --function-name MyCloudFormationLambda \
  output.txt
```

Check output:

```bash
cat output.txt
```

Expected result: `"Hello from Lambda using CloudFormation!"`

---

### Step 6: (Optional) Delete Stack

To clean up resources:

```bash
aws cloudformation delete-stack --stack-name MyLambdaStack
```

---

## Summary

| Step | Action |
|------|--------|
| 1 | Created Lambda code |
| 2 | Uploaded code to S3 |
| 3 | Wrote CloudFormation template |
| 4 | Deployed stack |
| 5 | Tested Lambda |
| 6 | (Optional) Deleted stack |
