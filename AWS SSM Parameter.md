# AWS Systems Manager Parameter Store

AWS Systems Manager (SSM) Parameter Store is a service for securely storing configuration values.

## Common use cases

- Database passwords
- API keys
- Environment variables
- Application configuration values (URLs, feature flags)

## Parameter types

| Type | Description |
|------|-------------|
| String | Plaintext |
| StringList | Comma-separated values |
| SecureString | Encrypted (recommended for secrets) |

## Step-by-step example

1. Create a parameter using the AWS CLI:
   ```bash
   aws ssm put-parameter --name "/myapp/db/password" --value "mysecret123" --type SecureString
   ```
2. Verify the parameter exists:
   ```bash
   aws ssm get-parameter --name "/myapp/db/password" --with-decryption
   ```
3. Example Lambda function that reads the parameter:
   ```python
   import boto3

   def get_parameter(param_name):
       ssm = boto3.client('ssm')
       response = ssm.get_parameter(
           Name=param_name,
           WithDecryption=True  # required for SecureString
       )
       return response['Parameter']['Value']

   def lambda_handler(event, context):
       password = get_parameter("/myapp/db/password")
       print("Password from SSM:", password)
       return {
           'statusCode': 200,
           'body': password
       }
   ```

## Required IAM permissions

The Lambda execution role should include at least:

- `AWSLambdaBasicExecutionRole`
- `AmazonSSMReadOnlyAccess`

> Note: For SecureString parameters, the role may also need permissions to use the KMS key that encrypts the parameter.



