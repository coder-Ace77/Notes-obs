
---

ec2 = elastic compute cloud = infra as service
EC2 is bound to an availabilty zone. 
ec2 is a composition of many services - 
1. Renting a vm(ec2)
2. Storing data on virtual drives(ebs)
3. Distributing load across various machines(ELB)
4. Autoscaling

We can configure our ec2 with

1. OS - Linux, mac or Windows
2. How much compute (cores)
3. How much RAM
4. How much storage space? There can be two kinds of storage spaces available - One is that is available over internet like EBS(elastic block storage) or Elastic file storage. Other variation is hasdware like EC2 instance store. 
5. Network card - speed of card, public ip
6. Firewall rules - security group
7. Bootstrap script - This is meant to be run only once the machine starts. bootstraping means leanunching commands once the machine starts. Script runs only once when the instance starts. This file is used to make the server have machine ready with software you write. It is also called user data. 


AWS ec2 instances  - 

There are seven families of aws ec2 instances they define the compute and ram of virtual server. 

1. General purpose - Balanced compute , memory and networking. Best for standard applications. t,m,a series
2. Compute optimised - High-performance **vCPUs** for compute-heavy applications. c series
3. Memory optimised - Designed for **in-memory processing**, huge RAM needs. r series , x series
4. Storage optimised - High **IOPS, SSD/NVMe**, optimized for disk-intensive workloads. i,D series 
5. Accelerated compute - Use GPUs, FPGAs, ML accelerators. G series 

Use cases  - 

1. General purpose are good for medium work loads. High compute optimised are good for high performance webservers. 
2. Memory optimised are good for databases, distributed caches etc, In memory databases
3. Storage optimised - Again for databases as provide high i/o
4. File systems

General naming `instanceclass generation.size` eg `t2.micro` t is instance class 2 is generation and micro is size.

## Security groups

### **Stateful Firewall at the Instance level**

Security Groups act as **virtual firewalls** for:

- EC2 instances
- RDS
- Lambda in VPC
- Load balancers

characteristics

| Feature                | Description                                                       |
| ---------------------- | ----------------------------------------------------------------- |
| **Stateful**           | If inbound is allowed, response outbound is automatically allowed |
| **Attached to ENI**    | SGs are bound to Network Interface, not the VPC                   |
| **Only Allow Rules**   | No deny rules                                                     |
| **Evaluated together** | If ANY SG allows → traffic is allowed                             |
Security group define which data can come in and out of the EC2 instances. They only contain allow rules. By default all the inbound traffic is blocked and all the outbound traffic is allowed. 
A security group can be attached to multiple ec2 and one ec2 can address multiple security groups. 
Most of the times if we try to access the ec2 and its blocked at firewall level then timeout will happen. 
Note- 0.0.0.0/0 means all ips. 

Ports - 

1. 22 - ssh 
2. 21 - ftp
3. 22  - sftp - Secure file transfer protocol - upload file using ssh
4. 80 - http - access unsecured website
5. 443 - https

Inbound rules - Defines the rules which state the connectivity from outside. 
Outbound rules - Defines the traffic rules followed when transferring data from inside. 

Since rules are addonly so two different security group can never contradict. At most there will be the case where one rule is not in other and in this case it will be additive. 

We can connect to our machine using ssh

```
ssh -i Key.pem ubuntu@ip 
```

We can also use ec2 connect to connect to an ec2 instance. 

Now IAM role can be given to an ec2 instance for example to make it to connect to block storage. 

## Purchase options

On demand - short workload, predictable pricing , pay by second
Reserved (1 and 3 years)
- Reserved instances - long workloads but can not change the type
- Convertible reserved instances - long workloads with flexible instances. 
Saving plan (1 and 3 years)
Commitment to a specific amount of usage, long workloads - instance family is locked but we can change between size. 
Spot instances - short workloads , cheap and can loose instances (less reliable) - we define the max price we are willing and as soon as price goes above we will loose the vm. 
Dedicated hosts - book and entire physical server  - most expensive
Dedicated instances - No other customer sharing same hardware. 

Additinally we can reserve the capacity in an AZ.

## Shared responsibility model - 

AWS is responsible for security and isolation on physical host if dedicated host. Compliance validation and replacing faulty hardware. 

However as the user we are responsible for the security in cloud. We define security group rules. OS patches and updates. 

# Storage options for EC2

## EBS - Elastic block volume- 

It is a network drive the can get attached to an ec2 instance and allows us to persist data even after termination. They can only be mounted to one instance at a time.EBS volume is specific to an availablity zone. Essentially is a distributed storage. EBS can be detached and can be attached to another ec2. It is a volume so the storage space needs to be specified first but can be increased later on. Note that is it possible to attach two ebs volumes attach to single ec2 instance. Snapshots can be taken to S3. **The root volume of almost all EC2 instances is an EBS volume.**

Just like ec2 is bounded to an az ebs is also bounded to an az. EBS volume can be deleted on termination of ec2 instance. Now we know that ec2 can use two kinds of storages one is EBS other is Instance store which (Ultra-fast temporary disk attached to the physical host — data is lost if you stop/terminate the VM) Note that it is available only on small number of machines and is used as the cache.

### Internal EC2 model

When you launch an EC2 instance:

1. AWS provisions a VM on a hypervisor (Nitro system)
2. A **network interface (ENI)** is created
3. An **EBS root volume** is attached over a high-speed network
4. The VM uses the EBS volume as its OS disk  
    (this is why EBS is “block storage”)
5. Optional: AWS also attaches **instance store** disks if the instance type includes them

AWS Nitro offloads:
- networking
- storage virtualization
- security

This is why EBS volumes look like local SSDs but are actually network devices.

