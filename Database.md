# Amazon RDS

## What is RDS?

RDS stands for **Relational Database Service**.
It is an AWS service for running relational databases without managing the underlying servers.

Relational databases organize data into:
- Tables
- Rows
- Columns

Example table:

| ID | Name    | City   |
| -- | ------- | ------ |
| 1  | Santosh | Delhi  |
| 2  | Sitaram | Noida  |

---

## Why RDS Exists

Managing a database on your own server is complex.
Without RDS, you must handle:
- Database server setup
- Operating system maintenance
- Patching and updates
- Backups and recovery
- Monitoring and performance tuning
- Replication and scaling
- High availability
- Security

Example manual setup:
- Launch an EC2 instance
- Install MySQL, PostgreSQL, or Oracle
- Manage the server and database yourself

---

## Managed Service Benefits

Amazon RDS is a **managed database service**.
AWS takes care of most operational tasks so developers and DBAs can focus on application data.

AWS manages:
- Infrastructure and hardware
- OS maintenance
- Database installation and patching
- Monitoring
- Automated backups
- Failover

You manage:
- Database schema
- Data
- Queries
- Users and permissions

---

## Supported Engines

Amazon RDS supports:
- MySQL
- PostgreSQL
- MariaDB
- Oracle
- Microsoft SQL Server
- Amazon Aurora

---

## RDS as PaaS

RDS is a **Platform as a Service (PaaS)**.

AWS handles:
- Server provisioning
- Storage and networking
- Maintenance and updates

You handle:
- Database usage
- Data modeling
- Query design
- Access control

---

## Fully Managed Features

RDS automatically provides:
- Database installation
- OS management
- Software updates and patching
- Maintenance
- Monitoring

---

## Automated Backups

RDS creates automated backups and transaction logs without manual effort.

**Transaction logs** capture database operations such as:
- `INSERT`
- `UPDATE`
- `DELETE`

Example:

```sql
INSERT INTO employees VALUES (1, 'Anas');
UPDATE employees SET name = 'Mohd Anas' WHERE id = 1;
DELETE FROM employees WHERE id = 1;
```

These logs enable recovery to a specific point in time.

---

## Point-in-Time Recovery (PITR)

PITR allows restoring a database to an exact timestamp.

For example:
- Database is corrupted at 2:30 PM
- Restore to 2:25 PM

RDS uses automated backups plus transaction logs for recovery.

---

## Manual Snapshots

Manual snapshots are user-created backups.
They are:
- Full backups
- Kept until manually deleted
- Stored in Amazon S3 internally by AWS

Use cases:
- Migration to another region or account
- Cloning a database
- Long-term backup retention

---

## Multi-AZ Deployment

Multi-AZ is designed for:
- High availability
- Disaster recovery
- Automatic failover

### Architecture

Primary database (AZ1):
- Accepts reads and writes
- Uses synchronous replication

Standby database (AZ2):
- Passive copy
- Not directly accessible
- Becomes active only on failure

### Important points

Primary instance:
- Handles both read and write traffic

Standby instance:
- Remains passive until failover

Replication:
- Synchronous between AZs

 Data written to both AZs simultaneously

---

 If Primary Fails

RDS automatically performs:

 Failover
 Standby becomes new primary

Application reconnects automatically using same endpoint.

---

 ------------------- Read Replicas -------------------

Read Replicas are different from Multi-AZ.

Purpose:

 Improve read performance
 Scale read-heavy applications

---

 Read Replica Characteristics

 Read-only copy
 Uses asynchronous replication
 Can be in same or different region

---

 Example Use Case

Primary DB handles:

 INSERT
 UPDATE
 DELETE

Read Replica handles:

 SELECT queries
 Reports
 Analytics

---

 ------------------- Manual Snapshot -------------------

 Characteristics

 Full backup
 Permanent
 User controlled

---

 ------------------------ Security ------------------------

Amazon RDS supports multiple security layers.

---

 IAM (Identity and Access Management)

