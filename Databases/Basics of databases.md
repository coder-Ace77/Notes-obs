
---

**Database** is far more than a simple electronic filing cabinet; it is a systematic, organized collection of data designed for rapid search and retrieval. The **need** for databases arises from the limitations of traditional file systems, specifically the requirements for **concurrency** (multiple people accessing data at once), **integrity** (ensuring data remains accurate), and **scalability** (handling petabytes of data without losing speed).

A **Database Management System (DBMS)** is the software layer that sits between the data and the user. It acts as a gatekeeper, handling tasks like storage, security, and transaction management. Without a DBMS, developers would have to manually write code to handle how data is physically written to a disk every single time they want to save a record.

#### Key Components of a DBMS:

- **Query Processor:** Interprets user commands (like SQL) and optimizes them for performance.
- **Storage Manager:** Controls how data is physically stored and retrieved from the hardware.
- **Transaction Manager:** Ensures that database operations are reliable and follow **ACID** properties.

The architecture of a database system dictates how the client (user), the server, and the business logic interact.

Data abstraction is the process of hiding complex storage details from users to simplify their interaction with the system. This is typically visualized through the

**Three-Schema Architecture**:

**Physical (Internal) Level:** The lowest level. It describes **how** the data is actually stored (e.g., on SSDs, using B-trees or hashing). This is managed by system programmers.**Physical (Internal) Level:** The lowest level. It describes **how** the data is actually stored (e.g., on SSDs, using B-trees or hashing). This is managed by system programmers.

**Logical (Conceptual) Level:** The middle level. It describes **what** data is stored and the relationships between them (e.g., tables, fields, and constraints). This is where Database Administrators (DBAs) work.

**View (External) Level:** The highest level. It describes only the **part** of the database relevant to a specific user. For example, a student can see their grades, but not the salaries of the professors


## SQL vs NOSQL

##### Rigidity

First thing about sql is its rigid realtional structure. Once schema is fixed it can not be changed. You must define your "schema" (the columns and data types) before you can insert a single piece of data. Data is usually present as normalized format here.

NoSQL databases are **Non-Relational** and come in various shapes (Documents, Key-Value, Graphs, Column-family).There is no fixed schema. One record could have 5 fields, and the next could have 50. Data is present in denormalized format here. 

#### Data integrity ACID vs BASE

SQL dbs are acid compliance by default. However most of NOSQL dbs are not ACID compiant when working in distributed enviornment (Mongodb is acid compliant). However NoSQL dbs follow something called as `BASE`. 

#### BASE

