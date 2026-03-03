
---

## Scaling:

There are two kinds of scaling horizontal scaling and vertical scaling. 

![[Pasted image 20251010143705.png]]

It is important to note that horizontal scaling is much more complicated than vertical scaling and adding more machines is not a free lunch.Oftentimes by scaling you're forced to contend with the distribution of work, data, and state across your system.

When horizontal scaling is the right solution, you'll need to consider how to distribute work across your machines. Most modern systems use a technique called ["Consistent Hashing"](https://www.hellointerview.com/learn/system-design/deep-dives/consistent-hashing) to distribute work across a set of machines - this is a technique that arranges both data and machines in a circular space called a "hash ring", allowing you to add or remove machines with minimal data redistribution.

## CAP thoerum

In any distributed data system, you can only guarantee at most two of the following three properties simultaneously:

1. consistency: all nodes/users see the same data all the time
2. availability: every request gets response(either successful or not)
3. partition tolerance: system works despite network failure between nodes. 

Note that partition tolerance is a must and always be supported so system should be working. So we need to prioritize between consisitency and availability.

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

Now cases:

### Strong consistency:

1. Ticket booking platform: If we sell a ticket(write) then everyone must see it as unavailable.
2. Inventory(amazon): If the stock goes out of stock. We must not sell it.
3. Financial systems:

### Strong availability systems:

1. Social media applications
2. Streaming service like netflix

Its often logical to choose over consistency vs availability with systems. Because if something catestropic happens if stale data is shown then we need to have strong consistent system otherwise we can go with strong availability systems.

Now to design strong consistent systems we may be needed to go ahead with some pointers:

- implement distributed transations
- limited to single node (single db will have no such issues)
- accept higher latency
- discuss concensus protocol
- tools - 
	- postgres
	- NoSql with strong consistency(Dynamo DB)

With availability we can go  with
- multiple read replicas 
- eventual consistency
- cassandra or dynamo db

Different parts of system can prioritize different requirements - 

for instance we can have availability for events but consistency for booking tickets

for tinder we can prefer consistency for matching but availability for viewing profile data.

Types of consistency:

Strong consistency: all reads reflect latest write
Casual consistency: related events appear in order
eventual consistency: updates will propogate eventually
### Work distribution:

Load balancers are essential to balance the load across many machines. However here we have a choice of algorithm which will assign the work.While load balancers often come with many different strategies (e.g. least connections, utilization-based, etc), simple round robin allocation is often sufficient.

Our task at this point needs to be to have as even load as possible across the set of nodes

### Data distribution:

Each service while handling a request needs a way to get the required data. The node may require to keep data in-memory on the node that is processing request. More frequently, this implies keeping data in a database that's shared across all nodes. Look for ways to partition your data such that a single node can access the data it needs without needing to talk to another node.

**Fan-out** refers to **how many downstream services, components, or nodes a single request or event is sent to**.
It measures **how widely one component’s output is distributed**.

If you do need to talk to other nodes (a concept known as "fan-out"), keep the number small. A common anti-pattern is to have requests which fan out to many different nodes and then the results are all gathered together. This "scatter gather" pattern can be problematic because it can lead to a lot of network traffic, is sensitive to failures in each connection, and suffers from tail latency issues if the final result is dependent on every response.

Note that since there will be many services handling the same database. This introduces synchronization challenges.


> [!NOTE] Synchronization
> **Synchronization** means keeping **multiple processes, threads, or distributed components** consistent and coordinated when they access or modify **shared data or resources**.

