

Distributed cache provide us a great way to reduce latency. Be it for some computation which takes lot of computation or may be storing some data for faster access. Be it any sue case cache is not 100% guaranteed to give us better result due to cache hit/cache miss.

Uses: 
1. Reduce load on db especailly if lot of read calls.
2. reduce number of db queries. 
3. Store some data in districuted cache if needed for frequent access. 

Caches have limited storage. eviction policy tells us which items will be removed when cache is full.

- Least Recently Used (LRU): Evicts the least recently accessed items first.
- First In, First Out (FIFO): Evicts items in the order they were added.
- Least Frequently Used (LFU): Removes items that are least frequently accessed.

**Cache Invalidation Strategy**: This is the strategy you'll use to ensure that the data in your cache is up to date. For example, if you are designing Ticketmaster and caching popular events, then you'll need to invalidate an event in the cache if the event in your Database was updated (like the venue changed).

**Cache Write Strategy**: This is the strategy you use to make sure that data is written to your cache in a consistent way. Some strategies are:

- Write-Through Cache: Writes data to both the cache and the underlying datastore simultaneously. Ensures consistency but can be slower for write operations.
- Write-Around Cache: Writes data directly to the datastore, bypassing the cache. This can minimize cache pollution but might increase data fetch times on subsequent reads.
- Write-Back Cache: Writes data to the cache and then asynchronously writes the data to the datastore. This can be faster for write operations but can lead to data loss if the cache fails before the data is written to the datastore

Redis is a key-value store that supports many different data structures, including strings, hashes, lists, sets, sorted sets, bitmaps, and hyperloglogs. Memcached is a simple key-value store that supports strings and binary objects.

## Redis 

Redis(Remote Dictionary server) is in memory data store that can act as - 

- Database
- Cache
- Message broker
- Queue system

It’s written in **C** and is known for **extremely fast performance** because it stores data in **memory (RAM)** instead of on disk (though it can persist data to disk too).

Because memory access is much faster than disk access, Redis achieves **millions of ops/sec**.

Everything in redis is a key-value pair and works only on one thread. This means every operation in redis is an atomic operation.

So all the commands which redis supports resemble of a Hash Table at high level. 

Eg: 
SET key value
GET key
INC key -- increases the value of key.

Redis maintains durability by two ways - 

- Either it will operate on data snapshot level where it will save data periodically
- Secondly it may store data written to it in forms of append only logs. 

### Event loop

Redis has a single threaded event loop build on reactor pattern. So it uses non-blocking I/O via epoll/kqueue.
It can also handle thousands of client connections concurrently in a single thread. Why single thread actually most of the operations of redis are memory bound not cpu bound so context switching and locks are unnecessary.

Redis stores data in a **global dictionary** called the **keyspace**. This dictionary maps the string keys to values. Keys are `SDS(Simple Dynamic Strings)` a custom type in C.

Each value is a struct called redisObject

```c
typedef struct redisObject {
    unsigned type:4;
    unsigned encoding:4;
    void *ptr;  // pointer to actual data
    ...
} robj;
```

type defines the data structure of store STRING, LIST, HASH, SET, etc.
encoding defines how the value is represented internally (e.g. ziplist, hashtable).

Redis usese own memory management and maintains a refernce count on each object. When count becomes 0 memory becomes freed.

You can configure Redis with `maxmemory` to define an upper memory limit — when full, Redis uses **Eviction Policies** like:
- `volatile-lru` (Least Recently Used)
- `allkeys-lfu` (Least Frequently Used)
- `random`, etc.

### Redis Cluster

Redis can run in cluster mode where we will have a main node and some secondry nodes. Write will only happen to  `Main`node and main node will write this to a append only file. Now all the secondry nodes read from these files only. 
So read can be scaled almost to `infinite` but write trhough put will be very bad for the single node. 

To solve that we maintain multiple redis `main-secondry` subpairs. So what happens is that we divide the keys into multiple slot where each key belongs to exactly one slot. Basically each key is taken modulo by number of subclusters and then that data is put into that `sub-cluster`. Also `each redis` subnodes talk through the gossip protocol. So if the client do not know which redis cluster to go to redis node will tell you where to go. However for optimal performance it is better if client is aware of which node to go to. 

![[Pasted image 20251027141700.png]]


> [!NOTE] Note
> Redis clusters the sharded keys across different masters/main by hash slots. This is called clustering. Now each master can have multiple replicas which is used for scaling reads and high availability.


### Redis use

Some of the most fundamental data structures supported by Redis:

- Strings
- Hashes (objects/dictionaries)
- Lists
- Sets
- Sorted Sets (Priority Queues)
- Bloom Filters (probabilistic set membership; allows false positives)
- Geospatial Indexes
- Time Series
Redis also supports different communication patterns like Pub/Sub and Streams.

