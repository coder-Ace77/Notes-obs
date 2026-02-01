
---


There are many ways to find the performance of postgres db one good utility is pgbench

```bash
pgbench -c 50 -j 8 -T 60 -h localhost -p 5432 -U admin -f search.sql mydb
```

This command is used to figuare out the throguhput and performance of db layer.
`pgbench` is a benchmarking tool that comes with PostgreSQL. It allows you to test database performance by simulating multiple clients running transactions.

-c means how many numbers of users will connect to the client.
-j define worker threads basically 50 clients will be managed through 8 threads
-T duration of test in seconds
-h host of psql server
-p port 
-U defies which postgres user will be used to connect to db
-f means custom file will be run. 
final arg is the name of db

## Client request flow:

Everything starts when a client connects to PostgreSQL:

- PostgreSQL uses a **process-per-connection model**. Each client connection is handled by a dedicated **backend process** (`postgres`), unlike MySQL’s thread-based approach.
- Once connected, the client can send SQL statements or transactions
- Once req is reached it is converted into a parse tree
- Query is rewritten at this stage using rules
- Planner converts the query into one or more query plans. 
	- Planner will consider multiple stragtegies and chooses one with the lowest cost.
- Each plan is composed of nodes
	-  **Scan Nodes:** TableScan, IndexScan
	- **Join Nodes:** Nested Loop, Hash Join
	- **Aggregate Nodes:** SUM, COUNT, AVG
	- **Sort Nodes:** ORDER BY
- Executer stage: Each plan is a tree and executor walks the tree recursively. Execution is **pull-based**: parent requests rows from children, not the other way around.PostgreSQL supports **pipelined execution**, so it doesn’t materialize the entire result in memory unless necessary (e.g., `ORDER BY` or `DISTINCT`).

- Now postgres reads data using buffer manager. If a page isn’t in memory, a **disk read** occurs. Writes follow **write-ahead logging (WAL)** for durability. Sequential scans often read in **8KB pages**, and the buffer pool can be tuned with `shared_buffers`
- PostgreSQL ensures ACID compliance using **MVCC (Multi-Version Concurrency Control)**:
- Data may be **formatted** according to the client’s requested format (text, binary, JSON, etc.).

#### Optimisation which postgres does:

PostgreSQL can split scans across multiple processes for large queries called (parallel_seq_scan).If all requested columns exist in an index, PostgreSQL can avoid reading the table entirely.

## Indexes:

At its core, an **index** is a _data structure_ that allows PostgreSQL to find rows faster, without scanning the entire table.

Without an index: PostgreSQL performs a **sequential scan** — it reads every row to check if it matches the `WHERE` clause.With an index: PostgreSQL can **jump directly** to relevant rows using a tree or hash lookup.

```sql
CREATE INDEX idx_salary ON employees (salary);
```

Builds a **separate data structure** stored on disk (in `pg_index` and related files).Keeps the index in sync with table changes (`INSERT`, `UPDATE`, `DELETE`).

Indexes are _not_ stored inside the table file — they’re separate physical files under the database’s directory in the data folder.

### Type of indexes: 

#### B-tree:(default) It is kind of Balanced tree but multilevel. 
Its best use cases are:
	-  Equality and range queries.
	- Sorting operations (`ORDER BY`).
	- Unique constraints (e.g., primary keys).

B-trees maintain physical order of keys → scanning a range (`BETWEEN`, `ORDER BY`) can read sequential disk pages, improving I/O locality.

B-tree are really good for getting overall `O(logn)` complexity. 

#### Hash index:  Hash table that maps key → row pointer.

```sql
CREATE INDEX idx_user_hash ON users USING hash (email);
SELECT * FROM users WHERE email = 'john@abc.com';
```

hash indexes are not good for order by or range queries. They are good for exact queries.

#### GIN (generalized inverted index):

