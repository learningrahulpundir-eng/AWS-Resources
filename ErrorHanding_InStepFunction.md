### Create a Lambda function

```python
import json

def lambda_handler(event, context):
    print("Lambda invoked")

    # Force an error
    raise Exception("Something went wrong in Lambda!")

    return {
        "statusCode": 200,
        "body": json.dumps("Success")
    }
```

### Step Function definition with error handling

This state machine will:
- call the Lambda function
- retry on failure up to 2 times
- if the Lambda still fails, catch the error and move to an error handler

```json
{
  "Comment": "Demo for Lambda error handling",
  "StartAt": "CallLambda",
  "States": {
    "CallLambda": {
      "Type": "Task",
      "Resource": "arn:aws:lambda:region:account-id:function:YourLambdaName",
      "Retry": [
        {
          "ErrorEquals": ["States.ALL"],
          "IntervalSeconds": 2,
          "MaxAttempts": 2,
          "BackoffRate": 2.0
        }
      ],
      "Catch": [
        {
          "ErrorEquals": ["States.ALL"],
          "ResultPath": "$.errorInfo",
          "Next": "HandleError"
        }
      ],
      "Next": "SuccessState"
    },
    "SuccessState": {
      "Type": "Pass",
      "Result": "Lambda succeeded",
      "End": true
    },
    "HandleError": {
      "Type": "Pass",
      "Result": "Error handled successfully",
      "End": true
    }
  }
}
```

### What happens when this runs
1. The Step Function starts at `CallLambda`.
2. The Lambda raises an exception.
3. Step Functions retries the task twice (three attempts total).
4. If the task still fails, execution moves to `HandleError`.
5. The error details are stored in `$.errorInfo`.

### Example output after error handling

```json
{
  "errorInfo": {
    "Error": "Exception",
    "Cause": "Something went wrong in Lambda!"
  }
}
```

### Optional: simulate custom errors
Use the same retry/catch pattern with a custom error name in `ErrorEquals` to handle specific failure types.

### Real-world enhancements
- Send an SNS notification from `HandleError`
- Log failures to DynamoDB
- Trigger compensation or rollback logic
- Send email alerts
- Publish CloudWatch metrics and alarms

### Final flow visualization

CallLambda
   ↓
[Error]
   ↓
Retry (2 times)
   ↓
Still fails?
   ↓ YES
Catch → HandleError ✅
