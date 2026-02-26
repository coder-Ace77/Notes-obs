
---

Data engineering is the branch how to collect the data, organize the data and store data at scale.
Data collection can be from various sources.

3 steps:

1. Ingredients: (data)
2. Organization(data organization)
3. Data processing - ETL(extract - transform - load ) 
4. Data systems
5. Data analysis
6. 

### 1. Ingredients: Data

The "ingredients" are the raw materials: the **data**. This can come from a wide variety of sources, including databases, flat files (like CSVs or Excel spreadsheets), APIs, streaming data feeds (from IoT devices or social media), or web scraping. Data can be structured (organized in a fixed schema, like a table), unstructured (like text documents or images), or semi-structured (like JSON or XML files). The quality and relevance of this raw data are critical to the success of any project.

### 2. Organization: Data Organization

Once you have the data, you need to organize it. This involves structuring and storing the data in a way that makes it accessible and useful. Key concepts in data organization include:

- **Data Models:** Creating a logical structure that defines how data is related and stored.
- **Data Warehouses:** Centralized repositories that store large amounts of historical data from multiple sources, typically for reporting and analysis.
- **Data Lakes:** Stores vast amounts of raw data in its native format, which is more flexible than a data warehouse.
- **Data Governance:** Establishing policies and procedures to ensure data quality, security, and privacy.

### 3. Data Processing: ETL (Extract-Transform-Load)

**ETL** is a fundamental process in data engineering. It’s the procedure for combining data from different sources into a single, unified view.

- **Extract:** You retrieve data from its source system.
- **Transform:** You clean, enrich, and standardize the data to make it consistent and usable. This may involve data type conversions, removing duplicates, handling missing values, or aggregating information.
- **Load:** You write the transformed data into a target data system, such as a data warehouse or data lake, where it's ready for analysis.

### 4. Data Systems

**Data systems** are the technologies and architectures used to manage and process the data. They provide the infrastructure for the entire data lifecycle. Examples include:

- **Databases:** Systems for storing and retrieving data, such as **MySQL** (relational) or **MongoDB** (NoSQL).
- **Big Data Frameworks:** Technologies like **Apache Spark** and **Apache Hadoop** are used for processing and managing extremely large datasets.
- **Cloud Platforms:** Services from providers like **AWS**, **Google Cloud**, and **Microsoft Azure** offer a wide range of tools for storage, processing, and analysis.

### 5. Data Analysis

**Data analysis** is the final stage, where the processed and organized data is examined to find insights and support decision-making. This can be done through various techniques:

- **Descriptive Analysis:** Summarizing historical data to understand what happened (e.g., "What were our total sales last quarter?").
- **Diagnostic Analysis:** Investigating the reasons behind past events (e.g., "Why did sales drop in the Midwest?").
- **Predictive Analysis:** Using statistical models and machine learning to forecast future outcomes (e.g., "What will sales be next quarter?").
- **Prescriptive Analysis:** Recommending actions to take based on the predicted outcomes (e.g., "To increase sales, we should target customers in this specific demographic").

Apache Kafka is a crucial technology in the field of data engineering, especially for building real-time data pipelines and streaming applications. Its core strength lies in its ability to handle high-throughput, low-latency data streams, making it a powerful tool for modern data architectures.

## Kafka

Kafka is distributed, which means it runs as a cluster of nodes spread across multiple servers. It's also replicated, meaning that data is copied in multiple locations to protect it from a single point of failure.

_Kafka helps you connect data sources to the systems using that data_

### Topics and Partitions

First, it's important to understand that a topic is a logical entity, a category to which records are published. However, the data isn't stored in the topic itself. Instead, a topic is divided into one or more **partitions**.

- **Partitions are the physical storage units of a topic.** Each partition is an ordered, immutable sequence of records.
- The partitioning of a topic is what enables Kafka's parallel processing. By dividing a topic into multiple partitions, you can have different producers and consumers reading and writing to them concurrently.

### The Role of Brokers

A **broker** is a single Kafka server, a node in the Kafka cluster. A cluster is composed of multiple brokers working together. The brokers are responsible for:

- **Storing data:** Brokers physically store the data for the partitions they are assigned to.
- **Serving read/write requests:** When a producer sends a message, it is sent to a specific broker, which then appends the message to a partition. Similarly, consumers connect to brokers to read messages from partitions.
- **Handling replication:** To ensure fault tolerance and high availability, partitions can be replicated. This means a copy of a partition is stored on multiple brokers. One broker is designated as the **leader** for that partition, handling all read and write requests. The other brokers with copies are called **followers**, and they simply replicate the leader's data. If the leader broker fails, one of the followers is automatically elected as the new leader.
- **Cluster Coordination:** Each broker in a Kafka cluster is aware of all other brokers and the metadata of the cluster (which topics exist, how many partitions they have, and which partitions are on which brokers).

### How They Work Together

The connection between topics and brokers is forged through the partitions. When you create a topic with multiple partitions, Kafka automatically distributes these partitions across the available brokers in the cluster.

For example, if you have a Kafka cluster with three brokers and you create a topic with six partitions, Kafka might distribute them like this:

- **Broker 1:** Leader for Partition 0, Replica for Partition 1
    
- **Broker 2:** Leader for Partition 1, Replica for Partition 2
    
- **Broker 3:** Leader for Partition 2, Replica for Partition 0
    

This distribution ensures:

1. **Load Balancing:** The workload of handling reads and writes is spread across all brokers.
    
2. **Scalability:** You can increase the throughput of your system by adding more partitions and more brokers.
    
