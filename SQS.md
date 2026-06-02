# Amazon SQS

## What is SQS?

Amazon SQS is a fully managed message queue service that helps applications communicate safely and independently.

In simple terms, it stores messages temporarily until they are processed.

## Real-life Example

Think of SQS like a post box 📬:

- Sender puts a letter (message) in the box.
- Receiver comes later and collects it.
- Even if the receiver is busy, the message is not lost.

## How SQS Works

1. **Producer sends a message**
   - An application or service (for example, SNS) sends a message to SQS.

2. **Message is stored in the queue**
   - SQS keeps the message safely until a consumer is ready.

3. **Consumer reads the message**
   - A consumer, such as AWS Lambda, retrieves the message.

4. **Message becomes invisible**
   - After reading, the message is hidden from other consumers.
   - This period is called the *visibility timeout*.

5. **Consumer processes the message**
   - For example, the consumer could stop an EC2 instance.

6. **Message is deleted**
   - If processing succeeds, the consumer deletes the message.
   - Once deleted, the message is gone permanently.

## What happens if processing fails?

- If the consumer does not delete the message, the visibility timeout expires.
- The message becomes visible again in the queue.
- Another consumer can then process it.

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