The core structure underneath Redis is a key-value store. Keys are strings while values which can be any of the data structures supported by Redis: binary data and strings, sets, lists, hashes, sorted sets, etc. All objects in Redis have a key.

Effectively, the way you organize the keys will be the way you organize your data and scale your Redis cluster.

### Commands

Redis' wire protocol is a custom query language comprised of simple strings which are used for all functionality of Redis.

```
SET foo 1  
GET foo     # Returns 1
INCR foo    # Returns 2
XADD mystream * name Sara surname OConnor # Adds an item to a stream
```

Redis is really, really fast. Redis can handle O(100k) writes per second and read latency is often in the microsecond range. This scale makes some anti-patterns for other database systems actually feasible with Redis. As an example, firing off 100 SQL requests to generate a list of items with a SQL database is a terrible idea, you're better off writing a SQL query which returns all the data you need in one request. On the other hand, the overhead for doing the same with Redis is rather low - while it'd be great to avoid it if you can, it's doable.

## Uses

### Redis cache

In this case, the root keys and values of Redis map to the keys and values in our cache.
When using Redis as a cache, you'll often employ a time to live (TTL) on each key. Redis guarantees you'll never read the value of a key after the TTL has expired.

A **hot key** is a key that receives **disproportionately high traffic** (reads or writes) compared to others. This may be the instagram account of some person accessed multiple times.

In a Redis cluster:

- Each key maps to exactly one node (based on its hash slot).
- If a few keys are accessed **thousands or millions of times more** than others,  
    the node(s) holding those keys get overloaded — CPU, memory bandwidth, and network saturation — even though other nodes remain underutilized.

A simple strategy is to break single key into multiple sharded keys and the way it is done is by appending it with some extra number. And now the hash of each key will make it so that all the duplicated keys are mapped to different nodes.

Second thing is that in cache the data should not be there for infinite time rather it should be updated from db time to time so that is reading correct value. The way it is done is through expiration policies. 
We attach EXPIRE commands to the sets and gets. In this expire setup the data will become stale even if memory is not full.

In LRU setup the data is put into the REDIS it is not full. Once its full we will remove the least recently used key and make the way for new key.

## Redis as rate limiter

In this case our task is to guard the down services from getting too many requests per second. So there has to be a  gateway. Many times gateway can store how many requests are comming in `not in db` but in memory. We implement bucket token protocol.

## Redis as the stream

 **Redis Streams** data type provides an append-only log structure that allows producers to continuously push events, while consumers read them in order, track their progress, and process them reliably. Each stream entry has an automatically generated, time-ordered ID and can store multiple key-value fields, making it ideal for handling structured event data such as logs, telemetry, or user activity.
 
  Redis Streams supports **consumer groups**, enabling multiple workers to share the workload without losing message order or duplicating processing. 
  
  Combined with Redis’s in-memory performance and persistence options, Streams deliver a high-throughput, low-latency mechanism for event collection, buffering, and distribution — serving as a lightweight alternative to traditional message brokers like Kafka or RabbitMQ in scenarios that require simplicity, speed, and horizontal scalability.

## Redis as distributed lock

Another common use of Redis in system design settings is as a distributed lock. Occasionally we have data in our system and we need to maintain consistency during updates. Eg Ticket master or the Uber design system. 

A very simple distributed lock with a timeout might use the atomic increment (INCR) with a TTL.

This is basically a shared counter. When we want to try to acquire the lock, we run INCR. If the response is 1 (i.e. we were the first person to try to grab the lock, so we own it!), we proceed. If the response is > 1 (i.e. someone else beat us and has the lock), we wait and retry again later. When we're done with the lock, we can DEL the key so that other processes can make use of it.

### Redis as Leaderboards

Redis' sorted sets maintain ordered data which can be queried in log time which make them appropriate for leaderboard applications. The high write throughput and low read latency make this especially useful for scaled applications where something like a SQL DB will start to struggle. 

This is usefull in schenarios where we have to maintain some leader board for example leetcode contests.

Finally note that redis can not perform aggragates across the nodes. Which means if you want to do the aggregation one needs to first get all the data and then use it. 
## CDN:

A content delivery network (CDN) is a type of cache that uses distributed servers to deliver content to users based on their geographic location. CDNs are often used to deliver static content like images, videos, and HTML files, but they can also be used to deliver dynamic content like API responses.

They work by caching content on servers bthat are close to users. When a user requests content, the CDN routes the request to the closest server. If the content is cached on that server, the CDN will return the cached content. If the content is not cached on that server, the CDN will fetch the content from the origin server, cache it on the server, and then return the content to the user.

CDN can also  be used for non static assests like blogs data or API response that remains same
Like other caches, CDNs have eviction policies that determine when cached content is removed. For example, you can set a time-to-live (TTL) for cached content, or you can use a cache invalidation mechanism to remove content from the cache when it changes

Common CDN are - Cloudflare , Akamai




