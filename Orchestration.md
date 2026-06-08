# Orchestration

## What is orchestration?
Orchestration means coordinating multiple workflow steps automatically.

Instead of doing this manually:
1. Upload data to S3
2. Run Glue ETL
3. Query in Athena

Orchestration runs these steps automatically and also handles:
- failures
- retries
- branching logic

## What is AWS Step Functions?
AWS Step Functions is a visual workflow orchestrator.
It lets you:
- define steps in a pipeline
- control execution flow
- handle retries and errors
- integrate AWS services

### Real-world analogy
Think of Step Functions as a factory manager that coordinates each task:
- receive raw data
- store it in S3
- transform it with Glue
- query it with Athena

## Core concepts
### 1. State machine
A state machine is the workflow definition.

Example:
`Start → Run Glue Job → Wait → Run Athena → End`

### 2. States
Each step in the workflow is called a state.

Common state types:
- **Task State**
  - runs a task such as Lambda, Glue job, or API call
  - example:
    ```json
    "RunGlueJob": {
      "Type": "Task"
    }
    ```
- **Choice State**
  - decision making, like an if/else branch
  - example: if file size > 0 → continue, else stop
- **Wait State**
  - pauses the workflow for a set time
- **Fail / Success State**
  - ends the workflow with failure or success

### 3. Transitions
Transitions define what runs next in the workflow.

Example:
`Step A → Step B → Step C`

## Example pipeline
Your current setup upgraded with orchestration:
1. S3 receives new data
2. Lambda triggers the workflow
3. Step Functions orchestrates the process
4. Glue ETL runs
5. Athena queries the transformed data
6. (Optional) QuickSight visualizes results

## Why Step Functions matters
- No manual work
- Automatic retries on failure
- Execution history and logs
- Visual debugging in the console

## Free tier
Step Functions has a free tier:
- 4,000 state transitions per month
- good for learning and small workloads

This makes Step Functions a safe and practical choice for building orchestrated AWS data workflows.


### Demo 1 — Quick start

Steps:
1. Upload a CSV file to S3
2. Create Glue ETL job(s) to process the file
3. Create a Step Functions state machine to run the ETL jobs


### Demo 2 — Parameterized ETL

Steps:
1. Upload a CSV file to S3
2. Create a parameterized Glue ETL job (accepts `--input_path`, `--output_path`)
3. Use a Step Functions state machine to pass parameters and invoke the job

Helpful: keep the Glue script and Step Function input as separate, copy-pasteable blocks.

Python Glue job (copy and paste into the Glue script editor):

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
spark = glueContext.spark_session
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

How to run the Glue job with parameters:
1. Open the Glue console → Jobs → Run job
2. Provide job parameters (example keys: `--input_path`, `--output_path`)

Step Functions state machine (copy and paste into the Step Functions editor):

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

Example execution input (copy and paste when starting the Step Function):

```json
{
  "input_path": "s3://your-bucket/input/users.csv",
  "output_path": "s3://your-bucket/output/"
}
```






### Demo 3 — Industry use case

Steps:
1. Upload CSV to S3
2. Trigger a Step Functions execution (via S3 event or Lambda)
3. Run a Glue Crawler to create or update the data catalog (DB + table)
4. Run a Glue ETL job to transform data and write results to S3
5. Invoke a Lambda to process transformed output
6. Insert processed records into DynamoDB

Final architecture (high-level)

```text
S3 (CSV upload)
  └─> Lambda (S3 event)
      └─> Step Functions
          ├─> Run Glue Crawler
          ├─> Run Glue ETL Job
          └─> Run Lambda (post-processing)
              └─> DynamoDB
```