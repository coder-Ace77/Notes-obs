
---

Software architectural patterns are the general solutions to the commonly occuring system design problems. They involve multiple components. 

## Multi tier architecture

The idea is to divide the architecture into multiple physical and logical tiers.

Logical separation - Limits the responsibility of each tier. 

Physical separation - Allows each tier to be separately-
- Developed
- Upgraded
- Scaled

Note that multilayer architecture and multitier are two different things - 

- Multilayer is used to define an application where there is an internal logical separation inside a single application into multiple layers and modules. However all the layers will be running as the single unit at runtime and this is the distinction with multi tier architecture where tiers means different run times. 

Key features 

- There are multiple tiers with certain ordering. The applications of adjacent tiers can communicate with client and server model but not allowed to skip a tier. 

### Three tier architecture

Three tiers - 

- Top tier (UI) - User interface - (presentation layer) - Client uses it to interact with the application in entirety. It normally does not have any bussiness logic. 
- Application tier / Logic tier - It provides the functionalities.
- Data tier - Responsible for the storage and persistance of user specific data. Eg - Database

It however has one major drawback. It is monolithic bussiness tier. Monolithic means each Application logic instance is bulky and needs high cpu demand and memory. 

### Two tier architecture

- Bussiness + UI layer in one application - Eg mobile application
- Data tier - 

### Four tier architecture

- Presentation layer
- API/caching/security layer
- Application layer
- Data tier

### Microservice architecture

Microservice architecture organises bussiness logic as collection of loosely coupled and independently deployed services. 

In a **microservice architecture**, instead of building one huge application, you split the system into **many small, independent services**, each responsible for **one business capability**.

So now, instead of “the shopping app”, you have:

- a **User Service**
- a **Product Service**
- an **Order Service**
- a **Payment Service**
- a **Notification Service**

Each service is like a **mini application** of its own.

Each one has:

- its **own codebase**
- its **own database**
- its **own deployment**
- its **own scaling rules**
They talk to each other over the network using APIs or messaging.

Microservices are loosely coupled but highly cohesive.

Each service does one thing well and doesn’t reach into another service’s database or internal logic. Communication happens only through **well-defined contracts** like REST, gRPC, or events.

In microservices, **each service owns its data**.

The Order Service has its own database.  
The User Service has its own database.

No service directly queries another service’s database. Ever.

Each service is built and deployed independently, often using **Docker and Kubernetes**.

## Event driven architecture

In **event-driven architecture**, systems don’t directly call each other to say _“do this now”_.  Instead, one part of the system **emits an event** saying _“this thing has happened”_.

Other parts of the system **react** to that event if they are interested.
No tight coupling. No direct dependency.

Suppose user places an order - 

The **Order Service** saves the order and then publishes an event:
**OrderPlaced**
That’s it. Order Service is done.

Now comes the **event broker** — the railway announcement system of software.

This could be:

- Kafka
- RabbitMQ
- AWS SNS/SQS
- Google Pub/Sub

The broker receives the **OrderPlaced** event and makes it available to anyone who subscribed.

The **Payment Service** is listening.
When it sees **OrderPlaced**, it processes payment and emits another event. and so on.
