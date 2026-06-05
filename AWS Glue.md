# AWS Glue

## What is AWS Glue?
AWS Glue is a fully managed data integration service for ETL (extract, transform, load) workloads.

### What it does
- Extracts data from multiple sources
- Cleans, transforms, and enriches data
- Loads data into targets such as Amazon S3 and Amazon Redshift

### Common use cases
- Data warehousing
- Data lake preparation
- Analytics pipelines

### Main components
- **Glue Jobs**: ETL scripts, often written in Python or Spark
- **Data Catalog**: central metadata repository
- **Triggers & Workflows**: schedule and manage ETL execution
- **Crawlers**: discover schema and create catalog tables automatically

## AWS Glue Crawler
A Glue Crawler scans data sources and builds metadata in the Glue Data Catalog.

### What it can do
- Read data from Amazon S3, relational databases, and other sources
- Detect formats such as CSV, JSON, and Parquet
- Infer schema details like columns and data types
- Create or update tables in the Glue Data Catalog

> A Glue Crawler is like a bot that inspects raw data and generates the metadata table for you.

### How it works
1. Connect to a data source (for example, an S3 bucket)
2. Read sample data
3. Infer the schema
4. Create or update a Data Catalog table
5. Make the table available for query tools

### Query tools that use the Data Catalog
- Amazon Athena
- Amazon Redshift Spectrum
- AWS Glue ETL jobs

## When to use a Glue Crawler
Use a Glue Crawler when:
- your data is raw or semi-structured
- the schema is unknown or changes frequently
- you want a fast way to create queryable tables

## When not to use a Glue Crawler
Avoid a Glue Crawler when:
- you already know the schema and prefer manual table definition
- you need strict schema control
- the data structure is too inconsistent for reliable inference

## Example pipeline
1. Upload CSV files to Amazon S3
2. Run the Glue Crawler
3. Glue creates the table definition in the Data Catalog
4. Query the data using Athena, Redshift Spectrum, or Glue ETL jobs

---

## End-to-end demo: Glue Crawler → Data Catalog → Athena

### Step 1: Upload a CSV file to S3
1. Open the Amazon S3 console
2. Click **Create bucket**
3. Use a unique name, for example `glue-demo-bucket-<unique>`
4. Choose the same region as Glue and Athena
5. Open the bucket and click **Upload**
6. Upload your `sales.csv` file

**Example CSV**
```csv
id,name,amount,city
1,Amit,1000,Delhi
2,Ravi,1500,Mumbai
3,John,2000,Bangalore
```

**Example S3 path**
`s3://glue-demo-bucket/sales.csv`

### Step 2: Create an IAM role for Glue
1. Open the IAM console
2. Go to **Roles** → **Create role**
3. Choose **Glue** as the service
4. Attach policies:
   - `AmazonS3FullAccess`
   - `AWSGlueServiceRole`
5. Name the role `AWSGlueDemoRole`
6. Complete role creation

### Step 3: Create the Glue Crawler
1. Open the AWS Glue console
2. Go to **Crawlers** → **Create crawler**
3. Configure the crawler:
   - **Crawler name**: `demo-crawler`
   - **Data source**: Amazon S3
   - **S3 path**: `s3://glue-demo-bucket/`
   - **IAM role**: `AWSGlueDemoRole`
   - **Schedule**: Run on demand
4. Choose or create a Glue database:
   - Database name: `demo_db`
5. Optional table prefix: `demo_`
6. Click **Create crawler**

### Step 4: Run the crawler
1. Select `demo-crawler`
2. Click **Run crawler**
3. Wait for the status to show **Completed** (usually 1–2 minutes)

### Step 5: Verify the table in the Data Catalog
1. Open AWS Glue → **Data Catalog** → **Databases**
2. Select `demo_db`
3. Click **Tables**
4. Confirm a table appears, such as `demo_sales`
5. Verify the detected columns and data types:
   - `id`, `name`, `amount`, `city`

### Step 6: Query the data in Athena
1. Open the Amazon Athena console
2. Set the query result location if prompted:
   - `s3://glue-demo-bucket/query-results/`
3. Run:
```sql
USE demo_db;
SELECT * FROM demo_sales;
```

---

## Summary
AWS Glue simplifies ETL and metadata discovery. A Glue Crawler is especially useful when the schema is unknown or changing, because it automates table creation for analytics tools.

## Architecture overview
Raw Data (CSV)
   ↓
S3 Bucket
   ↓
Glue Crawler
   ↓
Data Catalog Table
   ↓
Glue ETL Job (Transform & Clean)
   ↓
S3 (Processed Data - Parquet)
   ↓
Glue Crawler (Optional)
   ↓
Athena Query