### When you **STOP** an EC2:

- EBS volume persists (keeps data)
- Instance store is wiped
- VM is moved to another host

### When you **TERMINATE** an EC2:

- EBS may or may not persist  
    (depends on “DeleteOnTermination” flag)
- Instance store is always lost


| Type of data                        | Where to store?   |
| ----------------------------------- | ----------------- |
| Application code, OS, logs          | EBS               |
| Database files                      | Always EBS or EFS |
| Temporary processing (Spark/Hadoop) | Instance Store    |
| Caches                              | Instance Store    |
| Shared storage across instances     | EFS               |
| Object storage                      | S3                |

EBS snapshot is the backup of an EBS volume at any point of time. It is not necessary to detach an EBS volume is recommended to do so. Snapshots can be copied across AZ or regions.

EBS snapshot archieve - 75% cheaper than normal. Takes 24-72 hours to restore. We can also configure the recycle bin for snapshots. 

Volume can be recreated from an snapshot. 

## AMI


An **Amazon Machine Image (AMI)** is a **template used to launch EC2 instances**.  
It contains everything needed to boot an instance.

Think of an AMI as a **blueprint** for your virtual machine.
An AMI includes:

 1️⃣ **Operating System**
- Linux (Ubuntu, Amazon Linux, RedHat, etc.)
- Windows Server

 2️⃣ **Application Server & Software**

- Pre-installed packages (Python, Java, NGINX, Docker, etc.)

3️⃣ **Application Code (optional)**

- Your custom app baked into the image.

4️⃣ **Launch Permissions**

- Controls who can use the AMI (your account only / public / specific AWS accounts).

 5️⃣ **Root Volume Snapshot**

- Backed by EBS or Instance Store.



There are three kinds of AMIs

**1. AWS-Provided AMIs**

- Official, secure, frequently updated.
- Example: Amazon Linux 2023, Ubuntu 22.04.

**2. Marketplace AMIs**

- Provided by third-party vendors (e.g., WordPress, Jenkins, Fortinet Firewall).

**3. Custom AMIs**

- Created by **you** from an existing EC2 instance.
- Used for:
    - Preconfigured environments
    - Scaling groups
    - Backup of server configuration

We can direclty create the AWS ec2 instance from an ami.

## EC2 image builder

Used to automate the creation of virtual machines or container images. **EC2 Image Builder** is an AWS service that **automates the creation, patching, testing, and distribution of AMIs** (Amazon Machine Images) and container images.

Instead of manually creating AMIs, you can **build secure, up-to-date images automatically** on a schedule.

### Without Image Builder:

- You manually patch servers
- Create AMIs by hand
- Install packages manually
- Risk inconsistent images
- Hard to automate updates

### With Image Builder:

- Automated image creation
- Automatically applies OS patches + app updates
- Automatically tests images (integrity, security)
- Distributes images across regions
- Version control for images


It uses a **pipeline** with:

1️⃣ **Source Image**
- Your base AMI (e.g., Amazon Linux, Ubuntu).

2️⃣ **Components**
Re-usable steps:
- Install packages
- Apply security patches
- Configure OS
- Run scripts
- Hardening policies (CIS benchmark)

3️⃣ **Build Phase**
It launches a temporary EC2 instance → applies components → creates AMI.

 4️⃣ **Test Phase**
Runs test scripts to ensure image works.


## EC2 instance storage

**Instance Store** is **temporary, high-performance storage physically attached to the EC2 host machine** (the physical server).  
It is also called **Ephemeral Storage**.

Think of it as the **local SSD/HDD** of the physical server where your EC2 instance is running.

It is very fast however data is lost when the ec2 stops.

## EFS Elastic file system

As known EBS can be attached to one Ec2 at a time. EFS is a distributed file system and can be attached to many EC2's at a time. 

**Amazon EFS (Elastic File System)** is a **serverless, fully managed, shared file system** for Linux workloads.
Think of it like a **shared folder** that multiple EC2 instances can access **at the same time**.

It provides:

- **NFS-based file storage**
- **Automatic scaling** (up & down)
- **Pay-per-use pricing**

Many EC2 instances across **multiple Availability Zones (AZs)** can mount the same EFS filesystem simultaneously.EFS grows and shrinks **automatically** as files are added/removed.  
No provisioning like EBS.

Stores data across **multiple AZs**, designed for **99.999999999% (11 9’s)** durability. Note that it works only with linux based amis as it uses NFSv4. 
Finally note that EFS is very expensive. 

| Feature    | **EFS**            | **EBS**                | **S3**                |
| ---------- | ------------------ | ---------------------- | --------------------- |
| Type       | File system        | Block storage          | Object storage        |
| Access     | Multi-instance     | One instance at a time | Global via API        |
| Scaling    | Auto               | Must choose size       | Auto                  |
| OS Support | Linux              | Linux/Windows          | Any                   |
| Durability | Very high          | High                   | Highest               |
| Use Case   | Shared file system | Single VM disk         | Backups, static files |
EFS also has tire of support for example 
EFS-IA - Infrequent access Storage class optimised for files that are not accessed everyday. They are cheaper. 

## Shared reponsibility 

AWS is repsonsible for - 

1. Infrastructure
2. Replica for data for EBS volumes and EFs drives. 
3. Replaication of faulty software

As customer our reponsibility

1. Setting data encyption
2. Setting backup/ snapshot

## Amazon FSx

**Amazon FSx** is a family of **fully managed, high-performance file systems** designed for enterprise workloads.

Unlike EFS (which is Linux + NFS + simple), **FSx supports advanced, specialized file systems** used in Windows, HPC, ML, and enterprise apps.

Amazon Lustre is used to have very high performance.

