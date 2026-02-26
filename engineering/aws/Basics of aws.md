
---

A **Managed Service** in AWS is a service where **AWS takes care of the operational responsibilities**, so you don’t have to manage the underlying infrastructure.

AWS handles things like:

- Server provisioning
- Maintenance
- Patching
- Scaling
- Backups
- High availability
- Security of the underlying infrastructure


AWS has a ui console website to engage with services. 
Cloud computing is the on demand delievery of computer power, database storage and other IT resources. They follow pay-as-you-go pricing. We don't have to may for upfront cost of server. 

Type of cloud computing 

1. IaaS - Infrastructure as service 
It provides building blocks for cloud IT. Provides networking, computer and data storage space. This is an alternate to on premise.
2. Paas Platform as a service 

Removes the need to for organization to manage the underlying infra. Focus is only on the deployment and management of application.

3. Saas Software as service - So entire application software is available as a service for us to use. Example chatgpt , gmail.

The difference between Iaas and Paas is that - Infra as service provides the raw hardware as service so we have a server, sotrage and network managed by others but we need to manage the OS, middleware and runtime ourself. In Paas the aws  will manage in addition to this Runtime and OS and our only focus remains is to mange the application only. 


## Public vs private vs hybrid cloud

Public **Cloud hosted & maintained by a third-party provider (AWS, Azure, GCP, etc.)**  
Resources (compute, storage, networking) are shared across multiple customers in a secure, isolated way.

Private **Cloud infrastructure owned by a single organization** (on-premise or hosted).  
Examples: VMware, OpenStack, IBM Cloud, On-Prem Data Centers with virtualization.

Hybrid cloud **A combination of Public Cloud + Private Cloud**, working together.  
Data and applications can move between environments.


| Feature       | Public Cloud       | Private Cloud       | Hybrid Cloud            |
| ------------- | ------------------ | ------------------- | ----------------------- |
| Ownership     | Cloud provider     | Single organization | Both                    |
| Cost          | Low, pay-as-you-go | High upfront cost   | Medium                  |
| Scalability   | Very high          | Limited by hardware | High                    |
| Security      | Good               | Very high           | Very high               |
| Management    | Provider           | Customer            | Shared                  |
| Control       | Low                | High                | Medium                  |
| Best Use Case | Startups, SaaS     | Govt, banks         | Enterprises, transition |

