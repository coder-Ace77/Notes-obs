
---

Queues serve as buffers for bursty traffic or as a means of distributing work across a system. A compute resource sends messages to a queue and forgets about them. On the other end, a pool of workers (also compute resources) processes the messages at their own pace. Messages can be anything from a simple string to a complex object.

The queue's function is to smooth out the load on the system. If I get a spike of 1,000 requests but can only handle 200 requests per second, 800 requests will wait in the queue before being processed — but they are not dropped! Queues also decouple the producer and consumer of a system, allowing you to scale them independently. I can bring down and up services behind a queue with negligible impact.


> [!NOTE] 
> Be careful of introducing queues into synchronous workloads. If you have strong latency requirements (e.g. < 500ms), by adding a queue you're nearly guaranteeing you'll break that latency constraint.

Use cases:

**Buffer for Bursty Traffic**: In a ride-sharing application like Uber, queues can be used to manage sudden surges in ride requests. During peak hours or special events, ride requests can spike massively. A queue buffers these incoming requests, allowing the system to process them at a manageable rate without overloading the server or degrading the user experience.

n a cloud-based photo processing service, queues can be used to distribute expensive image processing tasks. When a user uploads photos for editing or filtering, these tasks are placed in a queue. Different worker nodes then pull tasks from the queue, ensuring even distribution of workload and efficient use of computing resources.

Most queues support FIFO however we can also maintain complex ordering.

Many queues have built-in retry mechanisms that attempt to redeliver a message a certain number of times before considering it a failure.

Dead letter queues are used to store messages that cannot be processed. They're useful for debugging and auditing, as it allows you to inspect messages that failed to be processed and understand why they failed.

Queues can be partitioned across multiple servers so that they can scale to handle more messages. Each partition can be processed by a different set of workers. Just like databases, you will need to specify a partition key to ensure that related messages are stored in the same partition.

- **Backpressure**: The biggest problem with queues is they make it easy to overwhelm your system. If my system supports 200 requests per second but I'm receiving 300 requests per second, I'll never finish them! A queue is just obscuring the problem that I don't have enough capacity. The answer is backpressure. Backpressure is a way of slowing down the production of messages when the queue is overwhelmed. This helps prevent the queue from becoming a bottleneck in your system. For example, if a queue is full, you might want to reject new messages or slow down the rate at which new messages are accepted, potentially returning an error to the user or producer.


## Apache Kafka

It is open source distributed event streaming platform that can be used either as message queue or stream processing system. 

But before that we need to look into the problem with queue servers. Suppose we start by making a server with simple queue in memory it will be fast but will not be able to handle the too many elements in the queue.

Next suppose we think about the persisting the queue in the db and each time element is pushed to the queue we write it on db. It will handle lot of data but will not be able to be so fast.
Again we can try to go with in memory queue and with multiple servers but now we will not be maintain the strict order.


Apache kafka solves all these problems. 
Kafka contains multiple nodes(servers) and there are multiple queues but queues are partitioned in such a way by the user so that a type of events is handled only by the given server only. 

So partitions exist in kafka. For details view `Kafka in depth` in the core engineering.

Now suppose producer is `producing` events very fast so that queue might be able to store and process(write and read) very fast but consumer may not be able to process them so fast. So we can make multiple copies of consumers. If we are allowed to have a entry read by multiple consumers its fine but if there is a constraint on a request should be taken by one consumer then we have to create consumer group.  

## Terms in kafka

### Message/Record

The unit of data within Kafka is called a message.

Messages can have an optional bit of metadata which is referred to as a key. The key is also a byte array, and as with the message, has no specific meaning to Kafka. Keys are used when messages are to be written to partitions in a more controlled manner.

For efficiency, messages are written into Kafka in batches. A batch is just a collection of messages, all of which are being produced to the same topic and partition. Batches are also typically compressed, which provides for more efficient data transfer and storage at the cost of some processing power.

### Schema:

While messages are opaque byte arrays to Kafka itself, it is recommended that additional structure be imposed on the message content so that it can be easily understood. Eg-- json,xml.


### Topics:

Messages in Kafka are categorized into topics. The closest analogy for a topic is a data‐ base table, or a folder in a filesystem.

Topics are additionally broken down into a number of partitions. Going back to the “commit log” description, a partition is a single log. Note that physically a log file exist for each partition involved.

Messages are written to it in an append-only fashion, and are read in order from beginning to end.

### Partitions:

Partitions are also the way that Kafka provides redundancy and scalability. Each partition can be hosted on a different server, which means that a single topic can be scaled horizontally across multiple servers to provide for performance far beyond the ability of a single server.

![[Pasted image 20250808085230.png]]

### Producers and Consumers

Producers create new messages. In other publish/subscribe systems, these may be called publishers or writers. In general, a message will be produced to a specific topic. If no key messages will be partitioned evenly. All the messages with a given key are written to same topic.

