# Amazon EventBridge — Overview

Amazon EventBridge is a serverless event bus that lets AWS services, applications, and SaaS partners communicate using events. It enables event-driven architectures by routing events to targets based on rules.

## Simple definition

EventBridge receives events and triggers actions automatically according to matching rules.

## Key concepts

- **Event**: A record of something that happened. Examples: EC2 instance state changes, file uploaded to S3, scheduled time trigger, user login.
Demo: Stop an EC2 instance using EventBridge

Architecture: EventBridge Rule → Lambda → EC2 (Stop)

Overview: schedule a rule or trigger on an event to invoke a Lambda that calls the EC2 StopInstances API.

1) IAM role for Lambda

- Create an IAM role with the Lambda trusted entity.
- Attach `AWSLambdaBasicExecutionRole` and a minimal custom policy that allows stopping the specific instance. Example policy (least privilege):

```json
{
	"Version": "2012-10-17",
	"Statement": [
		{
			"Effect": "Allow",
			"Action": ["ec2:StopInstances"],
			"Resource": ["arn:aws:ec2:REGION:ACCOUNT_ID:instance/INSTANCE_ID"]
		}
	]
}
```

Replace `REGION`, `ACCOUNT_ID`, and `INSTANCE_ID` as appropriate.

2) Lambda function

- Create a Python 3.x Lambda named `StopEC2Function` and attach the IAM role above.
- Use an environment variable `INSTANCE_ID` instead of hardcoding the ID.

Example Lambda code:

```python
import os
import boto3

ec2 = boto3.client('ec2')

def lambda_handler(event, context):
		instance_id = os.environ.get('INSTANCE_ID')
		if not instance_id:
				raise ValueError('INSTANCE_ID environment variable not set')
		try:
				resp = ec2.stop_instances(InstanceIds=[instance_id])
				print('StopInstances response:', resp)
				return {'statusCode': 200, 'body': 'Stop initiated'}
		except Exception:
				raise
```

3) Create an EventBridge rule

- In EventBridge, create a rule and choose either:
	- Schedule: `rate(5 minutes)` or `cron(0 18 * * ? *)` (example: daily at 6 PM UTC)
	- Event pattern: match a specific AWS event source
- Set the rule target to the `StopEC2Function` Lambda and pass no special input (it will read `INSTANCE_ID` from env).

4) Test and verify

- Start the target EC2 instance (state: Running).
- Use a short `rate()` expression for testing (e.g., `rate(5 minutes)`) or use the EventBridge console's test feature.
- Check Lambda logs in CloudWatch to confirm the function ran and returned successfully.
- Verify the EC2 instance transitions: Running → Stopping → Stopped.

Expected Lambda log snippet:

```
StopInstances response: { ... }
```

Note: follow least-privilege principles — avoid `AmazonEC2FullAccess` unless you need broad EC2 permissions.
rate(5 minutes)

Option 2 (Cron):
cron(0 18 * * ? *)

✅ Example:

Runs every day at 6 PM


🔹 Step 6: Select Target

Choose:

Target type: Lambda function


Select:

StopEC2Function




🔹 Step 7: Finish Rule

Keep default settings
Click Create rule


✅ ✅ Step 8: Start EC2 Instance
Before testing:

Go to EC2
Make sure instance is in:
👉 Running state


✅ ✅ Step 9: Wait or Test
👉 If using rate(5 minutes):

Wait 5 minutes

👉 Or manually test:

Go to EventBridge → rule → test


✅ ✅ Expected Output





















ComponentResultEventBridgeTriggers ✅LambdaExecutes ✅EC2Stopped ✅

✅ ✅ Step 10: Verify
✅ EC2

Running → Stopping → Stopped