
---

These are the basic steps of delievery framework.

![[Pasted image 20251010134636.png]]

## Requirements:

The goal of the requirements section is to get a clear understanding of the system that you are being asked to design. To do this, we suggest you break your requirements into two sections.

### Functional requirements:

Functional requirements are your "Users/Clients should be able to..." statements.

In **system design**, **functional requirements** describe **what the system should do**—the specific behaviors, features, or functions it must provide to meet user needs. They define the expected outputs, responses, and operations of the system under certain conditions.

For example, if you were designing a system like Twitter, you might have the following functional requirements:

- Users should be able to post tweets
- Users should be able to follow other users
- Users should be able to see tweets from users they follow

A cache meanwhile might have requirements like:

- Clients should be able to insert items
- Clients should be able to set expirations
- Clients should be able to read items

### Non functional requirements:

Non-functional requirements are statements about the system qualities that are important to your users. These can be phrased as "The system should be able to..." or "The system should be..." statements.

For example, if you were designing a system like Twitter, you might have the following non-functional requirements:

- The system should be highly available, prioritizing availability over consistency
- The system should be able to scale to support 100M+ DAU (Daily Active Users)
- The system should be low latency, rendering feeds in under 200ms

Coming up with non-functional requirements can be challenging, especially if you're not familiar with the domain. Here is a checklist of things to consider that might help you identify the most important non-functional requirements for your system. You'll want to identify the top 3-5 that are most relevant to your system.

1. **CAP Theorem**: Should your system prioritize consistency or availability? Note, partition tolerance is a given in distributed systems.
    
2. **Environment Constraints**: Are there any constraints on the environment in which your system will run? For example, are you running on a mobile device with limited battery life? Running on devices with limited memory or limited bandwidth (e.g. streaming video on 3G)?
    
3. **Scalability**: All systems need to scale, but does this system have unique scaling requirements? For example, does it have bursty traffic at a specific time of day? Are there events, like holidays, that will cause a significant increase in traffic? Also consider the read vs write ratio here. Does your system need to scale reads or writes more?
    
4. **Latency**: How quickly does the system need to respond to user requests? Specifically consider any requests that require meaningful computation. For example, low latency search when designing Yelp.
    
5. **Durability**: How important is it that the data in your system is not lost? For example, a social network might be able to tolerate some data loss, but a banking system cannot.
    
6. **Security**: How secure does the system need to be? Consider data protection, access control, and compliance with regulations.
    
7. **Fault Tolerance**: How well does the system need to handle failures? Consider redundancy, failover, and recovery mechanisms.
    
8. **Compliance**: Are there legal or regulatory requirements the system needs to meet? Consider industry standards, data protection laws, and other regulations.


### Capacity estimation:

**Requests per second** is a measure of **how many requests a server, API, or service can handle per second**.

**Bandwidth** is the **maximum amount of data that can be transmitted over a network connection in a given time**, usually measured in bits or bytes per second.

Storage capacity is used to define how much storage we will be needing in next this span of time.

Here we will figure out the request per second (rps) of the application.
For instance with 20k users and each making 5 todos per day with read:write ratio of 5:1. 

Our rps is `20k*5` = 100K/d and 
rps = `100k/10^5`

which is 1 write req per second and 5 read req per second due to read and write ratio.
Now with 6 req/s our bandwidth is `20kb*6=120kb`  

### Core entities:

Next you should take a moment to identify and list the core entities of your system. This helps you to define terms, understand the data central to your design, and gives you a foundation to build on. These are the core entities that your API will exchange and that your system will persist in a Data Model.

For our Twitter example, our core entities are rather simple:

- User
- Tweet
- Follow


### API or system requirements:

Before you get into the high-level design, you'll want to define the contract between your system and its users. Oftentimes, especially for full product style interviews, this maps directly to the functional requirements you've already identified (but not always!). You will use this contract to guide your high-level design and to ensure that you're meeting the requirements you've identified.

You have a quick decision to make here -- which API protocol should you use?

**REST (Representational State Transfer)**: Uses HTTP verbs (GET, POST, PUT, DELETE) to perform CRUD operations on resources. This should be your default choice for most interviews.

**GraphQL**: Allows clients to specify exactly what data they want to receive, avoiding over-fetching and under-fetching. Choose this when you have diverse clients with different data needs.

**RPC (Remote Procedure Call)**: Action-oriented protocol (like gRPC) that's faster than REST for service-to-service communication. Use for internal APIs when performance is critical.

For twitter we can define them as --

```cpp
POST /v1/tweets
body: {
  "text": string
}

GET /v1/tweets/{tweetId} -> Tweet

POST /v1/users/{userId}/follows

GET /v1/feed -> Tweet[]
```

Notice how we use plural resource names (tweets, not tweet) and put the userId in the path for the follow endpoint since it's required to identify which user to follow. The tweet endpoint doesn't include userId because we get that from the authentication token in the request header.

Never rely on sensitive information like user IDs from request bodies when they should come from authentication. Always authenticate requests and derive the current user from the auth token, not from user input.

## Data flow:

Sometimes for data intensive applications we might have to perform the long steps on input to get desired output

For a web crawler, this might look like:

1. Fetch seed URLs
2. Parse HTML
3. Extract URLs
4. Store data
5. Repeat

## High level design:

Now that you have a clear understanding of the requirements, entities, and API of your system, you can start to design the high-level architecture. This consists of drawing boxes and arrows to represent the different components of your system and how they interact. Components are basic building blocks like servers, databases, caches, etc


Focus on a relatively simple design that meets the core functional requirements, and then layer on complexity to satisfy the non-functional requirements in your deep dives section. It's natural to identify areas where you can add complexity, like caches or message queues, while in the high-level design. We encourage you to note these areas with a simple verbal callout and written note, and then move on.

For instance following can be the good design

![[Pasted image 20251010143216.png]]

## Deep dive:

Now its time to review application and see all the non functional requirements one by one and make them better.

So for example, one of our non-functional requirements for Twitter was that our system needs to scale to >100M DAU. We could then lead a discussion oriented around horizontal scaling, the introduction of caches, and database sharding -- updating our design as we go. Another was that feeds need to be fetched with low latency. In the case of Twitter, this is actually the most interesting problem. We'd lead a discussion about fanout-on-read vs fanout-on-write and the use of caches.