It’s called “inverted” because it maps **values → rows** (the inverse of a B-tree which maps rows → values).
So while a **B-tree** stores sorted key-value pairs like:
```psql
key → row location
```

a **GIN** index stores:
```psql
element/value → list of row locations containing that element
```

So suppose we have data as:

|id|description|
|---|---|
|1|"apple orange banana"|
|2|"orange mango"|
|3|"banana apple"|
Gin will break them as and index each word 

```
"apple"  → [1, 3]
"banana" → [1, 3]
"orange" → [1, 2]
"mango"  → [2]
```

so for example with the following query

```sql
SELECT * FROM products WHERE description @@ to_tsquery('apple & banana');
```

A GIN index has:

1. **Entries (keys):** distinct lexemes (e.g., words or tokens)
2. **Posting lists:** sorted lists of TIDs (tuple identifiers) — basically row pointers

Internally, it uses a **B-tree for the entries**, but each entry maps to **a set of rows** instead of one.

Good for:
- Full-text search (`tsvector` columns).
- Arrays (`@>`, `<@`, `&&`).
- JSONB containment queries.

GIN provides extremenly fast full test search.

First create index
```sql
CREATE INDEX idx_products_desc_gin ON products USING GIN (to_tsvector('english', description));
```

then we can search 

```sql
SELECT * FROM products WHERE to_tsvector('english', description) @@ plainto_tsquery('banana apple');
```

#### Trigram search(good for fuzzy search and substring):

```sql
CREATE EXTENSION pg_trgm;
CREATE INDEX idx_products_desc_trgm ON products USING GIN (description gin_trgm_ops);

SELECT * FROM products WHERE description ILIKE '%banana%';
SELECT * FROM products WHERE description % 'bananna'; -- fuzzy
```

A **trigram** is simply a **sequence of three consecutive characters** taken from a string. spaces are addes to the start and end so

```
String:  "apple"
Trigrams: "  a", " ap", "app", "ppl", "ple", "le ", "e  "
```

When you search for a string — e.g., `"aple"` (a typo of `"apple"`) — PostgreSQL compares the **overlap between trigrams** of both strings.

```
'apple' trigrams:  { ' ap', 'app', 'ppl', 'ple' }
'aple'  trigrams:  { ' ap', 'apl', 'ple' }
```

Without GIN however it will be performing very badly.

With a **GIN (or GiST) trigram index**:

```sql
CREATE INDEX idx_products_desc_trgm ON products USING GIN (description gin_trgm_ops);
```

PostgreSQL now stores **trigrams of each `description`** in an inverted index:

|Trigram|Rows containing it|
|---|---|
|"app"|[1, 5, 8, 22]|
|"ppl"|[1, 8, 22]|
|"ple"|[1, 22]|
|"ban"|[3, 9, 12]|
When you run the search, Postgres retrieves all rows containing trigrams similar to `"apple"` and only checks those — avoiding full-table scan.

Result: **massive speedup**, often 100x–1000x faster.

### Categorical data handling:

Unlike other colums it is tricky to handle categorical data. Reason being categorical values are limited while number of rows can be huge. We know indexing perfroms very bad when a column has very less unique values. If small number of elements are asked using pagination we can use offset and liit but it becomes slow as newer pages are used reason being offset needs to calculate the result first and then skip the rows. 

Better idea is to use keyset pagination.here we maintain a pointer on the rows and move this pointer forward and backward for paging.

