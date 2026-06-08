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
