
---

EC2 instance gets a new a public IP address every time you stop then start it (default). Private IPv4 – can be used on private networks (LAN) such as internal AWS networking (e.g., 192.168.1.1).Elastic IP – allows you to attach a fixed public IPv4 address to EC2 instance

Note that every IPv4 address is charged at 0.005 dollars per hour. IPv6 are free. 

VPC - Virtual Private Cloud: private network to deploy your resources (regional resource)
Subnets allow you to partition your network inside your VPC (Availability Zone resource) 
A public subnet is a subnet that is accessible from the internet 
A private subnet is a subnet that is not accessible from the internet 
To define access to the internet and between subnets, we use Route Tables.

An **Internet Gateway (IGW)** is a horizontally scalable router that allows **two-way communication** between a VPC and the internet.

Sometimes private subnets need **outbound internet** access (e.g., downloading patches, hitting external APIs), but **must not** be reachable from the internet.

To make any subnet public we need to connect a subnet to Internet gateway which helps us get connected to the internet.  

A **NAT Gateway** provides **outbound-only** internet access for instances in private subnets.

Note that there are two NATs - 

1. NAT gateway - managed by aws
2. NAT instances - managed by us

AWS uses standard IPv4 and IPv6 addressing based on **CIDR (Classless Inter-Domain Routing)** notation to define IP address ranges inside Virtual Private Clouds (VPCs) and subnets. CIDR blocks identify the network portion and host portion using a suffix such as `/16`, `/20`, or `/24`.

The **prefix length** (`/16`) tells how many bits represent the network.

A **smaller prefix** (e.g., /16) → **more IPs**
Each subnet reserves:

**5 IPs for AWS**
- Network address
- Broadcast address (still reserved even though AWS doesn’t use broadcast)
- VPC router
- DNS resolver
- Future AWS use

NAT Gateways (AWS-managed) & NAT Instances (self-managed) allow your instances in your Private Subnets to access the internet while remaining private

#### NACL (Network ACL)

•A firewall which controls traffic from and to subnet 
•Can have ALLOW and DENY rules •Are attached at the Subnet level 
• Rules only include IP addresses •
#### Security Groups 

• A firewall that controls traffic to and from an EC2 Instance 
• Can have only ALLOW rules
• Rules include IP addresses and other security groups

![[Pasted image 20251121193018.png]]

VPC flow logs can help us to catch info about the IP traffic going in to any of interfaces. VPC Flow logs data can go to S3, CloudWatch Logs, and Amazon Data Firehose

### VPC peering 

Connect two VPC, privately using AWS’ network. Make them behave as if they were in the same network. Must not have overlapping CIDR (IP address range) 
VPC Peering connection is not transitive (must be established for each VPC that need to communicate with one another)

VPC endpoints allows users to connect the AWS services using private network instead of public. Gives better security and latency. 

VPC endpoint gateway - USed for S3 and dynamo db - Global resources. 
VPC endpoint interface - Most services including S3 and dynamo db.

### Private link 

In many architectures, applications running in one VPC need to securely access services hosted in another VPC or in AWS services (like S3, EC2 APIs, etc.).

Organizations needed a way to connect **privately** to AWS services or third-party services without leaving the AWS backbone.
AWS PrivateLink enables **private connectivity** to AWS services, partner services, or your own services **via private IPs inside your VPC**.
So the third party vendor needs to create netwrok load balancer(L4) and you in your private vpc must have ENI(elastic network interface)

### AWS site to site vpn 

Companies often run hybrid networks where some systems remain in on-premises data centers.  They need a **quick, secure** way to connect their on-prem network with AWS.

AWS Site-to-Site VPN provides an **encrypted IPSec tunnel** over the public internet between:

- Your on-prem router
- The AWS Virtual Private Gateway (VGW) or Transit Gateway

So on prem site we must have Customer gateway and on AWS side we must have Virtual private gateway. 

### AWS direct connect (DX)

It provides a direct physical connection between on primese and aws. It goes over private network.

### AWS client vpn - 

It allows once computer to get connected using OpenVPN to your private netwrok on AWS and on-premises. So It allows one person to connect to ec2 instances in private ip. 

### Transit gateway - 

For having transitive peering between thousands of VPC and on-premises, hub-and-spoke (star) connection
One single Gateway to provide this functionality 
Works with Direct Connect Gateway, VPN connections

