
---

Cloud watch provides metric for every service in AWS. Metric is a veriable to monitor(CPU utiilisation, Networking etc)
We get the default metrics every 5 mins and custom metrics can also be done. 

Cloud watch alarms are used to trigger notification for any metric. So once a metric goes above a threshold we can trigger a cloud watch alarm. 

Then there are alarm actions which can be what the thing to do when metric is reached. 
We can also create billing alarm to see the bills. 

Cloud watch logs can collect logs from elastic beanstalk , ecs , cloudtrail etc. It allows real time monitoring. 

Amazon event bridge (formely called cloud watch events) is a fully managed **serverless event bus** that makes it easy to connect applications using events. 
EventBridge lets you publish events from your app or from AWS services and automatically route them to targets such as Lambda, Step Functions, SQS, SNS, Kinesis, API destinations, and many others. 
It uses schema-based routing rules, supports filtering, and handles retries, error handling, and high availability. 
With EventBridge, services communicate asynchronously and event-driven architectures become easy to build and scale.

Event bus is a logical channel where events flow.
Events themselves are JSON objects.
Rules define **what events to listen to** and **where to send them**.

AWS CloudTrail is a fully managed service that records, stores, and audits activity across your AWS account. It captures every API call made by users, roles, services, or applications, both from the AWS Management Console and programmatically through SDKs or CLI. CloudTrail enables complete visibility into “who did what, when, and from where” within your AWS environment. This centralized logging capability is essential for governance, compliance, operational troubleshooting, and security analysis.

AWS codeguru is automated code review and application performance recommendations. 

The AWS Health Dashboard is a centralized interface that provides real-time visibility into the operational status of AWS services and resources. It offers two views: a public-facing, service-wide status dashboard, and a personalized account-level dashboard. The service helps customers understand how ongoing AWS events—such as outages, maintenance activities, or performance degradations—may impact their workloads. Its primary goal is to increase transparency, provide timely notifications, and enable faster troubleshooting during AWS service disruptions.