A consumer can subscribe to one or more topics.
The consumer keeps track of which messages it has already consumed by keeping track of the offset of messages.

The offset is another bit of metadata, an integer value that continually increases, that Kafka adds to each message as it is produced. Each message within a given partition has a unique offset. By storing the offset of the last consumed message for each parti‐ tion, either in Zookeeper or in Kafka itself, a consumer can stop and restart without losing its place.

Consumers work as part of a consumer group. This is one or more consumers that work together to consume a topic. The group assures that each partition is only consumed by one member. In Figure 1-6, there are three consumers in a single group consuming a topic. Two of the consumers are working from one partition each, while the third consumer is working from two partitions. The mapping of a consumer to a partition is often called ownership of the partition by the consumer.

![[Pasted image 20250808090700.png]]

A single Kafka server is called a broker. The broker receives messages from producers, assigns offsets to them, and commits the messages to storage on disk.. It also services consumers, responding to fetch requests for partitions and responding with the messages that have been committed to disk.

### Controller:

Kafka brokers are designed to operate as part of a cluster. Within a cluster of brokers, one will also function as the cluster controller (elected automatically from the live members of the cluster). The controller is responsible for administrative operations, including assigning partitions to brokers and monitoring for broker failures. A partition is owned by a single broker in the cluster, and that broker is called the leader for the partition. A partition may be assigned to multiple brokers, which will result in the partition being replicated.

![[Pasted image 20250808091437.png]]

### Kafka producer

![[Pasted image 20250808094420.png]]

We start by creating a ProducerRecord, which must include the topic we want to send the record to and a value we are sending. Optionally, we can also specify a key and / or a partition. Once we send the ProducerRecord, the first thing the producer will do is serialize the key and value objects to ByteArrays, so they can be sent over the network.

Once partition is selected either by us or by partitioner(middleman to select partition if we don't select them.) It then adds the record to batch of records that will be send to same topic. Note that separate thread is responsible for sending the records. Broker also returns the record back. 

Now after this kafka needs to decide to which topic , broker and partition message be given to. If the message had key then partition will tell the kafka where this message goes. If no key then round robin is used. Now meesage reaches the broker and then broker will append the message to appropriate log file.

Now each message is also given a offset. 

![[Pasted image 20251027154242.png]]

Now consumer will keeps the offset with itself and will asks for the message with next offset and then will update offset. 
So that means if a new consumer has joined it will read from the begining of topic. So that might be good or bad depending upon the use case. Another thing that can happen is that a given consumer goes down we don't want the user to read from the begining so kafka solves it by periodically saving its offset to kafka cluster. 

To have durability and availabilty we have lot of replication in kafka. Replication is done at partition level. So for each pertition we have one leader replica and multiple follower replicas.To orchestrate this we need some cluster controller. 
Followers never interact with the client they act as stand alone if leader fails.

## When to use kafka

1. As the message queue
2. When tasks need to happen in certain order.
3. Decouple the producer and consumer. 
4. To handle real time data
5. Pub sub schenario

## Deep dives

Although kafka does not need this, it is however recommended to put limit on size of message. for eg 1mb for optimal performance. 

Kafka is designed for **high-throughput, sequential I/O** — it performs best when:
- Messages are small and numerous, allowing batching and compression.
- Brokers can efficiently append messages to logs and serve them to consumers.
Too-large messages cause:
- More disk I/O (page cache pressure)
- Longer replication times
- Higher network latency
- Potential broker and consumer memory pressure (since Kafka buffers messages in memory)

One broker can handle upto 1TB of data and 10k messages per second.

### Scaling kafka

For scaling we obviouly need more brokers. But we need to choose good partition key and this should be the important discussion point in interview. Good key which provides even distribution while bad key can lead to hot key problem. 

Handling hot partition:
Sometimes there will be too much data comming in for a given partition and how do you scale that. Suppose in ad click aggregator lot of data is comming in for a given ad_id. 

Solution 1 is to remove the key since it will mean now the data is randomly distribured across brokers.
Solution 2 is to use compound key eg add_id:user_id - means key will be stored not in one partition as hashes.
Solution 3 backpressure - Tell the producer to produce slowly

Fault tolerance and durability
When setting kafka cluster we have two important settings

- acts - how many followers need to have the message before taking next message. Larger acts more fault tolerance but less perfomant.
- replication factor - How many followers per leader.

When happens when consumer goes down? 

If single node then last read offset will already handle and consumer will read from there. However if consumer topic is there then in the mean time other consumers will have read and so there needs to be a rebalance. 

Kafka allows producer to retry to push the message into queue. Similarly consumer can also do returies.

#### Retention 

Kafka can retain messages for long period of times. By default it is 7days and 1gb.






