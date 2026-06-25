### Scenario 1: Deploy a single Python Lambda from S3 ZIP

📁 Example project structure

project/
├── template.yaml
└── lambda/
    └── app.py


🧾 Step 1: Package the Lambda function

```bash
cd lambda
Compress-Archive -Path app.py -DestinationPath function.zip
cd ..
```

🧾 Step 2: Upload the ZIP to S3

```bash
aws s3 mb s3://my-deployment-bucket      # create bucket once
aws s3 cp lambda/function.zip s3://my-deployment-bucket/function.zip
```

🧾 Step 3: Update the CloudFormation template

Use the S3 package for the Lambda function code.

```yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: Lambda with S3 deployment package

Resources:
  MyLambdaExecutionRole:
    Type: AWS::IAM::Role
    Properties:
      RoleName: demo-lambda-role
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
      FunctionName: my-sample-lambda
      Runtime: python3.12
      Handler: app.lambda_handler
      Code:
        S3Bucket: my-deployment-bucket
        S3Key: function.zip
      Role: !GetAtt MyLambdaExecutionRole.Arn
      MemorySize: 128
      Timeout: 10
```

🧾 Step 4: Deploy the CloudFormation stack

```bash
aws cloudformation create-stack \
  --stack-name my-stack \
  --template-body file://template.yaml \
  --capabilities CAPABILITY_NAMED_IAM
```

🧾 Step 5: Verify deployment

```bash
aws cloudformation describe-stacks --stack-name my-stack
```

---

### Scenario 2: Deploy multiple Lambda functions with CloudFormation

📁 Example project structure

project/
├── template.yaml
├── lambda1/
│   └── app.py
└── lambda2/
    └── app.py


🧾 Step 1: Upload both ZIP packages to S3

```bash
aws s3 cp lambda1/function.zip s3://my-deployment-bucket/function1.zip
aws s3 cp lambda2/function.zip s3://my-deployment-bucket/function2.zip
```

🧾 Step 2: Use a CloudFormation template for both functions

```yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: Multiple Lambda functions deployed from S3 packages

Resources:
  Lambda1:
    Type: AWS::Serverless::Function
    Properties:
      FunctionName: lambda1-function
      Runtime: python3.9
      Handler: app.lambda_handler
      CodeUri:
        Bucket: my-deployment-bucket
        Key: function1.zip

  Lambda2:
    Type: AWS::Serverless::Function
    Properties:
      FunctionName: lambda2-function
      Runtime: python3.9
      Handler: app.lambda_handler
      CodeUri:
        Bucket: my-deployment-bucket
        Key: function2.zip
```

🧾 Step 3: Deploy the stack

```bash
aws cloudformation create-stack \
  --stack-name my-multi-lambda-stack \
  --template-body file://template.yaml \
  --capabilities CAPABILITY_NAMED_IAM
```

