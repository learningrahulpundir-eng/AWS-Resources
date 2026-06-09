# Orchestration

## What is orchestration?
Orchestration automatically runs multiple workflow steps and manages their order, dependencies, retries, and errors.

Manual sequence:
1. Upload data to S3
2. Run Glue ETL
3. Query in Athena

Orchestrated sequence:
- S3 event triggers the workflow
- Glue ETL runs automatically
- Athena or downstream services consume transformed results
- Retries and error handling occur without manual intervention

## What is AWS Step Functions?
AWS Step Functions is a managed workflow orchestration service. It lets you define state machines that connect AWS services and manage execution flow.

Step Functions provides:
- Visual workflow design and execution history
- Built-in retry and error handling
- Integration with Lambda, Glue, DynamoDB, SNS, SQS, and more

### Real-world analogy
Step Functions is like a factory manager that coordinates tasks:
- receive input data
- store it in S3
- transform it with Glue
- query or publish results

## Core concepts

### State machine
A state machine is the workflow definition. It describes the ordered states and transitions.

Example:
`Start → Run Glue Job → Wait → Run Athena → End`

### States
Each workflow step is a state.

Common state types:
- **Task**: runs work such as a Glue job, Lambda function, or API call.
- **Choice**: branches execution based on conditions.
- **Wait**: pauses execution for a fixed time or timestamp.
- **Succeed / Fail**: ends the workflow successfully or with an error.

### Transitions
Transitions define the next state after the current state completes.

Example:
`Step A → Step B → Step C`

## Example pipeline
A typical orchestrated pipeline:
1. S3 receives new data
2. Lambda triggers a Step Functions execution
3. Step Functions runs Glue ETL
4. Athena queries the transformed result
5. QuickSight or another service visualizes the data

## Why Step Functions matters
- Eliminates manual workflow steps
- Supports automatic retries on failure
- Captures execution history and logs
- Offers visual debugging in the console

## Free tier
Step Functions free tier:
- 4,000 state transitions per month
- Good for learning and small workloads

## Demo 1 — Quick start
Steps:
1. Upload a CSV file to S3
2. Create Glue ETL job(s) to process the file
3. Create a Step Functions state machine to run the ETL jobs

## Demo 2 — Parameterized ETL
Steps:
1. Upload a CSV file to S3
2. Create a parameterized Glue ETL job that accepts `--input_path` and `--output_path`
3. Use a Step Functions state machine to pass parameters and invoke the job

### Python Glue job
Copy this script into the Glue script editor:

```python
import sys
from awsglue.transforms import *
from awsglue.utils import getResolvedOptions
from pyspark.context import SparkContext
from awsglue.context import GlueContext
from awsglue.job import Job

args = getResolvedOptions(sys.argv, ['JOB_NAME', 'input_path', 'output_path'])
input_path = args['input_path']
output_path = args['output_path']

sc = SparkContext()
glueContext = GlueContext(sc)
job = Job(glueContext)
job.init(args['JOB_NAME'], args)

datasource = glueContext.create_dynamic_frame.from_options(
    connection_type='s3',
    connection_options={'paths': [input_path]},
    format='csv',
    format_options={'withHeader': True}
)

filtered_data = Filter.apply(
    frame=datasource,
    f=lambda row: row.get('age') is not None and str(row.get('age')).isdigit() and int(row.get('age')) > 25
)

glueContext.write_dynamic_frame.from_options(
    frame=filtered_data,
    connection_type='s3',
    connection_options={'path': output_path},
    format='csv'
)

print('ETL job with filter completed successfully')
job.commit()
```

### Run the Glue job with parameters
1. Open the Glue console → Jobs → Run job
2. Add parameters:
   - `--input_path`
   - `--output_path`

### Step Functions state machine
Copy this state machine definition into Step Functions:

```json
{
  "StartAt": "Glue StartJobRun",
  "States": {
    "Glue StartJobRun": {
      "Type": "Task",
      "Resource": "arn:aws:states:::glue:startJobRun",
      "Parameters": {
        "JobName": "parameter_jobs",
        "Arguments": {
          "--input_path.$": "$.input_path",
          "--output_path.$": "$.output_path"
        }
      },
      "End": true
    }
  }
}
```

### Example execution input

```json
{
  "input_path": "s3://your-bucket/input/users.csv",
  "output_path": "s3://your-bucket/output/"
}
```

## Demo 3 — Industry use case
Steps:
1. Upload CSV to S3
2. Trigger a Step Functions execution (via S3 event or Lambda)
3. Run a Glue Crawler to update the data catalog
4. Run a Glue ETL job to transform data and write results to S3
5. Invoke a Lambda to process transformed output
6. Insert processed records into DynamoDB

### Architecture overview

```text
S3 (CSV upload)
  └─> Lambda (S3 event)
      └─> Step Functions
          ├─> Run Glue Crawler
          ├─> Run Glue ETL Job
          └─> Run Lambda (post-processing)
              └─> DynamoDB
```

### Step 1: Create the S3 bucket
- Create a bucket named `pipeline-demo-bucket`
- Create folders:
  - `input/`
  - `output/`

Upload your CSV file to:

```text
s3://pipeline-demo-bucket/input/data.csv
```

### Step 2: Create a Glue Crawler
- Go to Glue → Crawlers → Create crawler
- Data source: `s3://pipeline-demo-bucket/input/`
- Select or create an IAM role
- Output:
  - Database: `demo_db`
  - Table: auto-created
