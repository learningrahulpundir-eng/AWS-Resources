### Scenario-1: When we want to create one lambda function and we have one python file for lambda.

📁 Example Project Structure
project/
│
├── template.yaml
├── lambda/
│   └── app.py


🧾 Step 1: Zip your Python code

cd lambda
Compress-Archive -Path app.py -DestinationPath function.zip
cd ..


🧾 Step 2: Upload ZIP to S3
aws s3 mb s3://my-deployment-bucket   # create bucket (once)

aws s3 cp lambda/function.zip s3://my-deployment-bucket/function.zip

🧾 Step 3: Update CloudFormation Template
Reference S3 file inside template:

```yaml
AWSTemplateFormatVersion: '2010-09-09'
Transform: AWS::Serverless-2016-10-31
Description: Lambda with inline code

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
    Type: AWS::Serverless::Function
    Properties:
      FunctionName: my-sample-lambda
      Runtime: python3.12
      Handler: app.lambda_handler   
      CodeUri: s3://my-deployment-bucket/function.zip
      Role: !GetAtt MyLambdaExecutionRole.Arn
      MemorySize: 128
      Timeout: 10
```

🧾 Step 4: Deploy using AWS CLI
 aws cloudformation create-stack \
  --stack-name my-stack \
  --template-body file://template.yaml \
  --capabilities CAPABILITY_NAMED_IAM

🧾 Step 5: Monitor Deployment
    aws cloudformation describe-stacks --stack-name my-stack