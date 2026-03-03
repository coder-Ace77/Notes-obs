
---

Amazon DynamoDB is a fully managed, serverless, **NoSQL database service** designed by AWS to provide consistent, single-digit millisecond latency at any scale. DynamoDB is built on a **key-value and document data model**. It is inherently distributed, automatically replicating data across multiple Availability Zones (AZs) within an AWS Region to ensure high availability and durability.

DynamoDB organizes data into **Tables**, **Items**, and **Attributes**. A Table is a collection of Items (similar to rows), and each Item is a collection of Attributes (similar to columns). While the table is schema-less regarding non-key attributes, it requires a strictly defined Primary Key.
### Primary key , partition key and sort key

The primary key can be partition key or combination of partition key and sort key (composite primary key). The Partition Key is used by DynamoDB's internal hashing algorithm to distribute data across physical storage nodes, while the Sort Key allows for complex querying and ordering of data within the same partition. 

For example, if you're building a simple group chat application, it would make sense to use the chat_id as the partition key and message_id as the sort key. This way, you can efficiently query all messages for a specific chat group and sort them chronologically before displaying them to users.

**Hash Partitioning for Partition Keys:** The physical location of the data is determined by hashing the partition key. A request router consults a partition metadata service to map the hashed key to the correct storage node.

**B-trees for Sort Keys:** Within each partition, DynamoDB organizes items in a B-tree data structure indexed by the sort key. This enables efficient range queries and sorted retrieval of data within a partition.

**Composite Key Operations:** When querying with both keys, DynamoDB first uses the partition key's hash to find the right node, then uses the sort key to traverse the B-tree and find the specific items.

But what if you need to query your data by an attribute that isn't the partition key? This is where secondary indexes come in. DynamoDB supports two types of secondary indexes:

**Global Secondary Index (GSI)** - An index with a partition key and optional sort key that differs from the table's partition key. GSIs allow you to query items based on attributes other than the table's partition key.  A GSI is essentially a "shadow table" that stays in sync with your base table but can have an entirely different Partition Key and Sort Key. You can create, modify, or delete GSIs at any time.

**Local Secondary Indexes (LSI)** allow you to specify a different sort key for the same partition key, The Partition Key **must** be the same as the base table. Only the Sort Key changes. It must be created at the time of table creation. 

Finally remember that `GSI` has both partiton key and a sort key. While `LSI` only has sort key. In GSI again the `partition key` is used to find the node and `sort key` defines the index on `B+tree`. 


#### Working with indexes

Since we are working in NOSQL paradigm we can't just join datas. Instead usual pattern is to create alternatve view of data by secondry indexes. When we create an index we not need to copy data from base table to index. We can use projection types - 

- KEYS only - Only the Partition Key and Sort Key of the index (plus the base table keys) are stored. This is the cheapest and smallest option.
- **INCLUDE:** You specify a list of non-key attributes you want to be "projected" into the index.
- **ALL:** The entire item is copied. This is the most expensive but prevents "extra hops" to the base table.

So what an index creates is an additional table with the given partition and sort key. This is basically efficient for querying. Note that if we query for the attribute using secondry index but we do not have the attribute in projection then dynamo db will fetch the attribute from base table which is much slower. 

Now `GSI's`  are very good and ideal for indexing the data for searching using some other attributes. For example we might be wanting to get the `orders` so it makes sence to use `customer_id` as partition key as customer will be only seeing its data and being on the same physical device makes it works very fast. Second we can use `timestamp` as sort index reason being we will be required to show all the orders of customer by timestamp only. However suppose users can also query by price range then we can simply use price as another `LSI` local secondry index. 

Sometimes there are also the schenarios where we create `sparse GSI`. This is pattern usually with the status fields. For example we may be interested in the users with `admin` and `moderators` of some channel so we can directly use this GSI. 



### Consistency

DynamoDB supports two types of read consistency: **Eventually Consistent Reads** (maximizing throughput) and **Strongly Consistent Reads** (ensuring you get the most recent write). It also supports **ACID Transactions**, which are critical for complex workflows requiring "all-or-nothing" operations across multiple items or tables.  
More importantly this is not a table level configuration but a read level , you choose the consistency model on each individual read request by setting ConsistentRead=true in your GetItem, Query, or Scan calls.

Every read is eventually consistent unless you explicitly request otherwise. This provides the highest availability and lowest latency, but you might not see the most recent write immediately. DynamoDB generally behaves as an AP system with BASE properties.

When you set ConsistentRead=true, DynamoDB ensures the read reflects all successful writes that occurred before the read. This costs twice the read capacity (1 RCU per 4KB instead of 0.5) and may have slightly higher latency, but guarantees you see the latest data.

DynamoDB also supports ACID transactions via TransactWriteItems and TransactGetItems, which provide serializable isolation across up to 100 items spanning multiple tables.

Strong consistent reads are only supported on the base table and Local Secondary Indexes (LSIs). Global Secondary Indexes (GSIs) only support eventually consistent reads, so keep this in mind when designing access patterns that require strong consistency.

Working of consistency is pretty simple. Dynamodb maintains 3 read replicas for eah partition with one leader. Write is written directly to the leader which asyncronousy replicates it to the read replicas. So a single read may not be latest. In eventual consistency the read request can be returned from any one of the read replica. 