- Run the crawler once and verify the table exists in the Glue Data Catalog

### Step 3: Create a Glue ETL job
- Go to Glue → Jobs → Create job
- Use a parameterized script
- Job name: `etl-demo-job`
- Example run parameters:
  - `--input_path=s3://pipeline-demo-bucket/input/employee.csv`
  - `--output_path=s3://pipeline-demo-bucket/output/`

#### Glue ETL script

```python
import sys
from awsglue.transforms import *
from awsglue.utils import getResolvedOptions
from pyspark.context import SparkContext
from awsglue.context import GlueContext
from awsglue.job import Job

args = getResolvedOptions(sys.argv, ['JOB_NAME', 'input_path', 'output_path'])
input_path = args['input_path']
output_path = args['output_path']

sc = SparkContext()
glueContext = GlueContext(sc)
job = Job(glueContext)
job.init(args['JOB_NAME'], args)

datasource = glueContext.create_dynamic_frame.from_options(
    connection_type='s3',
    connection_options={'paths': [input_path]},
    format='csv',
    format_options={'withHeader': True}
)

filtered = Filter.apply(
    frame=datasource,
    f=lambda row: row.get('age') is not None and row.get('age').isdigit() and int(row.get('age')) > 25
)

glueContext.write_dynamic_frame.from_options(
    frame=filtered,
    connection_type='s3',
    connection_options={'path': output_path},
    format='csv'
)

job.commit()
```

### Step 4: Create the DynamoDB table
- Go to DynamoDB → Create table
- Table name: `demo_table`
- Partition key: `id` (String)

### Step 5: Create Lambda to load processed output into DynamoDB
- Go to Lambda → Create function
- Runtime: Python
- Add permissions:
  - S3 read
  - DynamoDB write

#### Lambda function code

```python
import csv
import boto3

s3 = boto3.client('s3')
dynamodb = boto3.resource('dynamodb')
table = dynamodb.Table('Users')

def lambda_handler(event, context):
    bucket = 'rahul-demo-bucket2017'
    prefix = 'output/'

    # ✅ List all objects in folder
    response = s3.list_objects_v2(Bucket=bucket, Prefix=prefix)

    if 'Contents' not in response:
        print("No files found")
        return {"status": "no files"}

    # ✅ Filter only valid data files
    files = [
        obj for obj in response['Contents']
        if 'part-' in obj['Key']  # Glue output files
    ]

    if not files:
        print("No valid data files found")
        return {"status": "no valid files"}

    # ✅ Get latest file based on LastModified
    latest_file = max(files, key=lambda x: x['LastModified'])
    latest_key = latest_file['Key']

    print(f"✅ Processing latest file: {latest_key}")

    # ✅ Read the file
    response = s3.get_object(Bucket=bucket, Key=latest_key)
    content = response['Body'].read().decode('utf-8').splitlines()
    reader = csv.DictReader(content)

    success_count = 0

    for row in reader:
        table.put_item(Item=row)
        success_count += 1

    print(f"Inserted {success_count} records from {latest_key}")

    return {
        "status": "success",
        "file_processed": latest_key,
        "records_inserted": success_count
    }
```

### Step 6: Create Step Functions workflow
- Go to Step Functions → Create state machine → Author with code

#### Step Function definition

```json
{
  "StartAt": "RunCrawler",
  "States": {
    "RunCrawler": {
      "Type": "Task",
      "Resource": "arn:aws:states:::aws-sdk:glue:startCrawler",
      "Parameters": {
        "Name": "your-crawler-name"
      },
      "Next": "WaitCrawler"
    },
    "WaitCrawler": {
      "Type": "Wait",
      "Seconds": 60,
      "Next": "RunGlueJob"
    },
    "RunGlueJob": {
      "Type": "Task",
      "Resource": "arn:aws:states:::glue:startJobRun",
      "Parameters": {
        "JobName": "etl-demo-job",
        "Arguments": {
          "--input_path": "s3://pipeline-demo-bucket/input/data.csv",
          "--output_path": "s3://pipeline-demo-bucket/output/"
        }
      },
      "Next": "WaitJob"
    },
    "WaitJob": {
      "Type": "Wait",
      "Seconds": 120,
      "Next": "RunLambda"
    },
    "RunLambda": {
      "Type": "Task",
      "Resource": "arn:aws:states:::lambda:invoke",
      "Parameters": {
        "FunctionName": "your-lambda-name"
      },
      "End": true
    }
  }
}
```

### Step 7: Create Lambda trigger for Step Functions
- Go to Lambda → Create function
- Use this code:

```python
import boto3

step = boto3.client('stepfunctions')

def lambda_handler(event, context):
    step.start_execution(
        stateMachineArn='YOUR_STEP_FUNCTION_ARN'
    )
    return {'status': 'started'}
```

### Step 8: Connect S3 to the Lambda trigger
- Open the S3 bucket properties
- Add an event notification for the `PUT` event
- Set the destination to the Lambda trigger function

## Final flow
1. Upload CSV to S3
2. S3 event triggers Lambda
3. Lambda starts the Step Functions workflow
4. Step Functions starts the Glue Crawler
5. Step Functions starts the Glue ETL job
6. ETL writes output to S3
7. Lambda processes output and writes to DynamoDB
8. Process completes