| #     | Challenge                                               | Description                                                                                                            | Example                                                                                         |
| ----- | ------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------- |
| **1** | **Race Conditions**                                     | Two or more threads/processes access shared data simultaneously, and the outcome depends on timing.                    | Two users update same bank account balance at once — final balance incorrect.                   |
| **2** | **Deadlocks**                                           | Two or more threads wait on each other indefinitely for resources.                                                     | Thread A locks Resource1 → waits for Resource2; Thread B locks Resource2 → waits for Resource1. |
| **3** | **Data Inconsistency**                                  | Different nodes have different versions of the same data.                                                              | Two replicas of a database disagree on the user’s latest order status.                          |
| **4** | **Network Latency & Partitioning**                      | In distributed systems, nodes communicate over unreliable networks — leading to message delays, losses, or duplicates. | Message arrives late, and an old write overwrites a new one.                                    |
| **5** | **Clock Synchronization**                               | Different nodes’ clocks drift apart, so timestamps can’t be trusted for ordering.                                      | Node A’s “later” write appears earlier due to unsynced clocks.                                  |
| **6** | **Concurrent Updates**                                  | Two updates happen simultaneously, and the system must resolve which one wins (last write wins, merge, etc.).          | Two users edit the same document concurrently.                                                  |
| **7** | **Event Ordering**                                      | Determining causal order (which event happened first) in distributed systems.                                          | Needed in chat apps, collaborative editing, etc.                                                |
| **8** | **Consistency vs Availability Trade-off (CAP Theorem)** | Synchronizing across nodes often conflicts with availability — you can’t always have both during partitions.           | Distributed database under partition — should it reject writes (CP) or accept them (AP)?        |
| **9** | **Throughput and Bottlenecks**                          | Centralized locks or coordination reduce parallelism.                                                                  | A global lock slows down a high-traffic service.                                                |

### Communication protocols:

You've got two different categories of protocols to handle: internal and external. Internally, for a typical microservice application which constitutes 90%+ of system design problems, either HTTP(S) or gRPC will do the job. Externally, you'll need to consider how your clients will communicate with your system: who initiates the communication, what are the latency considerations, and how much data needs to be sent.

Across choices, most systems can be built with a combination of HTTP(S), SSE or long polling, and Websockets

Use HTTP(S) for APIs with simple request and responses. Because each request is stateless, you can scale your API horizontally by placing it behind a load balancer. Make sure that your services aren't assuming dependencies on the state of the client (e.g. sessions) and you're good to go.

If you need to give your clients near-realtime updates, you'll need a way for the clients to receive updates from the server. Long polling is a great way to do this that blends the simplicity and scalability of HTTP with the realtime updates of Websockets. With long polling, the client makes a request to the server and the server holds the request open until it has new data to send to the client. Once the data is sent, the client makes another request and the process repeats. Notably, you can use standard load balancers and firewalls with long polling - no special infrastructure needed.

Websockets are necessary if you need realtime, bidirectional communication between the client and the server. From a system design perspective, websockets can be challenging because you need to maintain the connection between client and server. This can be a challenge for load balancers and firewalls, and it can be a challenge for your server to maintain many open connections. A common pattern in these instances is to use a message broker to handle the communication between the client and the server and for the backend services to communicate with this message broker. This ensures you don't need to maintain long connections to every service in your backend

Lastly, Server Sent Events (SSE) are a great way to send updates from the server to the client. They're similar to long polling, but they're more efficient for unidirectional communication from the server to the client. SSE allows the server to push updates to the client whenever new data is available, without the client having to make repeated requests as in long polling. This is achieved through a single, long-lived HTTP connection, making it more suitable for scenarios where the server frequently updates data that needs to be sent to the client. Unlike Websockets, SSE is designed specifically for server-to-client communication and does not support client-to-server messaging. This makes SSE simpler to implement and integrate into existing HTTP infrastructure, such as load balancers and firewalls, without the need for special handling.


> [!NOTE] Note
> Statefulness is a major source of complexity for systems. Where possible, relegating your state to a message broker or a database is a great way to simplify your system. This enables your services to be stateless and horizontally scalable while still maintaining stateful communication with your clients.

## Security:

Finally one may be required to handle security related question:

Authorization: Only allowed user is given access to the resource.
Authentication: User is what they claim to be. 

