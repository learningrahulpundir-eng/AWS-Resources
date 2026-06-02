# Amazon SQS

## What is SQS?

Amazon SQS is a fully managed message queue service that helps applications communicate safely and independently.

In simple terms, SQS stores messages temporarily until they are processed.

## Real-life Example

Think of SQS like a post box 📬:

- A sender drops a message into the box.
- A receiver collects it later.
- If the receiver is busy, the message still waits safely.

## How SQS Works

1. **Producer sends a message**
   - An application or service (for example, SNS) sends a message to SQS.

2. **SQS stores the message**
   - The message stays in the queue until a consumer retrieves it.

3. **Consumer reads the message**
   - A consumer, such as AWS Lambda, reads the message from the queue.

4. **Message becomes invisible**
   - After reading, the message is hidden from other consumers.
   - This is called the *visibility timeout*.

5. **Consumer processes the message**
   - For example, the consumer may stop an EC2 instance.

6. **Message is deleted**
   - If processing succeeds, the message is removed from the queue.
   - Once deleted, the message is gone permanently.

## What happens if processing fails?

- If the consumer does not delete the message, the visibility timeout expires.
- The message becomes visible again in the queue.
- Another consumer can then try processing it.

## Simple Definition

Amazon SQS is a managed service for sending, storing, and receiving messages in a reliable and decoupled way.

## Example Flow

`SNS → SQS → Lambda → EC2 Stop`

- SNS sends the message.
- SQS stores it.
- Lambda reads and processes it.

## Why use SQS?

- Prevents data loss
- Handles traffic spikes
- Decouples systems
- Adds reliability

---

## Architecture Overview

SNS → SQS → Lambda → EC2 Automation with Email

```text
+------------------+
|   SNS Topic      |
+------------------+
        ↓
+------------------+
|   SQS Queue      |
+------------------+
        ↓
+------------------+
|     Lambda       |
+------------------+
   ↓            ↓
Stop EC2    Email Alert
```

## Step-by-step Implementation

### Step 1: Create the SQS queue

1. Open the AWS Console and go to SQS.
2. Click **Create queue**.
3. Choose **Standard** queue type.
4. Name it `EC2StopQueue`.
5. Click **Create**.

Copy these values for later:

- Queue URL
- Queue ARN

### Step 2: Subscribe SQS to SNS

1. Go to SNS and open the topic `EC2StopTopic`.
2. Click **Create subscription**.
3. Set **Protocol** to `Amazon SQS`.
4. Set **Endpoint** to the `EC2StopQueue` queue.
5. Click **Create subscription**.

### Step 3: Allow SNS to send messages to SQS

1. Open SQS and select `EC2StopQueue`.
2. Click **Access Policy** → **Edit**.
3. Add this policy, replacing the placeholders with your real ARNs:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "Allow-SNS-SendMessage",
      "Effect": "Allow",
      "Principal": "*",
      "Action": "SQS:SendMessage",
      "Resource": "YOUR_SQS_ARN",
      "Condition": {
        "ArnEquals": {
          "aws:SourceArn": "YOUR_SNS_TOPIC_ARN"
        }
      }
    }
  ]
}
```

Replace:

- `YOUR_SQS_ARN`
- `YOUR_SNS_TOPIC_ARN`

### Step 4: Add SQS as a Lambda trigger

1. Open Lambda and select `StopEC2Function`.
2. Click **Add trigger**.
3. Select **SQS** as the source.
4. Choose the `EC2StopQueue` queue.
5. Click **Add**.

The flow is now:

`SNS → SQS → Lambda`

### Step 5: Update the Lambda IAM role

1. Open IAM and select the role `LambdaEC2StopRole`.
2. Attach the required SQS permissions.

For demo purposes, attach `AmazonSQSFullAccess`.

### Step 6: Update the Lambda code

Now the Lambda function receives an SQS event rather than a direct SNS event.

```python
import json
import boto3

ec2 = boto3.client('ec2')
INSTANCE_ID = 'i-xxxxxxxxxxxx'


def lambda_handler(event, context):
    try:
        print('Received SQS event:', json.dumps(event))

        for record in event['Records']:
            body = json.loads(record['body'])
            print('Message from SNS via SQS:', body)

        ec2.stop_instances(InstanceIds=[INSTANCE_ID])
        print('EC2 stopped successfully')

        return {
            'statusCode': 200,
            'body': 'EC2 stopped'
        }

    except Exception as e:
        print('Error:', str(e))
        raise
```

### Step 7: Keep email notifications

If SNS already sends email notifications, no change is needed.

The message path becomes:

- SNS sends a message to SQS
- SNS also sends email notifications

### Step 8: Test the full flow

1. Open SNS and publish a message.
2. Use `Subject: EC2 Stop with SQS`.
3. Use a message body like `Triggering EC2 stop via SQS pipeline`.

Expected result:

- SNS publishes the message.
- SQS receives the message.
- Lambda processes the message and stops the EC2 instance.
- Email notification is still sent.