3. **Fault Tolerance:** If Broker 1 goes down, the data in Partition 0 is not lost because a replica exists on Broker 3, and a new leader will be elected to take over.
    

In essence, brokers are the physical servers that host the partitions of topics, providing the essential infrastructure for data storage, replication, and serving clients. The topics, broken down into partitions, are the logical constructs that enable this distributed system to function efficiently and reliably.
### Producers

**Producers** are client applications that write (or "produce") data to Kafka topics. Their primary function is to:

- **Publish Records:** They create records, which are the basic unit of data in Kafka. A record consists of a key, a value, and a timestamp.

- **Select a Partition:** A producer can send a record to a specific partition of a topic. This is often done by using a hashing function on the record's key to ensure all records with the same key go to the same partition. This is crucial for maintaining message order for a specific key.

- **Handle Acknowledgements:** Producers can be configured to wait for acknowledgements from the broker that the record was successfully written and replicated. This provides different levels of durability guarantees.

Producers are responsible for the "publish" part of the publish-subscribe model.

### Consumers and Consumer Groups

**Consumers** are client applications that read (or "consume") data from Kafka topics. However, in Kafka, consumers almost always operate within a **consumer group**. This is the key to Kafka's scalable and fault-tolerant consumption model.

- **Consumer:** A consumer is a single process that subscribes to a topic and reads records from the partitions it is assigned. It keeps track of its position in the partition using an **offset**, which is a unique ID for each record within a partition.
    
- **Consumer Group:** A consumer group is a collection of one or more consumers that work together to consume a topic's partitions. All consumers in a group share a common `group.id`.
    

Here’s how they work together:

1. **Load Balancing:** When a consumer group subscribes to a topic, Kafka automatically distributes the topic's partitions among the consumers in that group. For example, if a topic has 6 partitions and a consumer group has 3 consumers, each consumer will be assigned 2 partitions. This allows for parallel processing of the data, significantly increasing the consumption throughput.
    
2. **State Management:** Each consumer in the group is responsible for a subset of the topic's partitions. The consumer group as a whole maintains the offset for each partition it is consuming. This means that if a consumer in the group fails, another consumer in the same group will be assigned its partitions and can resume consumption from the last committed offset, ensuring no data is lost.
    
3. **Scalability:** You can scale consumption by adding more consumers to a consumer group. As long as the number of consumers does not exceed the number of partitions, Kafka will automatically rebalance the partitions among the new, larger group. If the number of consumers exceeds the number of partitions, the extra consumers will be idle.
    

### An Analogy

Think of a library with multiple sections (the **partitions** of a **topic**).

- A **producer** is like an author who writes new books and places them in a specific section of the library.
    
- A single **consumer** is like a person reading books from one of the library sections.
    
- A **consumer group** is like a book club. The club decides to read every book in the library. To do this efficiently, the club members (the **consumers**) divide up the library sections among themselves so they can read in parallel. If one member gets sick, another member of the club can take over their sections and continue from where they left off.

## Zookeeper:

For many years, **Apache ZooKeeper** was an essential component of a Kafka cluster, acting as a centralized coordination service. Kafka brokers relied on ZooKeeper to manage and maintain the state of the cluster, ensuring all nodes were in sync.

However, it's important to note that **modern versions of Kafka are moving away from this dependency** by using a new, built-in metadata management system known as **KRaft (Kafka Raft)**.3 While many existing production environments still use ZooKeeper, new deployments are increasingly opting for the ZooKeeper-less architecture.

### Key Uses of ZooKeeper in Kafka Clusters

Before KRaft, Kafka brokers used ZooKeeper to perform several critical functions:4

- **Broker Registration:** When a Kafka broker starts, it registers itself with ZooKeeper.5 This allows all other brokers in the cluster to know which brokers are currently active and available.6 ZooKeeper maintains a list of all live brokers, acting as a registry for the cluster.7
    
- **Controller Election:** In a Kafka cluster, one broker is designated as the **controller**.8 This is a special role responsible for performing administrative tasks, such as assigning partitions to brokers and managing leader-follower relationships for all partitions.9 ZooKeeper facilitates the election of this controller and ensures there is always one active controller at a time.10
    
- **Metadata Management:** ZooKeeper stored all of the important metadata for the Kafka cluster.11 This included information about which topics exist, the number of partitions for each topic, and the location of the replicas for those partitions.12
    
- **Leader Election for Partitions:** If a broker that is the leader for a partition fails, ZooKeeper detects this failure and helps elect a new leader from the available replicas of that partition.13 This is crucial for maintaining the availability and fault tolerance of the data.14
    
- **Configuration and Synchronization:** ZooKeeper acted as a single source of truth for all cluster-wide configuration, ensuring that every broker had a consistent view of the cluster's state.15 It also managed distributed synchronization tasks to prevent conflicting actions.16

Running on system:

Run all commands from kafka folder only

```
.\bin\windows\zookeeper-server-start.bat .\config\zookeeper.properties
```

To run kafka first set envs:

```
$env:KAFKA_HEAP_OPTS="-Xmx512M -Xms512M"
.\bin\windows\kafka-server-start.bat .\config\server.properties
```

Creating topic:

--topic -> name of topic

```
 .\bin\windows\kafka-topics.bat --create --topic test-topic  --bootstrap-server localhost:9092
```

Producer prompt:

```
 .\bin\windows\kafka-console-producer.bat --topic test-topic --bootstrap-server localhost:9092
```

Consumer propt:

```
 .\bin\windows\kafka-console-consumer.bat --topic test-topic --bootstrap-server localhost:9092
```

