
---

AWS shared responsibilty model states that is the collective reponsibilty of AWS and customer to secure entire applciation. 

AWS is responsible for the security of the cloud. It protects the cloud infra and all the managed services. 

Customer responsiliry is to manage the scurity in the cloud. So for example maanging the security of OS patch updates etc. IAM and firewall and also encryption of data. 

Shared controls is where both need to collaborate - This is patch management and configuaration management. 

### DDOS attack protection

A Distributed Denial-of-Service (DDoS) attack aims to overwhelm an application or server by sending a massive number of requests from distributed sources (botnets). When the target server cannot handle the load, it becomes slow, unresponsive, or completely unavailable, preventing legitimate users from accessing the service.

AWS provides multiple managed services to protect applications from such attacks, offering layered security at the network edge, application layer, and compute layer.

- An attacker controls multiple “master” servers.
- These masters command thousands of **bots**.
- All bots simultaneously send large volumes of traffic to the target application server.
- The server becomes overwhelmed and stops responding.
- Legitimate users are unable to access the application.

### AWS shield

It is a managed service having two tiers

- standard - free
- premium - 3000$ per month and has more sophisticated uses. 

Shield allows the customer to have protection from ddos attacks using following techniques-

Provides automatic protection against common network-layer attacks:

- SYN floods
- UDP reflection attacks
- Other Layer 3 / Layer 4 volumetric attacks

Now at this point is important to understand that users will be routed towards the application using route 53. Route 53 is already shield protected. The content delievered through CloudFront is also Shield protected. In case you need filtering we can use WAF(web applciation firewall) here. 

WAF is L7 and can be used only for http type and can be deployed on Application load balancer,API gateway and cloud front. 

Web access control lists are the rules that can be configured on WAF. and rules are based on - 

Rules based on:

- IP addresses
- HTTP headers
- Request body
- URL strings
- Geo-location
- Size constraints

**Rate-based rules** (e.g., block IPs making >5 requests/sec).

To serve the application we can use load balancer in the public subnet again protected by shield. Finally microservice is present in private subnet inside autoscaling gp to increase size. 

To protect the entire VPC from L3 to L7 attacks we can use AWS network firewall. Here we can filter based on - 

1. VPC to internet
2. Outbound to internet
3. Inbound 
4. Direct connect and site to site VPN

**AWS Firewall Manager** is a **centralized security management service** that allows security administrators to create, enforce, and manage firewall policies across the entire AWS Organization.

It works with security tools such as:

- AWS WAF
- AWS Shield Advanced
- VPC Security Groups
- Network Firewall
- Route 53 Resolver DNS Firewall

Firewall Manager ensures that every existing and newly created resource **automatically** complies with the security rules defined by the organization.

AWS allows its customers to do penetration testing on 8 services some of which are - EC2 instances , NAT gateways or RDS. 

Other activities like DDos or Dos are prohibited. 

## Encryption

Encryption in aws can happen at levels - 

 - Data in transit - Moving data
 - Data at rest - Archieved on device

We want both of our data to be encrypted. AWS has a service called KMS(Key maanagment service). It manages the encryption for us so we don't have keys with us. So once KMS encrpytion is opt in we have data encrpytion in -

- EBS volumes 
- S3 buckets - server side encryption of objects (SSE - S3 is enabled by default, SSE - KMS is opt in)
- Redshift - encryption of data
- RDS database 
- EFS drives - encryption of data

Some services have encryption enabled by default like cloud trail

CLOUD HSM - It is not fully managed aws just provisions the encryption hardware.  We have to manage our keys. 

Two kinds of keys - 

- customer managed keys
- aws managed keys - used by aws services like s3 , ebs etc 
- aws owned keys - owned and maanged by aws and we can't even look at them.
- Cloud HSM keys - generated from own cloud hsm 

### ACM(AWS certificate manager)

Now its time to protect our data at transit. It is done using https. ACM lets us to easily provision, manage and deploy SSL/TLS certificates. Used for providing inflight encryption. Free of charge for TLS certificate

- integrates well with 
- ELB(elastic load balancer)
- Cloud front 
- API on API gateway

### Secret manager

Newer service for storing the secrets. It has capability to force rotation of secrets we can autogenerate secrets using lambda and has integration with RDS. Secrets are encryptied using KMS. It is a paid service however. 