Controls:

 Who can create RDS
 Who can modify RDS
 Permissions

---

 VPC (Virtual Private Cloud)

RDS runs inside VPC.

Controls:

 Network isolation
 Private communication

---

 Private/Public Subnet

 Private Subnet

 No direct internet access
 Recommended for databases

 Public Subnet

 Internet accessible
 Usually avoided for production DBs

---

 KMS (Key Management Service)

Used for encryption.

Encrypts:

 Storage
 Snapshots
 Backups

---

 Encryption Example


Original Data:
password1234

Encrypted Data:
xcjskjsheyeyfd


---

 ------------------------ Scaling ------------------------

Scaling means increasing database capacity.

---

 Vertical Scaling

Increase power of same server.

Example:

 More CPU
 More RAM
 More Storage
 Higher IOPS

---

 Example


db.t3.medium  ---> db.r6g.large

---

 Horizontal Scaling

Add more servers.

----

Example:

 Read Replicas
 Distributed systems

Used for:

 Heavy read traffic
 Large applications

---

 ------------------------ Amazon Aurora ------------------------

Amazon Aurora is AWS's cloud-native database.

Compatible with:

 MySQL
 PostgreSQL

Applications can often migrate with minimal changes.

---

 Why Aurora?

Traditional databases were originally designed for:

 On-premises infrastructure
 Physical servers

Aurora is designed specifically for cloud infrastructure.

---

 Benefits of Aurora

 Better performance
 Faster failover
 High scalability
 Distributed storage
 High availability
 Automatic replication
 Shared storage architecture

---

 ------------------------ Aurora Architecture ------------------------

Aurora separates:

 Compute Layer
 Storage Layer

---

 Compute Layer

Handles:

 SQL Queries
 Query Processing
 Connections
 Transactions

Example:


SELECT  FROM employees;


Aurora compute instances process the query.

---

 Storage Layer

Stores actual database data.

Includes:

 Tables
 Records
 Transaction logs
 Indexes

---

 Aurora Distributed Storage

Aurora storage automatically spreads across:

 3 Availability Zones

Aurora keeps:

 6 copies of data
 2 copies per AZ

---

This improves:

 Durability
 Fault tolerance
 Availability

---

 ------------------------ Traditional DB Architecture ------------------------

Traditional databases:

Each database server has its own storage.

Example:


Primary DB  ---> Own Storage
Replica DB  ---> Separate Copied Storage


---

 Problems

 Replication Lag

Replica takes time to sync.

 Slower Failover

Replica may not be fully updated.

 Storage Duplication

Each replica stores separate copy.

---

 ------------------------ Aurora Architecture Advantage ------------------------

Aurora replicas use the SAME shared storage layer.

Example:


Writer Instance
        |
Shared Distributed Storage
        |
Reader Instances


---

 Important Benefit

No need to copy entire database storage separately.

All instances access same distributed storage.

---

 Faster Failover

If writer crashes:


Reader Instance ---> Promoted to Writer


Because replicas already use same storage:

 Failover is much faster
 Minimal replication lag

---

 Aurora = Centralized Distributed Storage Layer

Aurora storage is:

 Shared
 Distributed
 Fault tolerant
 Auto replicated

---

 ------------------------ Connecting to RDS ------------------------

Example MySQL connection command:


mysql -h rds-demo-db.czmqew8okjst.ap-south-1.rds.amazonaws.com -u admin -p


---

 Breakdown

 mysql

MySQL client command

 -h

Host endpoint of RDS

 -u

Username

 -p

Prompt for password

---

 ------------------------ Summary ------------------------

 Amazon RDS

 Managed relational database service
 Supports multiple DB engines
 Automated backups
 Multi-AZ HA
 Read Replicas
 Secure and scalable

---

 Amazon Aurora

 AWS cloud-native database
 MySQL/PostgreSQL compatible
 Shared distributed storage
 Faster failover
 Better performance
 High availability
 6-way replicated storage across 3 AZs

--------------------------------------------------------
