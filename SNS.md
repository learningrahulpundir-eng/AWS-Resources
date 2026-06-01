# Amazon SNS (Simple Notification Service)

## What is SNS?
Amazon SNS is a fully managed publish/subscribe messaging service. It enables you to send a single message and deliver it to many subscribers at once.

### Common uses
- Real-time notifications
- Event broadcasting
- Decoupling microservices

## How SNS works
SNS uses a simple publisher → topic → subscriber pattern:

1. A publisher sends a message to an SNS topic.
2. The topic acts as a communication channel.
3. SNS pushes the message to all subscribers.

## Key components
- **Topic**
  - The logical access point for messages.
  - Example: `order-topic`.
- **Publisher**
  - The service or application that sends messages.
  - Examples: Lambda, EC2, API Gateway.
- **Subscriber**
  - Receives messages from the topic.
  - Supported endpoints include:
    - Email
    - SMS
    - HTTP/HTTPS
    - AWS Lambda
    - SQS queues

## Delivery model
- **Push-based delivery**
- SNS sends messages instantly to subscribers.
- No polling is required.

## Key features
- Fan-out messaging (1 → many)
- High scalability
- Fully managed (no server setup)
- Integrates with many AWS services
- Supports basic message filtering

## Simple use case
When a new order is placed:
- Send an email to the customer
- Notify the billing system
- Trigger the shipping service

## Diagram
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
