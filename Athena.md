# AWS Athena — Complete Overview

Amazon Athena is a serverless, interactive query service that lets you analyze data directly in Amazon S3 using SQL. There is no need to manage infrastructure.

## Key Features

### 1. Serverless Architecture
- No servers to provision, patch, or manage
- Automatically scales with query load
- Pay only for the data scanned per query

### 2. SQL-Based Querying
- Uses standard SQL via Presto / Trino engine
- Easy for analysts and developers familiar with SQL

### 3. Works Directly with S3
Query data stored in Amazon S3, including:
- CSV
- JSON
- Parquet
- ORC
- Avro

### 4. Schema-On-Read
- No need to load data into a database first
- Define the schema when querying via AWS Glue or manually

### 5. Integration with AWS Services
- AWS Glue (Data Catalog)
- Amazon QuickSight (visualization)
- AWS Lambda
- Amazon SageMaker
- AWS Lake Formation

## How Athena Works
1. Store data in Amazon S3
2. Define schema using the AWS Glue Data Catalog
3. Run queries in the Athena SQL editor or via API
4. Query results are written back to S3

## Conceptual Architecture
- User → Athena Query Engine → Data Catalog (Glue)
- Athena reads data from S3
- Query results are stored in S3

## Pricing Model
Athena pricing is based on the amount of data scanned per query.
- Typical cost: about $5 per TB scanned (varies by region)

### Cost optimization tips
- Use columnar formats like Parquet or ORC
- Partition data by common query keys
- Compress files

## Supported Data Formats

| Format   | Use Case                         |
|----------|----------------------------------|
| CSV      | Simple structured data           |
| JSON     | Semi-structured logs/events      |
| Parquet  | Columnar, optimized analytics    |
| ORC      | High-performance query workloads |
| Avro     | Schema evolution use cases       |

## Key Concepts

### Database & Tables
- Logical metadata constructs in Athena
- Tables reference data stored in S3

### Partitions
- Break data into smaller slices, such as by date
- Improves query performance and lowers cost

### AWS Glue Data Catalog
- Stores table and schema metadata
- Acts as Athena's metadata repository

### Workgroups
- Separate workloads
- Control cost and access policies

### Query Result Location
- Athena requires an S3 bucket for query output
- Results are saved automatically after each query

## Demo: Query Data with Athena
This demo shows how to upload a sample CSV, create a table, run queries, and inspect results.

### Step 1: Create & Upload a CSV File to S3
1. Create a file named `employees.csv` with sample data:

```csv
id,name,department,salary
1,Amit,IT,50000
2,Riya,HR,40000
3,John,Finance,60000
4,Sara,IT,55000
```

2. Upload the file to S3:
- Open the AWS Console
- Go to Amazon S3
- Create a bucket (example: `athena-demo-bucket-xyz`)
- Upload `employees.csv` to the bucket

### Step 2: Set Up Athena
1. Open Athena in the AWS Console
2. Go to the Query Editor
3. Set the query result location to an S3 folder, for example:

```text
s3://athena-demo-bucket-xyz/results/
```

### Step 3: Create a Database and Table
Run these queries in Athena:

```sql
CREATE DATABASE demo_db;
USE demo_db;
```

Create the external table:

```sql
CREATE EXTERNAL TABLE employees (
  id INT,
  name STRING,
  department STRING,
  salary INT
)
ROW FORMAT DELIMITED
FIELDS TERMINATED BY ','
STORED AS TEXTFILE
LOCATION 's3://athena-demo-bucket-xyz/';
```

> Replace the bucket name if you are using a different S3 bucket.

### Step 4: Run Simple Queries

#### View all rows
```sql
SELECT *
FROM employees;
```

#### Filter by department
```sql
SELECT name, salary
FROM employees
WHERE department = 'IT';
```

#### Aggregate by department
```sql
SELECT department,
       AVG(salary) AS avg_salary
FROM employees
GROUP BY department;
```

#### Sort by salary
```sql
SELECT *
FROM employees
ORDER BY salary DESC;
```

### Step 5: Check Results
- Query results appear in the Athena console
- Results are also saved automatically to the configured S3 location
- You can download results as CSV or view query history