In consistent mode the read request is forwarded directly to the leader and read from there this means lower throughput but consistent. 


### Working with dynamodb

We've already touched on this a bit, but let's dive deeper into how you can access data in DynamoDB. There are two primary ways to access data in DynamoDB: Scan and Query operations.

- Scan operations - In this mode we read every item in the table and then returns the paginated response. It should be avoided mostly. 
- Query operations - In this mode data is retrieved using the primary or secondry key. Queries are more efficient than scans, as they only read items that match the specified key conditions. Queries can also be used to perform range queries on the sort key.

Unlike traditional SQL databases, DynamoDB's primary interface is through the AWS SDK or the AWS console rather than a standalone query language. 

Example of query. 

```js
const params = {
  TableName: 'users',
  KeyConditionExpression: 'user_id = :id',
  ExpressionAttributeValues: {
    ':id': 101
  }
};

dynamodb.query(params, (err, data) => {
  if (err) console.error(err);
  else console.log(data);
});
```

Example of scan

```js
const params = {
  TableName: 'users'
};

dynamodb.scan(params, (err, data) => {
  if (err) console.error(err);
  else console.log(data);
});
```

### Scaling

DynamoDB scales through auto-sharding and load balancing. When a partition reaches capacity (in size or throughput), DynamoDB automatically splits it and redistributes data. Hash-based partitioning ensures even distribution across nodes, balancing traffic and load.

DynamoDB is designed to provide high availability and fault tolerance through its distributed architecture and data replication mechanisms. The service automatically replicates data across multiple Availability Zones within a region, so that data is durable and accessible even in the event of hardware failures or network disruptions.

DynamoDB automatically replicates your data across three Availability Zones within a region -- this is not user-configurable. Each partition maintains three replicas (one leader and two followers) managed entirely by AWS. For cross-region replication, you can enable Global Tables to add replicas in additional AWS regions.

Under the hood, each partition uses Multi-Paxos consensus with a leader-based replication group of three nodes. The leader handles all writes: it generates a write-ahead log (WAL) entry and sends it to its peers, and the write is acknowledged once a quorum (2 of 3) persists the log record. For strongly consistent reads, DynamoDB routes the request directly to the leader, which always has the most up-to-date data. For eventually consistent reads, any of the three replicas can serve the request, which provides lower latency but might return slightly stale data.

Data is encrypted at rest by default in DynamoDB, so your data is secure even when it's not being accessed. DynamoDB also enforces TLS for all API calls, so data in transit is always encrypted -- there's no separate configuration needed.

### Pricing 

Dynamo db is priced based on throughput of read per second and write per second. Each DynamoDB partition supports up to 3,000 read capacity units and 1,000 write capacity units. This means a single partition can handle 12MB of reads per second (3,000 × 4KB) and 1MB of writes per second (1,000 × 1KB). 

There are two pricing models for DynamoDB: on-demand and provisioned capacity. On-demand pricing charges per request, making it suitable for unpredictable workloads. Provisioned capacity, on the other hand, requires users to specify read and write capacity units, which are billed hourly. This model is more cost-effective for predictable workloads but may result in underutilized capacity during low-traffic periods.

Pricing is based on, what Amazon calls, read and write capacity units. These units are a measure of the throughput you need for your DynamoDB table. You can think of them as a measure of how much data you can read or write per second. Read capacity units(RCU) is about 1 dollar per million requests while write capacity is around 5 dollars per million requests. 

For example, if you were planning on storing YouTube views in DynamoDB, each write (regardless of how small) consumes at least 1 WCU since DynamoDB rounds up to the nearest 1KB. With 1,000 WCU per partition, a single partition supports about 1,000 writes per second. If you expect 10,000,000 views per second, you'd need roughly 10,000 partitions. Using provisioned capacity pricing (~$0.00065 per WCU-hour in us-east-1), that's 10,000,000 WCU × $0.00065/hour × 24 hours ≈ $156,000 per day. On-demand pricing would be significantly higher. These numbers can help you gut check whether your application will be able to handle the expected load without incurring unrealistic costs.

### Advanced features

#### DAX (Dynamo db acclerator)

DAX is a caching service designed to enhance DynamoDB performance by delivering microsecond response times for read-heavy workloads. Using DAX requires swapping your DynamoDB client for the DAX client SDK (available for Java, .NET, Node.js, Python, Go) -- the API is compatible, so the changes are minimal, but it's not completely transparent. DAX operates as both a read-through and write-through cache: it caches read results and delivers them directly to applications, and writes data to both the cache and DynamoDB. An important nuance is that DAX only auto-invalidates cached items for writes that go through DAX itself.

DAX maintains two caches: an item cache (for GetItem/BatchGetItem results) and a query cache (for Query and Scan results). Both are always active -- there's no configuration to choose one over the other. One important caveat: DAX does not cache strongly consistent reads. When you request a strongly consistent read through DAX, it passes the request directly to DynamoDB and returns the result without caching it.

### Streams

Dynamo also has built-in support for Change Data Capture (CDC) through DynamoDB Streams. Streams capture changes to items in a table and make them available for processing in real-time. Any change event in a table, such as an insert, update, or delete operation, is recorded in the stream as a stream record to be consumed by downstream applications.

This can be used for a variety of use cases, such as triggering Lambda functions in response to changes in the database, maintaining a replica of the database in another system, or building real-time analytics applications.