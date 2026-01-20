
---

## **RDS and aurora**

RDS - Relational Database service a managed db service and allows to create the db in cloud.
This service supports many databases - 

1. Postgres
2. Mysql 
3. MariaDb
4. Aurora(Amazon propriotory)

RDS is a managed service and provides - 

- Scaling by deploying more read replicas.
- Monitoring dashboards
- Continous backup

But we can not ssh into our db service. Aurora is cloud optimised in context of AWS and gives over 5x performance boost. Storage increases automatically.

Aurora also comes with serverless tier where least management is required. Here we pay per second and is therefore more cost effective. Behind the schene there is going to be a proxy fleet which is managed by aurora. 

We can create snapshots of RDS and can be really handy if we want to use the snapshot elsewhere.

Read Replicas - We  can create upto 15 read replicas writing will be done only to mean RDs. This improves the performacne for read heavy workloads.

MultiAZ - They provide the high availability meaning if failover happens in a AZ then failover will be triggered by the AWS and failover db (secondry db) will takeover. Note that failover db is in different AZ and is not used untill main db is there. 

Finally we can have Multi-region deployments where read replicas are spread across multiple regions to have better read performance. 

## Elastic cache

There are inmemory database build on top of -

1. Redis
2. Memcache


## Dynamo db

Amazon **DynamoDB** is a **fully managed NoSQL database** service offered by AWS. It is designed for applications that need **consistent performance at any scale**, with **single-digit millisecond latency**, high availability, and the ability to handle **millions of requests per second**. Because DynamoDB is a managed service, AWS takes care of everything such as hardware provisioning, scaling, backups, replication, security, and patching—so you don’t have to manage servers.

DynamoDB stores data as **key-value pairs** or **documents**, making it ideal for modern applications like mobile apps, gaming, IoT platforms, serverless applications, and real-time systems. It is built to scale horizontally—meaning it can automatically distribute data and traffic across multiple servers/partitions.

DynamoDB organizes data into:

- **Tables** → like a collection
- **Items** → like rows
- **Attributes** → like fields/columns

Each item is uniquely identified by a **primary key**, which can be:

- **Partition key only** (simple key)
- **Partition key + Sort key** (composite key)

The partition key determines **where the item is stored**, while the sort key allows you to store multiple related items under the same partition.

- It is fully managed and serverless
- Massively scalable
- Low latency
- Global and fault tolerant


DynamoDB allows two modes:

**1. Provisioned Capacity**

You specify:
- Read Capacity Units (RCU)
- Write Capacity Units (WCU)

Best when workload is predictable.

**2. On-Demand Capacity**

No provisioning required. DynamoDB automatically scales up/down and charges per request.
Best for unpredictable traffic.

DAX - Dynamo db accelerator is fully managed in memory cache for dynamo db

DynamoDB is fundamentally built on **partitioning** — distributing data across multiple physical storage nodes to achieve **massive scalability** and **consistent performance**.  To partition data efficiently, DynamoDB relies on **keys**.

DynamoDB tables can be created with **two possible primary key designs**:

1. Simple primary key(Partiton key) - No two items in a table can have the same partition key.DynamoDB uses this key **only to decide which partition (node) stores the item**.
2. Composite primary key(Partition Key + Sort Key) 
	- Partition Key = Hash Key
	- Sort Key = Range Key
	Items can share the same partition key but need **unique sort keys**.

Note that it is serverless and flexible so even if data is stored in tables each row can have complelety diffrenet set of attributes. 

Global tables are the way by which we can make a dynamo db table accessible in multiple regions. When this happens each table is replaicated in every region and each region's replica is both readable/writable. 


### Redshift

Amazon Redshift is AWS’s **fully managed, petabyte-scale data warehouse** service designed for **analytics**, **BI workloads**, and **complex SQL queries** on very large datasets. It allows organizations to run massively parallel analytical queries across structured and semi-structured data at high speed and low cost.

Redshift is used for **analytics**, not transactional workloads.
Redshift stores data **column-wise**, not row-wise.  
This improves performance for analytical queries that scan specific columns rather than entire rows.

Redshift distributes data across multiple nodes in a **cluster**.  
Tasks are run in parallel, making large queries extremely fast.

There is also an option to go serverless with redshift. A new mode where you **don’t manage clusters**, only pay per second of compute used.

Redshift easily integrates with:

- **Amazon S3** (fastest ingestion via `COPY`)
- **AWS Glue**
- **Kinesis**
- **DynamoDB**
- **RDS / Aurora**
- **Lake Formation**
- **Data APIs**

## Amazon EMR

Amazon EMR (Elastic MapReduce) provides the managed solution by offering a fully scalable, cloud-native platform for running big data frameworks such as Apache Spark, Hadoop, Hive, Presto, HBase, Flink, and more. Instead of users setting up clusters themselves, EMR automatically provisions compute nodes, installs frameworks, optimizes them for performance, and integrates with AWS storage like S3. It lets you run distributed big-data jobs efficiently while only paying for the compute you use, with the ability to scale clusters dynamically or even run them serverless (EMR Serverless). In short, EMR removes the operational burden of maintaining big data infrastructure while giving you fast, flexible, and cost-effective processing for large-scale data analytics workloads.

### Amazon Athena

Serverless query tools to perform analysis against the S3 objects. 

### Amazon Quicksight 

Serverless ML powered bussinees intelligence to create interactive dashboards. 

### Document DB

It is an NoSql db campatible with mongodb and is counterpart with the RDS for Mongodb.
It is highly scalable, fully managed and highly available and can automatically grow. 

### Neptune

IT is fully managed graph db. Highly available across multiple AZ

### TimeStream

It is fully managed serverless time series database and it automatically scales up/down to adject based on capacity. Sotres and analyses trillions of events per day. 

### Amazon managed blockchain

It is managed service to join public blockchains or create own scalable private network. It is compatable with frameworks like hyperledger and and fabric. 

### Amazon Glue

Managed extract,transform and Load(ETL) service. and used for transformation of data. 

### DMS 

Database migration service fully managed db to migrate dbs. Origin db remains available through out and supports homogenous migration- Same tech to same 
Or Heterogenous migration - One tech to different




