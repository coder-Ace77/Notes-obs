
---

Entire AWS infra is divided into seveeral regions. Each region is composed of atleast 3 availability zones and each AZ is completely isolated from each other. Each AZ is made up of multiple datacenters. Datacenters finally house the actual hardware. 
Edge locations are used to deliever the content as fast as possible may or may not contain entire set of services available. 

Route 53 is managed dns. DNS is a collection of rules and records which helps clients understand how to reach a server through URLs.

In AWS, the most common records are:
• www.google.com => 12.34.56.78 == A record (IPv4) 
• www.google.com => 2001:0db8:85a3:0000:0000:8a2e:0370:7334 == AAAA IPv6 
• search.google.com => www.google.com == CNAME: hostname to hostname
• example.com => AWS resource == Alias (ex: ELB, CloudFront, S3, RDS, etc…)


![[Pasted image 20251121174606.png]]

Simple Routing policy - The need arises when you want a straightforward DNS resolution with **one domain → one endpoint**. Simple routing provides exactly that. It returns a single record with a single IP or resource. This is useful for small applications or when there’s only one backend service. No load balancing or health checks are involved.

Weighted routing - When teams want to perform controlled traffic distribution—such as load balancing across resources, testing new versions (canary releases), or gradually shifting users—they need a way to split DNS traffic. Weighted routing solves this by letting you assign weights (like 70–30 or 50–50) to different endpoints. Route 53 returns DNS responses based on these weights, enabling smooth traffic control and experimentation.

Latency routing policy- Global applications need users to connect to the **closest** and **fastest** region. Routing everyone to a single region increases latency and degrades user experience. Latency-based routing solves this by routing each user to the AWS region that offers the **lowest latency** from their location. This significantly improves speed and performance.

Failover roting policy-High availability requires automatic recovery when the primary environment goes down. Traditional DNS systems cannot switch traffic based on health. Failover routing solves this by setting up **primary** and **secondary** records with health checks. If the primary endpoint fails, Route 53 automatically routes traffic to the secondary, providing seamless disaster recovery.


## Cloud front - 

AWS cloud front is content delievery network.Improves read performance, content is cached at the edge.
DDoS protection (because worldwide), integration with Shield, AWS Web Application Firewall.

Cloud front origins - 

- S3 bucket - for the content
- VPC origin - Applications hosted. 
- Custom Origin - a s3 website. 

![[Pasted image 20251121175318.png]]

### AWS global accelarator-

Improve global application availability and performance using the AWS global network
•Leverage the AWS internal network to optimize the route to your application (60% improvement) 
•2 Anycast IP are created for your application and traffic is sent through Edge Locations 
•The Edge locations send the traffic to your applicatiom

### AWS outpost

Hybrid cloud is when client keeps the on premises with the cloud infrastructure. They have two systems - 

- One for AWS cloud
- Other for on premise infrastructure

AWS outposts are server racks that provide the services like of on premises in your own building. 

### AWS wavelength

AWS Wavelength extends AWS services to the **edge of 5G networks** by embedding AWS compute & storage inside telecom operator facilities (called _Wavelength Zones_).  
These zones sit very close to mobile users → drastically reducing latency compared to regular AWS Regions.

Normal Path (Without Wavelength)

Mobile user → 5G Network → Internet backbone → AWS Region  
Lots of hops → **20–100 ms latency**

With Wavelength

Mobile user → 5G Network → AWS Wavelength Zone (inside the telecom network)  
No internet travel → **1–10 ms latency**

A Wavelength Zone is similar to an AZ but located **inside** the data center of telecom providers (like Verizon, Vodafone, KDDI).

**AWS Local Zones** are AWS infrastructure deployments **closer to major population/metro areas** to provide **low-latency access (single-digit millisecond)** to selected AWS services—without needing a full AWS Region.

They extend an AWS Region into a nearby city so customers can run:

- latency-sensitive applications
- local data processing
- apps requiring local compute near users or on-prem systems

Local Zones exist in major cities globally—for example, **Mumbai**, **Delhi**, **Los Angeles**, **Boston**, **Osaka**, etc.

A Local Zone is an extension of a parent AWS Region.  
You create a **Local Zone subnet** inside your VPC, and AWS automatically routes traffic to that zone.


### Global application architecture

In aws applciations can be deployed in many ways- 

- First one is Single region and single zone- Difficulty is easy but availability is low and global latency is high. 
- Second is Single region and multi az - High availability , But global latency is bad
- Multi region but read only in all regions and write in one region
- Multi region with both read and write in all region provides very low latency but difficult to implement


### Amazon SQS

- Oldest AWS offering (over 10 years old)
- Fully managed service (~serverless), use to decouple applications
- Scales from 1 message per second to 10,000s per second
- Messages are deleted after they’re read by consumers 
- Is is async communation pattern

Amazon SQS is fully managed queue service. 

### Amazon Keinesis

**Amazon Kinesis** provides a fully managed platform for **real-time data streaming and processing** at massive scale. It allows you to collect, process, and analyze streaming data continuously, rather than waiting for batches. Kinesis automatically handles scaling, fault tolerance, and durability, so applications can produce and consume data instantly with sub-second latency. It integrates seamlessly with AWS Lambda, S3, Redshift, DynamoDB, and analytics tools, enabling complete end-to-end real-time pipelines.

Keinesis data stream - A service for ingesting and processing real-time data streams.  
You create a stream, divide it into _shards_, and producers push data while consumers process records in real time.

Keinesis data firehose - A fully managed delivery service for sending streaming data → S3, Redshift, OpenSearch, or Splunk.


### Amazon SNS

**Amazon Simple Notification Service (SNS)** provides a fully managed **pub/sub messaging service**. With SNS, publishers send a message to a _topic_, and SNS automatically delivers it to all subscribed endpoints—such as email, SMS, mobile push notifications, SQS queues, HTTP endpoints, or Lambda functions. AWS handles all infrastructure, retries, scaling, and delivery guarantees. It can process millions of messages per second, making it ideal for sending alerts, broadcasting updates, or triggering distributed workflows.

Note the differnece between SQS and SNS is that SQS where Consumers _pull_ messages from the queue when ready.
SNS _pushes_ messages to endpoints (email, SMS, Lambda, HTTP, SQS, etc.).

Secondly 

**Fan-out delivery**: one message → many receivers.No message storage; messages are delivered instantly.

**One-to-one delivery**: each message is processed **by only 1 consumer**.Message is stored until processed or deleted.

Third in SQS message remain until explicitely deleted while in SNS messages are not stored. 

### Amazon MQ 

It is an managed message broker service for 

- Rabbit MQ
- Active MQ
It does not scale as much as SQS/SNS
