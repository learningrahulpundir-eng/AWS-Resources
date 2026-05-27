# Amazon DynamoDB

Amazon DynamoDB is a fully managed NoSQL database service from AWS. It is designed for applications that require high performance, automatic scaling, and a flexible data model.

## Key benefits

- Serverless architecture: no hardware provisioning or maintenance.
- High performance: single-digit millisecond latency for reads and writes.
- Automatic scaling: capacity adjusts automatically with traffic.
- Flexible schema: each item can have different attributes.
- High availability and durability: data is replicated across availability zones.
- Security: supports encryption at rest and in transit, plus fine-grained IAM control.

## Core concepts

- **Table**: a collection of items.
- **Item**: a single record, similar to a row.
- **Attribute**: a field in an item, similar to a column.

### Example

Table: `Users`

Item:

```json
{ "UserID": 1, "Name": "Rahul", "Age": 21 }
```

## Keys and indexes

### Primary key

- **Partition key** (simple primary key): unique for each item and determines data distribution.
- **Composite key**: partition key + sort key. Supports sorted data and more flexible querying.

### Secondary indexes

- **Global Secondary Index (GSI)**: alternative partition key and sort key for different query patterns.
- **Local Secondary Index (LSI)**: same partition key but different sort key for additional local queries.

## How DynamoDB works

- Data is stored in tables.
- Each item is identified by its primary key.
- Data is partitioned and distributed automatically across storage nodes.
- Queries are most efficient when they use primary keys or indexes.

## Common use cases

- E-commerce shopping carts
- Gaming leaderboards
- Mobile app backends
- High-traffic web applications
- IoT data storage
- User sessions and profiles

## Pricing basics

DynamoDB pricing is pay-as-you-go, with two common modes:

- **On-Demand**: pay per request, ideal for variable or unpredictable workloads.
- **Provisioned**: reserve capacity ahead of time, often cheaper for stable workloads.

Costs generally include:

- Read and write requests
- Storage
- Backups and data transfer

## Advantages

- Fully managed service
- High scalability
- Low latency
- Built-in high availability
- Flexible schema
- Native AWS integration

## Disadvantages

- No SQL-style joins
- Limited support for complex ad hoc queries
- Requires careful data modeling
- Costs can grow with heavy traffic
- Maximum item size is 400 KB

## When to use DynamoDB

Use DynamoDB when you need massive scale, fast reads/writes, and a serverless database with automatic scaling.

Avoid DynamoDB for workloads that require complex relational queries, multi-table transactions, or large-scale joins.

## Demo ideas

### Demo 1: Basic table operations
- Create a DynamoDB table
- Add an item to the table
- Scan table data
- Query the table using the Query Editor

### Demo 2: Lambda integration
- Create a Lambda function
- Create an IAM role that allows Lambda to access DynamoDB
- Add logic to the Lambda function to insert a new item into the table
- Test the Lambda function
- Verify that the data was inserted into DynamoDB

### Demo 3: S3 to DynamoDB data import
- Upload an Excel sheet to S3
- Create a Lambda function to fetch data from S3
- Create a DynamoDB table
- Add logic to push the imported data into DynamoDB

### Example Lambda code to push item in Dynmo DB
```python
import json
import boto3
import uuid

# Initialize DynamoDB resource
dynamodb = boto3.resource('dynamodb')

# Replace with your table name
table = dynamodb.Table('Users')

def lambda_handler(event, context):
    try:
        # Create a sample item
        item = {
            'userId': str(uuid.uuid4()),  # unique ID
            'name': 'Rahul',
            'age': 25,
            'city': 'Agra'
        }

        # Insert into DynamoDB
        table.put_item(Item=item)

        return {
            'statusCode': 200,
            'body': json.dumps('Item inserted successfully!')
        }

    except Exception as e:
        return {
            'statusCode': 500,
            'body': json.dumps(f'Error: {str(e)}')
        }
```

### Example Lambda code for pull data from S3 and put into Dynamo DB
```python
import json
import boto3
import pandas as pd

s3 = boto3.client('s3')
dynamodb = boto3.resource('dynamodb')

# Replace with your values
BUCKET_NAME = 'my-data-bucket'
FILE_KEY = 'users.xlsx'
TABLE_NAME = 'Users'

def lambda_handler(event, context):
    try:
        # Download file from S3 to /tmp
        local_file = '/tmp/users.xlsx'
        s3.download_file(BUCKET_NAME, FILE_KEY, local_file)

        # Read Excel file
        df = pd.read_excel(local_file)

        # Connect to DynamoDB
        table = dynamodb.Table(TABLE_NAME)

        # Insert records
        for _, row in df.iterrows():
            item = {
                'userId': str(row['userId']),
                'name': str(row['name']),
                'age': int(row['age']),
                'city': str(row['city'])
            }

            table.put_item(Item=item)

        return {
            'statusCode': 200,
            'body': json.dumps('Data imported successfully!')
        }

    except Exception as e:
        return {
            'statusCode': 500,
            'body': json.dumps(str(e))
        }
```
