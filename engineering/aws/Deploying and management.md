
---

## Cloud formation

AWS CloudFormation solves this by enabling **Infrastructure as Code (IaC)**, where infrastructure is defined using templates written in JSON or YAML. These templates describe the exact AWS resources you want, and CloudFormation automatically provisions, configures, and manages them in a predictable, repeatable manner. It handles dependency ordering, rollback on failures, updates, and drift detection without any manual effort.

AWS cloud Development kit - Allows us to define cloud infra using familiar language. The is then compiled to Cloud formation template and then this template is applied to register the infra.

### Bean stack 

The **3-tier architecture** cleanly separates an application into **three logical layers**:  
1️⃣ **Presentation Tier (Frontend)** – the user interface (web, mobile, UI logic)  
2️⃣ **Application Tier (Business Logic / API Layer)** – processes data, runs application rules, exposes APIs  
3️⃣ **Data Tier (Database Layer)** – stores and manages data in databases


AWS Elastic Beanstalk provides a fully managed platform for deploying and running web applications. Developers simply upload their application code (Node.js, Java, Python, Go, .NET, PHP, Ruby, Docker, etc.), and Elastic Beanstalk automatically provisions the necessary infrastructure—EC2 instances, Auto Scaling groups, security groups, load balancers, monitoring, and environment setup. It handles application deployment, rolling updates, health checks, logs, and scaling without requiring manual configuration. At the same time, you still have full access to the underlying AWS resources if you need custom control. This blend of simplicity and flexibility makes Elastic Beanstalk ideal for quickly deploying modern web apps with minimal operational overhead.