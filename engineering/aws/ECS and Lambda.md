
---

Amazon ECR(Elastic container registry) is private registry for docker images. ECS stants for elastic container service. 

Amazon ECS (Elastic Container Service) provides the managed orchestration layer that solves these problems by automating how containers are deployed, scheduled, scaled, and monitored. ECS abstracts away the complexity of operating container clusters and ensures reliable execution of Docker containers using AWS-managed infrastructure. You can run ECS on **EC2** (you manage the servers) or **Fargate** (serverless containers with no infrastructure management). ECS handles service discovery, load balancing, task placement, auto scaling, health checks, and rolling updates—giving developers a simple, reliable, and deeply integrated AWS-native way to run containerized applications at scale.

With fargate it is serverless and we not need to provision servers. 

EKS is elastic kubernetes service and allows us to manage kubernetes cluster on aws. It is an open source system for managing ,deploying and scaling of applications. 

## Serverless

In a serverless model, the cloud provider automatically provisions, scales, and maintains the compute resources. Developers only write and deploy code or containerized tasks, and the platform runs them on demand.

### AWS lambda

With Lambda, you simply upload your function code, and AWS automatically runs it in response to events such as API requests, S3 uploads, DynamoDB changes, or scheduled triggers. There are **no servers to manage**, no capacity planning, and no idle costs—Lambda scales instantly from zero to thousands of concurrent executions and charges only for the compute time actually used. It is reactive kind of service. 
Lambda can run the containers as well but it is recommended to use ECS/Fargate. Also lambda functions have limited time of invokation. We can simply give it code it is fully serverless and we pay only for the use we do. 

## API gateway

Amazon API Gateway solves this by providing a fully managed service that handles the entire lifecycle of APIs. It allows developers to create, publish, secure, monitor, and scale REST, HTTP, and WebSocket APIs without managing servers. API Gateway integrates seamlessly with AWS Lambda, ECS, EC2, DynamoDB, and other backend services, acting as the **front door** for applications. It automatically takes care of authentication (IAM, Cognito, OAuth tokens), throttling, caching, traffic management, request/response transformation, and API versioning. By offloading all API management responsibilities, API Gateway lets teams focus on backend logic while ensuring high-performance, secure, and globally scalable APIs.

### AWS batch 

AWS Batch provides a fully managed service that automates the provisioning, scheduling, and scaling of compute resources for batch workloads. You submit jobs, define their requirements (CPU, memory, GPU), and AWS Batch takes care of everything else—creating EC2 or Fargate compute environments, optimizing job placement, handling retries, managing queues, and scaling infrastructure dynamically based on demand. Because it integrates natively with containers, AWS Batch enables you to package your workload once and run it reliably at any scale. This eliminates the need for managing servers or batch schedulers, reduces cost by dynamically using Spot instances, and allows engineers to focus on the job logic rather than infrastructure management.
Batch has no run time limit and runs on top of ec2. 

### Lightsail 

Amazon Lightsail solves this by providing a simplified, beginner-friendly cloud platform with fixed monthly pricing and preconfigured environments. Lightsail offers easy-to-launch virtual servers, managed databases, containers, load balancers, object storage, and even one-click application stacks like WordPress, Node.js, LAMP, and Django. It abstracts away the complexity of VPC networking, security groups, and scalable configurations, giving users a straightforward experience similar to traditional VPS providers.

It has high availability but little to no auto scaling.

### AWS code build

Used to build and deploy the code. 
AWS CodePipeline provides a fully managed **continuous integration and continuous delivery (CI/CD)** service.

Codeartifact - 

It is used for artifact management

