# Amazon SNS (Simple Notification Service)

## What is SNS?
Amazon SNS is a fully managed publish/subscribe messaging service. It lets you send one message and deliver it to many subscribers instantly.

### Common uses
- Real-time notifications
- Event broadcasting
- Decoupling microservices
- Alerting and monitoring workflows

## How SNS works
SNS follows a simple pattern:

1. A publisher sends a message to an SNS topic.
2. The topic acts as a message hub.
3. SNS pushes the message to all subscribers.

## Key components
- **Topic**: The logical access point for messages.
- **Publisher**: The sender, such as Lambda, EC2, or API Gateway.
- **Subscriber**: The receiver, such as an email address, SMS number, HTTP endpoint, Lambda function, or SQS queue.

## Delivery model
- Push-based delivery
- Messages are delivered immediately
- Subscribers do not need to poll for updates

## Key features
- Fan-out messaging (1 → many)
- High scalability and availability
- Fully managed service
- Integrates with AWS services like Lambda, SQS, and CloudWatch
- Supports message filtering and retry policies

## Example use case
When a new order is placed:
- Send an email confirmation to the customer
- Notify the billing service
- Trigger the shipping workflow

## Architecture diagram
```text
              +--------------------+
              |    Publisher       |
              |  (App / Lambda)    |
              +---------+----------+
                        |
                        v
               +------------------+
               |    SNS Topic     |
               |   "OrderTopic"  |
               +--------+---------+
                        |
        -------------------------------------
        |                |                 |
        v                v                 v
+---------------+  +--------------+  +-------------+
|   Email       |  |   SQS Queue  |  |   Lambda    |
| Notification  |  |  (Billing)   |  | (Shipping)  |
+---------------+  +--------------+  +-------------+
```

## Demo: Event-driven EC2 shutdown with SNS
This demo shows how to use SNS to trigger a Lambda function and send an email notification when an EC2 instance is stopped.

### Architecture overview
- An SNS topic receives the trigger event.
- An email subscription notifies you.
- A Lambda function stops the EC2 instance.

### Setup steps

#### 1. Create the SNS topic
1. Open the AWS Console and go to SNS.
2. Click **Create topic**.
3. Select **Standard**.
4. Enter the name: `EC2StopTopic`.
5. Click **Create topic**.
6. Copy the Topic ARN for later.

#### 2. Add an email subscription
1. Open `EC2StopTopic`.
2. Go to **Subscriptions**.
3. Click **Create subscription**.
4. Set **Protocol** to `Email`.
5. Set **Endpoint** to your email address.
6. Click **Create subscription**.
7. Confirm the subscription from the email message.

#### 3. Create an IAM role for Lambda
1. Open IAM and go to **Roles**.
2. Click **Create role**.
3. Choose **Lambda** as the trusted entity.
4. Attach these policies:
   - `AmazonEC2FullAccess`
   - `AWSLambdaBasicExecutionRole`
5. Name the role `LambdaEC2StopRole`.
6. Click **Create role**.

#### 4. Create the Lambda function
1. Open Lambda and click **Create function**.
2. Choose **Author from scratch**.
3. Set **Runtime** to `Python 3.x`.
4. Set **Function name** to `StopEC2Function`.
5. Attach the `LambdaEC2StopRole` role.
6. Click **Create function**.

#### 5. Add the SNS trigger to Lambda
1. Open the `StopEC2Function` function.
2. Click **Add trigger**.
3. Select **SNS**.
4. Choose the topic `EC2StopTopic`.
5. Click **Add**.

#### 6. Add Lambda code
Replace the default Lambda code with the following Python function:

```python
import json
import boto3

ec2 = boto3.client('ec2')
INSTANCE_ID = 'i-xxxxxxxxxxxx'  # Replace with your EC2 instance ID


def lambda_handler(event, context):
    try:
        print('Received SNS event:', json.dumps(event))

        response = ec2.stop_instances(
            InstanceIds=[INSTANCE_ID]
        )

        print('EC2 Stop Response:', response)
        return {
            'statusCode': 200,
            'body': 'EC2 instance stopped successfully'
        }

    except Exception as e:
        print('Error:', str(e))
        raise
```

- Replace `INSTANCE_ID` with the actual EC2 instance ID.

#### 7. Start your EC2 instance
1. Open the EC2 console.
2. Confirm the target instance is in the **running** state.

#### 8. Publish a message to SNS
1. Open SNS and select `EC2StopTopic`.
2. Click **Publish message**.
3. Set a subject like `EC2 Stop Alert`.
4. Enter a message body such as `EC2 instance stop process initiated via SNS`.
5. Click **Publish**.

### Expected results
- SNS publishes the message.
- Lambda receives the SNS event.
- Lambda stops the EC2 instance.
- You receive an email notification.

### Verify the demo
- Check the EC2 instance state: `running → stopping → stopped`.
- Confirm the subscription email 
  contains the subject `EC2 Stop Alert`.

## Notes
- SNS is ideal for fan-out patterns and real-time notifications.
- Use subscriptions such as Lambda, SQS, or email depending on your workflow.
- Standard topics support high throughput and best-effort ordering.

Click Publish message

Fill:


Subject:
EC2 Stop Alert


Message:
EC2 instance stop process initiated via SNS




Click Publish


✅ ✅ Expected Results

























ServiceResultSNSSends message ✅LambdaTriggered ✅EC2Stops (running → stopped) ✅EmailNotification received ✅

🔹 Step 10: Verify Everything
✅ Check EC2

Go to EC2 → Instance state:

running → stopping → stopped




✅ Check Email
You will receive:
📧 Subject: EC2 Stop Alert
📧 Message: EC2 instance stop process initiated via SNS

✅ Check Logs (Important for demo)

Go to CloudWatch → Log groups
Open Lambda logs
Show logs to interviewer/demo audience