- Bascially available - The system guarantees that it will respond to any request, even if some nodes are down or the data is in an inconsistent state. Idea is that it is better to give the user _some_ answer (even if it's slightly stale) than no answer at all. If you're on a social media site and a database node in Europe goes down, you might not see the latest comment from a friend there, but the site itself still loads and functions for you.

- Soft state - In an ACID system, data is "hard"—it is either committed or it isn't. In BASE, the state of the system can change over time, even without any new input. **The Philosophy:** Because data is being replicated across many servers, the "true" state of the system is fluid. In Practice: Different nodes in a cluster might hold different versions of the same data for a brief window of time while they synchronize.

- Eventual consistency - System will not be consistent at all the times but will become evental consistent povided no new updates are done to the db. Howeer in this respect every nosql server behaves differently. For example Mongodb and cassandra have levels consistency which can be defined for example majority write which is slow but is more consistent , all write then acknowledge or one node acknowledge which is fastest. 
### Scalablity

Traditional SQL databases (like MySQL, PostgreSQL, or Oracle) are designed to run on a **single node**. To handle more load, you typically make that single server more powerful. While adding more read replicas is good for availabilty and more speed. But it does not increase the scaalbility as one node must have entire data in sql. 

To scale SQL horizontally, you must implement **Sharding**. This involves breaking the database into smaller chunks (shards) and spreading them across different servers. 

There are mainly two reasons to why sql dbs are not sharded in this way - 

- Sharding is usually not native to traditional SQL. It requires complex application-level logic to know which shard holds which data.
- **Join Issues:** Performing `JOIN` operations across different shards is extremely difficult and slow, often forcing developers to denormalize data anyway. The reason for slowness is due to complex join which are across nodes. Tihs is slow as application itself needs to get all the data needed for the join from all the nodes and then manually joining it. Larger the fan out slower will be the system. 

However join issues can removed sometimes if the data is stored as highly normalize format. This can be done due to descrease in storage costs. So the lines between sql and nosql is getting blurred. 

NoSQL databases use **Automatic Sharding** (Partitioning). They are designed fro the software level to be distributed and therefore application logic does not need to work extensively. 
- **Data Distribution:** Data is automatically distributed across nodes based on a "partition key."
- **High Availability:** They use replication (copying data across multiple nodes) so that if one server fails, the system stays online.
- **Elasticity:** Many NoSQL services allow you to "spin up" new nodes with zero downtime, and the database automatically rebalances the data.

Never the less both sql and nosql can handle read replicas to enhance availability. In this architecure each shared parttion is cluster of write master and read slaves. Read is done primarlity from slaves and write is done to master. If master goes down one slave is automatically upgraded to master. This can be done by both though nosql dbs might be able to do it natively but sql dbs may need to do some work on application level. 

All this means nosql dbs can be horizonatally scaled up to virtually infinite capacity by adding more nodes. 

#### Horizontal scaling 

In horizontal scaling happens when we shard the data. Now with this there are multiple shars and each shard is an cluster of replicated read-write servers(Sometimes each cluster can be single node). With horizontal scaling there is an issue to auto rebalance data when some nodes are added to cluster or deleted from cluster(Here node means entire one replication cluster). Now this 

In a standard distributed system (like a cache cluster), you might use a simple **Modulo Hashing** algorithm to decide where to store data:

`Server=hash(key)(modn)`

If you have 4 servers and add a 5th (horizontal scaling), n changes from 4 to 5. Because the denominator has changed, almost every existing key will now map to a different server.
- **The Result:** A massive "cache miss" storm where you have to move nearly 100% of your data to new locations, potentially crashing your backend database.

**Consistent Hashing** minimizes the disruption caused by adding or removing nodes. Instead of mapping keys directly to a fixed number of servers, it maps both the **servers** and the **keys** onto a conceptual "Hash Ring."

1. **The Ring:** Imagine a circle representing the entire range of possible hash values (e.g., from 0 to 232−1).
2. **Placing Servers:** Each server is hashed and placed at a specific point on this ring.
3. **Placing Keys:** When a piece of data (a key) needs to be stored, it is also hashed onto the ring.
4. **The Assignment:** The key is assigned to the **first server it encounters** while moving clockwise around the ring.

When a new server is added to the ring, it only "takes over" a small segment of the ring from its immediate neighbor meaning only a fraction of the keys (roughly 1/n) need to be moved, rather than the entire dataset.

A common issue with basic consistent hashing is non-uniform distribution. If servers are placed randomly, one server might end up responsible for a much larger "slice" of the ring than others, leading to a "hot spot."

- Instead of placing a server once on the ring, we hash it multiple times (e.g., `Server1_A`, `Server1_B`, `Server1_C`).
- This spreads the server's presence across the ring, ensuring that if one server fails or a new one is added, the load is redistributed more evenly across the entire cluster.

### Querying and Relationships

- **SQL (JOINS):** SQL is the king of relationships. It uses a powerful, standardized language (SQL) that allows you to perform complex joins across dozens of tables. It is perfect for heavy reporting and analytics.

- **NoSQL (Patterns):** NoSQL is optimized for speed of access. You don't perform JOINS; you query for a specific ID and get a big chunk of data back instantly. If you need complex relationships in NoSQL, you often have to handle that logic in your application code, which can be messy.


### CAP theorum

In any distributed data system, you can only guarantee at most two of the following three properties simultaneously:

1. consistency: all nodes/users see the same data all the time
2. availability: every request gets response(either successful or not)
3. partition tolerance: system works despite network failure between nodes. 

Note that partition tolerance is a must and always be supported so system should be working. So we need to prioritize between consisitency and availability. Note that network failure does not only means broken internet but also means in distributed systems there is some time needed for communication in that small time disfferent nodes are virtually in different states. 

Let’s consider what happens **when a network partition occurs**:
### Example scenario

- You have **two replicas**, A and B.
- Network link between A and B **breaks** (partition).
- A client writes a new value `x=5` to **A**.
- Another client reads from **B**.

Now:
- If **B** responds immediately → it might return the **old value (x=3)** ⇒ violates **Consistency**.
- If **B** refuses or delays the request until it reconnects to A ⇒ violates **Availability**.

Thus, **under a partition**, you must **sacrifice either consistency or availability**.  
That’s the essence of CAP.

Consider a single replicated cluster (master - slave ) architecture now When a write is done to the master node.  Therefore the slave read only nodes get out of sync. Now there two options to block read from slaves untill all the slaves don't have the latest read this is more consistent system at the expense of availability. Or give out stale data immediately if we want more available system. 

In CP systems  - If a network partition occurs, the system will **refuse to respond** to requests until the data can be synchronized across the partition.

Best for - Banking, ATM withdrawals, and inventory management where an incorrect balance is a disaster.

Example - **Examples:** MongoDB (in certain configurations), Redis, Etcd.

In AP system -  If a network partition occurs, the system will **keep responding**, but the data might be out of sync

Best for - Social media "likes,"
**Examples:** Cassandra, DynamoDB, CouchDB.

CA systems are basically single node servers. 

By default, MongoDB is a CP system. It uses a Single-Master architecture (Replica Sets). If the master node can't talk to the majority of the other nodes, the system stops allowing writes until a new master is elected. Redis is typically CP. In a clustered setup, if a partition occurs, the "minority" side of the partition will eventually stop accepting writes to prevent data divergence.

CockroachDB  - This is a Distributed SQL database. It is strictly CP. It uses a consensus algorithm (Raft) to ensure that every single node agrees on the data before a write is confirmed. It would rather go offline than be inconsistent.

**Apache Cassandra:** The poster child for **AP**. It is "masterless," meaning any node can handle any request. If half the nodes are cut off, the other half will keep working. You’ll get your data, but it might be "eventually consistent."
**Amazon DynamoDB:** Usually classified as **AP**. It is built to never go down. However, it is "tunable"—you can actually request a "Strongly Consistent Read," which effectively moves that specific request into the CP category.
**CouchDB:** Another AP favorite. It uses a multi-master approach where nodes can be offline for days, and when they reconnect, they merge their changes using conflict resolution.

### Master-slave and master-master

This is the most common replication setup. It establishes a clear hierarchy: one server is the source of truth, and the others are followers. The master handles all **write** operations (`INSERT`, `UPDATE`, `DELETE`). It records every change in a "binlog" (binary log). Slaves  stay in sync by reading the Master's log and applying those same changes to their own local data. They are strictly **Read-Only**. 

This is perfect for apps with a high read-to-write ratio (like a news site or social media feed). You can add 10 slaves to handle millions of readers, while the master focuses on the few people posting updates. Because the data takes time to travel from the Master to the Slave, a user might write something to the Master and then instantly try to read it from a Slave, only to find it "missing" for a few milliseconds. This is used by mongodb. 

In master master setup there is no hierarchy. Every node in the cluster is a "peer" that can do everything.

- **Circular/Bi-directional:** Every node can handle both **Read** and **Write** operations.
- **Syncing:** When you write to Master A, it immediately broadcasts that change to Master B, and vice-versa.

These kind of dbs can have very high write throughput as multiple nodes can take the data. If Master A goes down, Master B is already primed and active. There is no "promotion" delay. However we can have clashes which might not e good. 