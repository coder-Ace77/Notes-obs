
---

Suppose we are starting from single node sql server and noware facing the issue in latency and throughput. Basically load on db is much more than it can handle. 

First thing we can do is to introduce application level cache. This might solve the problem of static data like user profile but not of dynamic data. Secondly we can reduce the normalization in database which will reduce the time of heavy normailization. 

We can also introduce indexing depending upon if the system is read heavy also we can optimise the queries.

Another parallel optimization that you can do is tweaking around database connections. Database client libraries and external libraries are available in almost all programming languages. You can use connection pool libraries to cache database connections or can configure connection pool size in the database management system itself.

Next up we can scale up the server. Upto its limit. Uptill now our db was not distributed system but rather a single node server. Now we might need to scale it up. First we can introduce multiple read replicas incase system is read heavy or use master-master architecure if system is write heavy. 

Now finally to handle more write thorughput with high `r/w` ratio we should be working in cluster logic with sharded db. DBs canstore data according to geographical locations which might be a good partition key. Most of the modern NoSQL dbs can handle such kind of data as it is. 