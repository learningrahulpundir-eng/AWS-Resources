# AWS Glue

## What is AWS Glue?
AWS Glue is a fully managed data integration service for ETL (extract, transform, load) workloads.

It helps you:
- Extract data from multiple sources
- Transform data by cleaning, filtering, and enriching it
- Load data into target stores such as Amazon S3, Amazon Redshift, and more

### Key uses
- Data warehousing
- Data lake preparation
- Analytics pipelines

### Main components
- **Glue Jobs**: ETL scripts, usually written in Python or Spark
- **Data Catalog**: central metadata repository
- **Triggers & Workflows**: schedule and manage ETL execution
- **Crawlers**: discover data structure and schema automatically

## What is an AWS Glue Crawler?
A Glue Crawler scans data sources and builds metadata in the Glue Data Catalog.

It can:
- Read data from Amazon S3, relational databases, and other supported sources
- Detect file formats such as CSV, JSON, Parquet, and more
- Infer schema details like columns and data types
- Create or update tables in the Glue Data Catalog

> A Glue Crawler is like a bot that inspects raw data and generates the metadata table for you.

## How a Glue Crawler works
1. Connect to a data source (for example, an S3 bucket)
2. Read sample data from the source
3. Infer the data schema
4. Create or update a table in the Glue Data Catalog
5. Make the table available for query tools

Tools that can use the cataloged data include:
- Amazon Athena
- Amazon Redshift Spectrum
- AWS Glue ETL jobs

## When to use a Glue Crawler
Use a Glue Crawler when:
- You have raw or semi-structured data
- The schema is unknown or changes frequently
- You want to quickly create tables for querying

## When not to use a Glue Crawler
Avoid a Glue Crawler when:
- You already know the schema and prefer manual table definition
- You need strict schema control
- The data structure is too complex or inconsistent for reliable inference

## Example workflow
1. Upload CSV files to Amazon S3
2. Run the Glue Crawler
3. Glue creates the table definition in the Data Catalog
4. Query the data using Athena, Redshift Spectrum, or Glue ETL jobs

---

## End-to-end demo: Glue Crawler → Data Catalog → Athena

### Step 1: Upload a CSV file to S3
1. Open the Amazon S3 console
2. Click **Create bucket**
3. Use a unique name, for example:
   - `glue-demo-bucket-<unique>`
4. Choose the same region as Glue and Athena
5. Open the bucket and click **Upload**
6. Upload your CSV file, for example `sales.csv`

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
3. Enter configuration:
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
3. Wait until status shows **Completed** (usually 1–2 minutes)

### Step 5: Verify the table in the Data Catalog
1. Open AWS Glue → **Data Catalog** → **Databases**
2. Select `demo_db`
3. Click **Tables**
4. Confirm a table appears, for example `demo_sales`
5. Open the table and verify:
   - Detected columns: `id`, `name`, `amount`, `city`
   - Inferred data types

### Step 6: Query data in Athena
1. Open the Amazon Athena console
2. Set the query result location if prompted:
   - `s3://glue-demo-bucket/query-results/`
3. In the Athena query editor, run:
```sql
USE demo_db;
SELECT * FROM demo_sales;
```

---

## Summary
AWS Glue automates ETL and metadata discovery. A Glue Crawler is useful when data schema is unknown or changing, and it simplifies table creation for analytics tools.