|Feature|OFFSET + LIMIT|Keyset|Explanation|
|---|---|---|---|
|**Jump to any page**|✅ Yes|❌ No|OFFSET pagination simply skips `offset` rows — so you can say `OFFSET 20000 LIMIT 10` to jump to page 2001. Keyset pagination depends on the _last seen id_, so to go to page 2001 you’d need all previous 2000 page markers.|
|**Performance for deep pages**|❌ Slows down|✅ Constant|OFFSET must _scan and discard_ all rows before the offset. Even with an index, PostgreSQL still walks through them internally. Keyset pagination uses a direct comparison like `WHERE id > last_id`, which is **O(1)** with a proper index.|
|**UI “Next/Prev”**|✅ Easy|✅ Perfect|Both can implement “Next/Prev” easily, but Keyset is smoother for infinite scroll or feed-like UIs — no jumpiness when data changes.|
|**Consistent results when data changes**|❌ No|✅ Yes|OFFSET pagination breaks if rows are inserted/deleted during navigation — later pages may shift. Keyset uses a cursor (last id), so newly inserted rows before that id won’t change the next page’s results.|
|**Memory / CPU usage**|🧠 Heavy|🧠 Light|OFFSET pagination makes Postgres read and discard rows up to the offset — wasting CPU and memory. Keyset only reads _continuation rows_ from the index — much more efficient.|

Sure — here’s the clear comparison in one paragraph:

When you use **`WHERE order_id > last_seen AND category = 1 ORDER BY order_id`** (keyset pagination), PostgreSQL uses the index on `(category, order_id)` to **directly jump to the first row after `last_seen` in that category** and return the next batch (`LIMIT 10` or whatever). It reads **only the rows you need**, so query time is nearly constant regardless of how deep you are in the dataset. In contrast, **`LIMIT ... OFFSET ...`** still uses the index for filtering/sorting, but PostgreSQL must **scan and skip all rows up to the offset** before returning your requested batch. That means the deeper the page (higher offset), the more rows are read and discarded internally, causing query time to grow linearly with the offset, even if you have the same indexes. Keyset is O(1) per page, OFFSET is O(n) per offset.

#### Practically working:

In reality websites often don't allow to jump to a speciifc page. For example amazon does not allows to skip to last page. Similarly only forward or backward going is possible while going forward is easy.

```sql
SELECT * 
FROM products 
WHERE category = 1 
  AND order_id > :last_seen
ORDER BY order_id ASC
LIMIT 10;
```

Backward

```sql
SELECT *
FROM products
WHERE category = 1
  AND order_id < :first_seen  -- note: "<" instead of ">"
ORDER BY order_id DESC          -- reverse order
LIMIT 10;
```

Now a note about the total count of pages. Usually webiste don't count the total number of products rather they do approx count and return the result

```sql
SELECT CEIL(reltuples / 20.0) AS estimated_pages
FROM pg_class
WHERE relname = 'products';
```

```sql
CREATE INDEX idx_products_category_id
ON products (category, id);
```

One more technique which can be used is to have compound indexing. 
Postgres B-tree indexes are **ordered left-to-right**.  
It means this index can efficiently handle:

- Filtering by `category` (the first column),
- Then ordering or range lookups by `id`.

```sql
SELECT * FROM products WHERE category = 'electronics' ORDER BY id LIMIT 20;
```

Note that swaping the orders won't work - 

```sql
CREATE INDEX idx_products_id_category ON products (id, category);
```


## Query optimizations

So far we've covered the different types of indexes PostgreSQL offers, but there's more to query optimization than just picking the right index type.

### Covering Index

When PostgreSQL uses an index to find a row, it typically needs to do two things:

1. Look up the value in the index to find the row's location
2. Fetch the actual row from the table to get other columns you need

But what if we could store all the data we need right in the index itself? That's what covering indexes do:

```sql
-- Let's say this is a common query in our social media app:
SELECT title, created_at 
FROM posts 
WHERE user_id = 123 
ORDER BY created_at DESC;

-- A covering index that includes all needed columns
CREATE INDEX idx_posts_user_include 
ON posts(user_id) INCLUDE (title, created_at);
```

Covering indexes can make queries significantly faster because PostgreSQL can satisfy the entire query just from the index without touching the table. The trade-off is that the index takes up more space and writes become slightly slower.

### Parital index

Sometimes you only need to index a subset of your data. For example, in our social media platform, most queries are probably looking for active users, not deleted ones:

```sql
-- Standard index indexes everything
CREATE INDEX idx_users_email ON users(email);  -- Indexes ALL users

-- Partial index only indexes active users
CREATE INDEX idx_active_users 
ON users(email) WHERE status = 'active';  -- Smaller, faster index
```

Partial indexes are particularly effective in scenarios where most of your queries only need a subset of rows, when you have many "inactive" or "deleted" records that don't need to be indexed, or when you want to reduce the overall size and maintenance overhead of your indexes. By only indexing the relevant subset of data, partial indexes can significantly improve both query performance and resource utilization.

## Practical performance

1. Query performance - 10k per s per core
2. Complex joins - 1k per s
3. Max table size upto - 100M

These aren't hard limits - PostgreSQL can handle much more with proper optimization. But they're good rules of thumb for when you should start considering partitioning, [sharding](https://www.hellointerview.com/learn/system-design/core-concepts/sharding), or other scaling strategies.

Keep in mind, memory is king when it comes to performance! Queries that can be satisfied from memory are orders of magnitude faster than those requiring disk access. As a rule of thumb, you should try to keep your working set (frequently accessed data) in RAM for optimal performance.

## Write performance

When a write occurs in PostgreSQL, several steps happen to ensure both performance and durability:

1. **Transaction Log (WAL) Write Disk**: Changes are first written to the Write-Ahead Log (WAL) on disk. This is a sequential write operation, making it relatively fast. The WAL is critical for durability - once changes are written here, the transaction is considered durable because even if the server crashes, PostgreSQL can recover the changes from the WAL.
    
2. **Buffer Cache Update Memory**: Changes are made to the data pages in PostgreSQL's shared buffer cache, where the actual tables and indexes live in memory. When pages are modified, they're marked as "dirty" to indicate they need to be written to disk eventually.
    
3. **Background Writer Memory → Disk**: Dirty pages in memory are periodically written to the actual data files on disk. This happens asynchronously through the background writer, when memory pressure gets too high, or when a checkpoint occurs. This delayed write strategy allows PostgreSQL to batch multiple changes together for better performance.
    
4. **Index Updates Memory & Disk**: Each index needs to be updated to reflect the changes. Like table data, index changes also go through the WAL for durability. This is why having many indexes can significantly slow down writes - each index requires additional WAL entries and memory updates.

This architecture is why PostgreSQL can be fast for writes - most of the work happens in memory, while ensuring durability through the WAL. The actual writing of data pages to disk happens later and is optimized for batch operations.


### Throughput

Now we know about what happens when a write occurs in PostgreSQL, before we go onto optimizations, let's first talk about the practical limits of write throughput. This is important to know as it will help you decide whether PostgreSQL is a good fit for your system.

A well-tuned PostgreSQL instance on good (not great) hardware can handle:

- Simple inserts: ~5,000 per second per core
- Updates with index modifications: ~1,000-2,000 per second per core
- Complex transactions (multiple tables/indexes): Hundreds per second
- Bulk operations: Tens of thousands of rows per second

These numbers assume PostgreSQL's default transaction isolation level (Read Committed)

Remember, we're talking about a single node here! So if your system has higher write throughput that, say, 5k writes per second, this does not mean that PostgreSQL is off the table, it just means that you are going to need to shard your data across multiple nodes/machines.

Several factors:

- Hardware: Write throughput is often bottlenecked by disk I/O for the WAL
- Indexes: Each additional index reduces write throughput
- Replication: If configured, synchronous replication adds latency as we wait for replicas to confirm
- Transaction Complexity: More tables or indexes touched = slower transactions

### Optimisation for write performance

Simple is vertical scaling. Second is batch processing. Instead of processing each write individually, we collect multiple operations and execute them in a single transaction. For example, instead of inserting 1000 likes one at a time, we can insert them all in a single transaction.

Writing offload - 

Some writes don't need to happen synchronously. For example, analytics data, activity logs, or aggregated metrics can often be processed asynchronously. Instead of writing directly to PostgreSQL, we can:

1. Send writes to a message queue (like Kafka)
2. Have background workers process these queued writes in batches
3. Optionally maintain a separate analytics database

For large tables, partitioning can improve both read and write performance by splitting data across multiple physical tables. The most common use case is time-based partitioning.

```sql
CREATE TABLE posts (
    id SERIAL,
    user_id INT,
    content TEXT,
    created_at TIMESTAMP
) PARTITION BY RANGE (created_at);

-- Create partitions by month
CREATE TABLE posts_2024_01 PARTITION OF posts
    FOR VALUES FROM ('2024-01-01') TO ('2024-02-01');
```

First, different database sessions can write to different partitions simultaneously, increasing concurrency. Second, when data is inserted, index updates only need to happen on the relevant partition rather than the entire table. Finally, bulk loading operations can be performed partition by partition, making it easier to load large amounts of data efficiently.

Conveniently, it also helps with reads. When users view recent posts, PostgreSQL only needs to scan the recent partitions. No need to wade through years of historical data.


Sharding is the most common solution in an interview. When a single node isn't enough, sharding lets you distribute writes across multiple PostgreSQL instances. You'll just want to be clear about what you're sharding on and how you're distributing the data.

For example, we may consider sharing our posts by user_id. This way, all the data for a user lives on a single shard. This is important, because when we go to read the data we want to avoid cross-shard queries where we need to scatter-gather data from multiple shards.

You typically want to shard on the column that you're querying by most often. So if we typically query for all posts from a given user, we'll shard by user_id.

## Replication

Replication is the process of copying data from one database to one or more other databases. This is a key part of PostgreSQL's scalability and availability story.

PostgreSQL supports two main types of replication: synchronous and asynchronous. In synchronous replication, the primary waits for acknowledgment from replicas before confirming the write to the client. With asynchronous replication, the primary confirms the write to the client immediately and replicates changes to replicas in the background. While the technical details may not come up in an interview, understanding these tradeoffs is important - synchronous replication provides stronger consistency but higher latency, while asynchronous replication offers better performance but potential inconsistency between replicas.


Replication of data across various nodes helps in - 

### Scaling reads

The most common use for replication is to scale read performance. By creating read replicas, you can distribute read queries across multiple database instances while sending all writes to the primary. This is particularly effective because most applications are read-heavy.

There's one key caveat with read replicas: replication lag. If a user makes a change and immediately tries to read it back, they might not see their change if they hit a replica that hasn't caught up yet. This is known as "read-your-writes" consistency.

### High availability

The second major benefit of replication is high availability. By maintaining copies of your data across multiple nodes, you can handle hardware failures without downtime. If your primary node fails, one of the replicas can be promoted to become the new primary.

In your interview, emphasize that replication isn't just about scaling - it's about reliability. You might say: "We'll use replication not just for distributing read load, but also to ensure our service stays available even if we lose a database node."

### Data consistency

If you've chosen to prioritize consistency over availability in your non-functional requirements, then PostgreSQL is a strong choice. It's built from the ground up to provide strong consistency guarantees through ACID transactions.

One of the most common points of discussion in interviews ends up being around transactions. A transaction is a set of operations that are executed together and must either all succeed or all fail together. This is the foundation for ensuring consistency in PostgreSQL.

Consider simple case of auction bidding a simple operation may be to get the bit place a bid higher than it and then updating the bid.

```sql
BEGIN;
-- Get current max bid for item 123
SELECT maxBid from Auction where id = 123;

-- Place new bid if it's higher
INSERT INTO bids (item_id, user_id, amount) 
VALUES (123, 456, 100);

-- Update the max bid
UPDATE Auction SET maxBid = 100 WHERE id = 123;
COMMIT;
```

Even though this is in a transaction, with PostgreSQL's default isolation level (Read Committed), we could still have consistency problems if two users bid simultaneously. Both transactions could read the same max bid before either commits.

Here's how this could lead to an inconsistent state:

1. User A's transaction reads current max bid: $90
2. User B's transaction reads current max bid: $90
3. User A places bid for $100
4. User A commits
5. User B places bid for $95
6. User B commits

Note that in this case bid 95 will be placed. To solve this we have two options- 

1. Row level locking - When this is done a lock is aquired at the row and will not be released until transaction is done. Thus bids will be placed correctly

```sql
BEGIN;
-- Lock the item and get current max bid
SELECT maxBid FROM Auction WHERE id = 123 FOR UPDATE;

-- Place new bid if it's higher
INSERT INTO bids (item_id, user_id, amount) 
VALUES (123, 456, 100);

-- Update the max bid
UPDATE Auction SET maxBid = 100 WHERE id = 123;
COMMIT;
```

2. Higher isolation levels
One thing to note about the isolation levels is postgres predominantly have three levels - Read commited , Repeatable read and Serializable. However **Repeatable Read** in PostgreSQL provides stronger guarantees than the SQL standard requires. It creates a consistent snapshot of the data as of the start of the transaction, and unlike other databases, PostgreSQL's implementation prevents both non-repeatable reads AND phantom reads. This means not only will the same query return the same results within a transaction, but no new rows will appear that match your query conditions - even if other transactions commit such rows.

**Serializable** is the strongest isolation level that makes transactions behave as if they were executed one after another in sequence. This prevents all types of concurrency anomalies but comes with the trade-off of requiring retry logic in your application to handle transaction conflicts.

## When not to use postgres

1. Extreme write throughput - Use apache cassandra instead.

### Change data capture

**Change Data Capture (CDC)** is a modern software design pattern used to determine and track data that has changed in a database. Instead of copying an entire database every time you want to update a data warehouse or a search index, CDC allows you to stream only the "deltas"—the specific inserts, updates, and deletes—in real-time.

## How CDC Works (High Level)

Most modern CDC systems follow these steps:

1. **Capture:** The system monitors the database for any changes (DML operations).
2. **Extract:** The specific change (e.g., "User 5 updated their email") is captured in a structured format (usually JSON or Avro).
3. **Transport:** The change is sent to a message broker (like Apache Kafka) or a destination system.
4. **Load:** The destination (like Snowflake, Elasticsearch, or a microservice) applies the change.


PostgreSQL implements CDC primarily through a feature called **Logical Decoding**. This is built on top of the **Write-Ahead Log (WAL)**.

Every time you make a change in Postgres, it is first written to the WAL. This is a binary file that records every single operation for crash recovery purposes. Since the WAL already contains all the changes, it is the perfect source for CDC.

The binary data in the WAL is hard for external systems to read. **Logical Decoding** is the process of taking those binary logs and "decoding" them into a readable format (like a series of SQL statements or JSON).
- **Replication Slot:** This is a "bookmark" in the Postgres WAL. It tells Postgres: _"Don't delete these old log files yet; the CDC consumer hasn't read them yet."_ This ensures no data is lost even if the CDC tool goes offline for a few minutes.
- **Output Plugin:** This is the "translator." It takes the raw WAL data and formats it. A popular plugin is `pgoutput` (the standard for Postgres 10+).

## The Postgres CDC Workflow
1. **Operation:** A user runs `UPDATE users SET status = 'active' WHERE id = 10;`.
2. **WAL Entry:** Postgres writes this operation to the WAL file on disk.
3. **Logical Slot:** A CDC tool (like **Debezium**) connects to a specific replication slot.
4. **Decoding:** The `pgoutput` plugin reads the WAL and transforms the binary update into a message: `{ "op": "u", "before": {"id": 10, "status": "inactive"}, "after": {"id": 10, "status": "active"} }`.
5. **Streaming:** This message is pushed out to the listener in real-time.