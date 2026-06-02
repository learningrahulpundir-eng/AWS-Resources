# Amazon EventBridge — Overview

Amazon EventBridge is a serverless event bus that lets AWS services, applications, and SaaS partners communicate using events. It enables event-driven architectures by routing events to targets based on rules.

## Simple definition

EventBridge receives events and triggers actions automatically according to matching rules.

## Key concepts

- **Event**: A record of something that happened. Examples: EC2 instance state changes, file uploaded to S3, scheduled time trigger, user login.
- **Event bus**: Where events are received and processed.
	- Default — AWS service events
	- Custom — Application events you send
	- Partner — Events from SaaS partners (e.g., Shopify)
- **Rule**: Matches incoming events and routes them to targets. Rules can filter by `source`, `detail-type`, and `detail` fields.

	Example rule (match EC2 instances that stopped):

```json
{
	"source": ["aws.ec2"],
	"detail-type": ["EC2 Instance State-change Notification"],
	"detail": { "state": ["stopped"] }
}
```

- **Target**: The destination that acts when a rule matches. Common targets: Lambda, SNS, SQS, Step Functions, API Gateway.

## How EventBridge works (step-by-step)

1. Event occurs (e.g., an EC2 instance stops).
2. The event is sent to EventBridge.
3. Rules evaluate the event and select matching targets.
4. EventBridge delivers the event to the configured target(s) (for example, invoke a Lambda function).

**Basic flow:** Event Source → EventBridge → Rule → Target

## Example architecture

EC2 state change → EventBridge → Lambda → (notify / stop instance / update DB)

## When to use EventBridge

- Decouple producers and consumers in distributed systems.
- Build serverless workflows triggered by events.
- Integrate third-party SaaS events into your AWS account.

## Quick tips

- Use fine-grained rule filters to reduce unnecessary target invocations.
- Test rules with sample events in the console or via the AWS CLI.
- Consider retry and dead-letter configurations for resilient target delivery.

---

_File rewritten for clarity and easier scanning._

## EventBridge Scheduler

EventBridge supports scheduled rules using cron or rate expressions. Use these to trigger targets on a schedule without running servers.

Example — stop an EC2 instance daily at 10:00 PM (UTC):

```text
cron(0 22 * * ? *)
```

Quick notes:

- Cron expressions use UTC by default; convert times if you need a local timezone.
- You can also use `rate()` expressions, e.g. `rate(1 day)` for simple intervals.
- Test scheduled rules with short intervals before applying them in production.
- Ensure the rule has permissions to invoke the target and configure retries / dead-letter queues for resilience.