# Amazon Redshift

Amazon Redshift is a fully managed, cloud-based data warehouse service from AWS.

## Quick idea

- Athena = query data directly from S3
- Redshift = store and analyze large structured datasets with high performance

## What Redshift is used for

Redshift is used to:

- store large amounts of structured data
- run fast SQL analytics queries
- support business intelligence and reporting

---

## Redshift architecture

### 1. Cluster
A Redshift cluster is the main unit of the service.

It usually contains:

- ✅ Leader node
- ✅ Compute nodes

### 2. Leader node
The leader node:

- receives SQL queries
- creates the execution plan
- distributes work to compute nodes

### 3. Compute nodes
Compute nodes:

- store the data
- execute queries in parallel

This model is called Massive Parallel Processing (MPP).

---

## Key features

### Columnar storage
- stores data column-by-column
- improves analytics performance
- reduces disk I/O

### Massively Parallel Processing (MPP)
- splits queries across nodes
- runs tasks in parallel
- delivers very fast performance

### Compression
- automatically compresses data
- saves storage
- improves query speed

### SQL support
- uses PostgreSQL-like SQL syntax
- easy for SQL users to adopt

### Integration
Redshift works well with:

- Amazon S3
- AWS Glue
- Amazon Athena
- Amazon QuickSight
- BI tools such as Tableau and Power BI

---

## Redshift data flow

1. Data comes from S3 or other databases such as RDS and DynamoDB
2. Data is loaded into Redshift using the COPY command
3. Queries are run using SQL
4. Results are used in BI tools or dashboards

---

## Redshift Spectrum

Redshift Spectrum lets you query data directly from S3 without loading it into Redshift.

This is useful when you want to:

- keep data in S3
- query it on demand
- combine Redshift tables with S3 data

---

## Storage types

### 1. Dense Compute (DC)
- fast CPUs
- less storage
- more expensive

### 2. Dense Storage (DS)
- large storage capacity
- slower than DC

### 3. RA3 (modern choice)
- best option for most use cases
- separates compute and storage
- uses managed storage

---

## Pricing model

You pay for:

- compute nodes
- storage (especially RA3 managed storage)

Options:

- On-demand
- Reserved instances (usually cheaper)

---

## When to use Redshift

Use Redshift when:

- you need fast repeated analytics queries
- your data is structured
- you want a data warehouse solution

## When not to use Redshift

Avoid Redshift when:

- you want a serverless solution → use Athena
- your dataset is small → use RDS
- you need real-time transaction processing → use an OLTP database

---

## Demo: Amazon Redshift (step by step)

### Goal
Create a Redshift cluster, load data, run SQL queries, and visualize the result.

### Step 1: Log in to AWS

1. Open the AWS Console
2. Search for Redshift
3. Open Amazon Redshift

### Step 2: Create a Redshift cluster

1. Click Create cluster
2. Fill in the basic details:
   - Cluster identifier: demo-redshift-cluster
   - Admin username: admin
   - Admin password: use a strong password
3. Choose configuration:
   - Node type: ra3.xlplus (or dc2.large if needed)
   - Number of nodes: 1 for demo
4. Set networking:
   - choose the default VPC
   - enable Publicly accessible = Yes
5. Add permissions:
   - attach an IAM role for S3 access
6. Click Create cluster

Wait 5–10 minutes until the cluster status becomes Available.

### Step 3: Connect to Redshift Query Editor

1. Open Query Editor v2
2. Click Connect
3. Enter:
   - Cluster: your cluster name
   - Database: dev
   - Username: admin
   - Password: your password
4. Click Connect

### Step 4: Create a sample table

Run this SQL:

```sql
CREATE TABLE sales (
  id INT,
  product VARCHAR(50),
  amount INT
);
```

### Step 5: Insert sample data

```sql
INSERT INTO sales (id, product, amount)
VALUES
  (1, 'Laptop', 50000),
  (2, 'Mobile', 20000),
  (3, 'Tablet', 15000),
  (4, 'Laptop', 70000);
```

### Step 6: Query the data

Basic query:

```sql
SELECT * FROM sales;
```

Aggregation query:

```sql
SELECT
  product,
  SUM(amount) AS total_sales
FROM sales
GROUP BY product
ORDER BY total_sales DESC;
```

Why this matters:

- Laptop has the highest sales
- this shows Redshift’s analytics strength

### Step 7: Load data from S3

1. Upload a CSV file to S3, for example `sales.csv`

```csv
5,TV,40000
6,Headphones,5000
```

2. Run the COPY command:

```sql
COPY sales
FROM 's3://your-bucket-name/sales.csv'
IAM_ROLE 'your-iam-role-arn'
CSV;
```

This is a very important interview and demo concept.

### Step 8: Query the combined data

```sql
SELECT * FROM sales;
```

You can explain that the new data from S3 is now part of the same table.

### Step 9: Performance demo

Explain that Redshift distributes data across nodes and runs queries in parallel, which makes large analytics workloads much faster.

### Step 10: Optional – Redshift Spectrum

You can also show that data can be queried directly from S3 without loading it first:

```sql
SELECT *
FROM spectrum_schema.sales_ext;
```

### Step 11: Clean up

To avoid charges:

1. Go to the Redshift cluster
2. Click Delete
3. Disable snapshot for the demo
4. Confirm deletion