When designing production systems, security should be top of mind. While system design interviews are rarely going to require you to do detailed security testing of your design, they are looking for you to emphasize security where appropriate. This means that you should be prepared to discuss how you'll secure your system at every level. Some of the most common security concerns are:

### Authentication / Authorization

In many systems you'll expose an API to external users which needs to be locked down to only specific users. Delegating this work to either an API Gateway or a dedicated service like Auth0 is a great way to ensure that you're not reinventing the wheel. Your interviewer may want you to discuss the finer details like how specific entities are secured, but often it's sufficient to say "My API Gateway will handle authentication and authorization".

### Encryption

Once you're handling sensitive data, it's important to make sure you're keeping it from snooping eyes. You'll want to cover both the data in transit (e.g. via protocol encryption) and the data at rest (e.g. via storage encryption). HTTPS is the SSL/TLS protocol that encrypts data in transit and is the standard for web traffic. If you're using gRPC it supports SSL/TLS out of the box. For data at rest, you'll want to use a database that supports encryption or encrypt the data yourself before storing it.

For sensitive data, it can often be useful for the end-user to control the keys. This is a common pattern in systems that need to store sensitive data. For example, if you're building a system that stores user data, you might want to encrypt that data with a key that's unique to each user. This way, even if your database is compromised, the data is still secure.

### Data Protection

Data protection is the process of ensuring that data is protected from unauthorized access, use, or disclosure. Frequently, production systems are concerned with data that's sensitive when exposed but might not be within an authorization path (e.g. a user's data might be exposed via a createFriendRequest endpoint). Many exposures of this sort are discovered by scraping endpoints, so having some sort of rate limiting or request throttling is a good idea.

## Concurrency

Sequencial concurrency happens when requests go one by one in order of commming. 
Parallel execution happens when multiple tasks are running parallely. Each CPU core can be think of as single unit. Parallelism is about doing multiple tasks simultaneouly concurrency is about managing multiple tasks. In concurrency single processor can be made to give you illusion that multiple process are running at the same time. 

In a single core env only concurrency can be achieved while in multicore both can be achieved simultaneously.

Any good application needs concurrency for an instance a web server running on say 8 cores can handle more than 100s of request per second because of concurrency. A os is able to run all the keyboard input and simultaneously also give output on monitor becasue of concurrency.

“**concurrency errors**” happen when **multiple threads, processes, or tasks access shared resources at the same time** in an uncoordinated way, causing **unexpected, inconsistent, or incorrect results**.

common concurrency issues:

| Type                    | Description                                                                                                           | Example                                                                                                       |
| ----------------------- | --------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------- |
| **Race condition**      | Two or more threads access shared data simultaneously, and the final result depends on the timing of their execution. | Two threads increment the same counter → both read the same value before incrementing → final result = wrong. |
| **Deadlock**            | Two threads each hold a lock and wait for the other’s lock, so both are stuck forever.                                | Thread A locks X, waits for Y; Thread B locks Y, waits for X.                                                 |
| **Livelock**            | Threads keep responding to each other’s actions but make no progress.                                                 | Two threads repeatedly yield to each other while trying to acquire the same lock.                             |
| **Starvation**          | One thread never gets CPU time or access to a resource because others monopolize it.                                  | High-priority threads continuously run, low-priority thread never executes.                                   |
| **Atomicity violation** | Operation assumed to be atomic actually isn’t.                                                                        | `if (!flag) flag = true;` executed by two threads — both see `flag = false` and both set it `true`.           |
| **Order violation**     | One thread depends on another’s operation order, but it’s not enforced.                                               | Thread B reads variable before Thread A initializes it.                                                       |
Now concurrency errors can occur at memory level. For example two threads sprwned tried to access same memory data. However it can also occur with any other resource like database , cache etc. 

Note that languages like nodejs avoid shared memory concurency so concurrency errors not happen there but in langauges like java and c++ which do not have any of the internal transaction mechanism so we need to use locks or mutexes.
