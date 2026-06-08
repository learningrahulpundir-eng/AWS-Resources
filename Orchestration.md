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


Demos-1:
1️⃣ Upload CSV → S3
2️⃣ Create ETL Jobs
3️⃣ Create Step Function to Run the ETL Jobs


Demo-2
1️⃣ Upload CSV → S3
2️⃣ Create Paramterized ETL Jobs
3️⃣ Run Parametrized Step Function to Run the ETL Jobs

Helpful for the Parameterized Script
import sys
from awsglue.transforms import *
from awsglue.utils import getResolvedOptions
from pyspark.context import SparkContext
from awsglue.context import GlueContext
from awsglue.job import Job

# ✅ Read parameters
args = getResolvedOptions(sys.argv, ['JOB_NAME', 'input_path', 'output_path'])

input_path = args['input_path']
output_path = args['output_path']

# ✅ Initialize
sc = SparkContext()
glueContext = GlueContext(sc)
spark = glueContext.spark_session
job = Job(glueContext)
job.init(args['JOB_NAME'], args)

# ✅ Read data from S3
datasource = glueContext.create_dynamic_frame.from_options(
    connection_type="s3",
    connection_options={"paths": [input_path]},
    format="csv",
    format_options={"withHeader": True}
)

# ✅ Apply FILTER (change condition as needed)
filtered_data = Filter.apply(
    frame=datasource,
    f=lambda row: row["age"] is not None and row["age"].isdigit() and int(row["age"]) > 25
)

# ✅ Write data to S3
glueContext.write_dynamic_frame.from_options(
    frame=filtered_data,
    connection_type="s3",
    connection_options={"path": output_path},
    format="csv"
)

print("ETL job with filter completed successfully")

job.commit()


## Run directly ETL Service with paramter
Go To Run with Paramters
Then Job Parmeters
Than add value of the paramter

## Run Step funciton with paramter
Below the sample script

`
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
`

During the running the step function

{
  "input_path": "s3://rahul11-demo111-bucket2017/input/users (2).csv",
  "output_path": "s3://rahul11-demo111-bucket2017/output/"
}






Demo-3 (Industry usecase)
1️⃣ Upload CSV → S3
2️⃣ Step Function starts
3️⃣ Run Glue Crawler → creates DB + table
4️⃣ Run Glue ETL → transform data → save to S3
5️⃣ Run Lambda → read processed data
6️⃣ Insert into DynamoDB

Final Architecture
S3 (CSV upload)
   ↓ (event)
Lambda Trigger
   ↓
Step Functions
   ├── Run Glue Crawler
   ├── Run Glue ETL Job
   ├── Run Lambda (process output)
   ↓
DynamoDB