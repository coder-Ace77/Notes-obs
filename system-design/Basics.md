
---

There are 6 steps for any system design

1. Requeirement engineering
2. Capacity estimation
3. data modelling
4. API design
5. System design
6. Design discussion

## Requirement engineering:

It can have 
### Functional requirements:

Functional requrirement list down the requirements of any application. They can be of two types-
core features and suport features.

### Non functional requirements:

They can be anything other than about features. However in system design they are related to scalability, performance and availability.

- scale requrirements: highly scalable
- latency : should be small
- highly available

We should be comming up with the figures for each of them.
Sometimes we also want to define the scale of system - 

For instance in todo application

Core features:
- user can create/modify/complete todo 

 Support features:
 
- Todo items are private  
- Each item has text,title and due date

Scale of system:

- 1000 active users per day
- Each user creates 5 todo per day
- Each todo has size of 20KB
- Read/write ratio is 5:1


## Data modelling:

Here we define which database should be used. But before giving out decision of database we should start with identifying entities, attributes and relationship

In todo there are two entities user and todos , each todo has many attributes text, date and description. Each user can have many todos so one to many model.


## System design:

Now in a system design we have to design the diagram keeping everything in mind. 

Service is the backbone of every arch diagram. It represnts the piece of software with clear defined purpose.
Representation layer is the front end component of application. When running multiple instances of any service we use load balancer




