
---

Scalability means application can handle greater loads by adapting. 
There are two kinds of scalability in cloud

- Vertical scalability (Increase the size of instance)
- Horizontal scalability (More instances) - Implies distibuted system.

**High availability** means running application in atleast two different in atleast 2 different zones. The goal is to survive the data center loss. 

Elasticity means weather you are able ton quickly increase or descrease the load or not. 

Durability means how much data is lost for example if say have durability of x% then out of 100 objects 100-x will be lost each year. 

Availability means how readily service is available. 


## ELB Elastic load balancer

Load balancer are the servers that forward internet traffic to multiple ec2 instances downstream. IT can be used to spread the load on multiple servers. They can provide the ssl certificate for webiste through load balancers. They provide high availablity - across multiple az. 

ELB is managed load balancer. There are 4 kinds of load balancers in aws

1. Application load balancer - Layer 7 (http/https)
2. Network load balancer - (ultra high performance) L4
3. Gateway load balancer - Layer 3
4. Classic load balancer - L4-L7 (Retired)

All load balancers are part of **Elastic Load Balancing (ELB)**.

#### **Application Load Balancer (ALB)**

**Layer 7 (HTTP/HTTPS) load balancer**

Key features:
- Content-based routing (URL, Host, Path, Query)
- Supports **WebSocket**
- Supports **HTTP/2**
- Ideal for microservices & container apps (ECS/EKS)
- Works with target groups

#### **Network Load Balancer (NLB)**

**Layer 4 (TCP/UDP/TLS) load balancer**

Key features:
- Extremely high performance (millions of requests/sec)
- Ultra-low latency
- Static IP & Elastic IP support
- TLS offloading

#### **Gateway Load Balancer (GWLB)**

**Layer 3 (IP) load balancer + transparent proxy**

 Key features:
- Used for **security appliances**
- Sends traffic to 3rd-party firewalls or IDS/IPS
- Uses **GENEVE** protocol for encapsulation

To connect and route the traffic from Internet to down services we need to create the group to which  this load balancer will route the traffic to. 

Auto scaling group is created to scale out (add ec2)/scale in (remove ec2) to match the load while insuring we have minimum and maximum number of matched instances. 

An **Auto Scaling Group (ASG)** in AWS is a service that automatically manages the number of EC2 instances in your application to maintain performance, availability, and cost-efficiency. Instead of manually launching or stopping servers, an ASG continuously monitors your environment and adjusts the capacity based on real-time demand. This ensures that your application always has the right amount of compute power—no more, no less.

At its core, an Auto Scaling Group works by defining a **minimum**, **maximum**, and **desired** number of EC2 instances. The _minimum_ capacity ensures the application always stays up, even during failures. The _maximum_ capacity prevents costs from growing uncontrollably. The _desired_ capacity is the target number of instances that should normally be running. When traffic increases and existing instances cannot handle the load, the ASG automatically adds new instances. When traffic decreases, it terminates unnecessary instances to save cost.

ASGs rely heavily on **scaling policies** to make these decisions. Scaling can be triggered by metrics such as CPU utilization, memory usage (via CloudWatch custom metrics), network traffic, request count, or even application-level health checks. They also support **scheduled scaling**, where you can scale out or scale in at specific times—useful for predictable traffic patterns like office hours or nightly batch processing. AWS also provides **predictive scaling**, which uses machine learning to forecast traffic and pre-scale the infrastructure before load spikes occur.

Another critical component of Auto Scaling Groups is **health checks**. ASGs monitor the health of each instance using EC2 status checks or Elastic Load Balancer health checks. If an instance becomes unhealthy—due to OS issues, application crashes, or network failures—the ASG automatically replaces it with a new one. This self-healing capability ensures high availability without manual intervention.

Auto Scaling Groups are commonly used with **Load Balancers**. When used with an Application Load Balancer or Network Load Balancer, new instances launched by the ASG are automatically registered to the load balancer’s target group. This means incoming traffic is immediately routed to healthy and freshly launched instances. Together, ASG + Load Balancer creates a fully elastic, highly available, and fault-tolerant architecture.

Overall, Auto Scaling Groups help applications handle unpredictable workloads while minimizing operational effort and cost. They combine monitoring, automation, health management, and elasticity into one powerful system that keeps your application resilient and responsive under all conditions.

To have the autoscaling group we need to have an launch template which is set of config used to create ec2. 

A **target** is simply a **destination** that receives traffic from an AWS load balancer.

## Scaling - 

Scaling strategies in AWS determine **how and when** your application infrastructure should increase or decrease its capacity. These strategies ensure that your system can handle fluctuations in traffic while optimizing performance and cost. Instead of manually adjusting the number of servers, AWS offers automated and intelligent scaling methods integrated with Auto Scaling Groups, ECS, Lambda, and many other services.

One of the most commonly used strategies is **Reactive (Dynamic) Scaling**, where the system responds to real-time metrics. For example, if CPU utilization goes above 70%, the Auto Scaling Group automatically launches new EC2 instances. When traffic falls, it terminates them. This approach ensures that your application always has the right amount of resources, reacting appropriately to changing workload trends. However, reactive scaling can sometimes lag during sudden traffic spikes because it waits for metrics to cross thresholds.

To address predictable or known workload patterns, AWS provides **Scheduled Scaling**. This strategy allows you to scale your resources at fixed times based on expected demand. For instance, if you know your application receives peak traffic during business hours, you can schedule a scale-out at 9 AM and scale-in at 7 PM. Scheduled scaling is ideal for workloads with consistent daily or weekly patterns, ensuring cost efficiency and readiness without waiting for alarms to trigger.

Another advanced approach is **Predictive Scaling**, where AWS uses machine learning to analyze historical traffic patterns and forecast future demand. Based on these predictions, AWS proactively adjusts your infrastructure before the load increases. This results in smoother performance during sudden surges and eliminates delays seen in reactive scaling. Predictive scaling is especially valuable for applications with cyclical traffic, such as retail websites or event-driven platforms.

AWS also supports **Target Tracking Scaling**, which is the easiest and most recommended strategy for most workloads. With target tracking, you simply define a metric target—like keeping CPU utilization at 50%—and AWS automatically increases or decreases capacity to maintain that level. It behaves similarly to how a thermostat maintains room temperature. This strategy simplifies scaling configurations while maintaining consistent performance.

Lastly, **Step Scaling** provides more granular control. Here, scaling actions depend on the severity of metric breaches. For example, if CPU goes above 60%, add 1 instance; if it exceeds 80%, add 3 instances. Step scaling allows a more tailored response compared to simple threshold-based scaling and is useful for workloads with varying levels of demand intensity.


