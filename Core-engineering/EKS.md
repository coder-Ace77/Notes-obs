
---

EKS is a managed control plane service that means it won't manage worker nodes for you. To deploy worker node we have to go by either EC2 or Fargate. 

When application is deployed as the pod in the private subnet. When pod is deployed from the kubctl it just has a cluster ip meaning it can be accessed inside the plane however end user can not access it. 

To make it accessible from outside world we have to first make it a service 
Deploying using service we have three options:
1. deploy using cluster ip: this is default and meant only for internal communication. Kubernetes assigns a virtual IP (ClusterIP) that other Pods can use to access the service.
2. Nodeport: Here we get a mapping for each pod to node working meaning mapping the 
		port of service to port on node -> Service is assigned a port on every node.
3. Expose service using a **cloud provider’s load balancer**. A public IP is provisioned automatically by the cloud provider.

#### Ingress:

**Ingress** is an API object that manages **external access** to services within a Kubernetes cluster, typically **HTTP** and **HTTPS** traffic.

Instead of exposing every service via `NodePort` or `LoadBalancer` (which can be expensive and less scalable), **Ingress** allows you to:

- Use a **single external IP** for multiple services.
- Route traffic **based on path or hostname**.
- Handle **TLS termination** (HTTPS).
- Apply **authentication, rate limiting, etc.** via annotations/middleware.

**Ingress Resource (YAML)**
- A set of rules that define how traffic should be routed.
To work with this routing we aso need an ingress controller which is a pod that implements ingress rules.  

Clusters  are usually deployed in private subnets. 

In real schenario a pod is deployed in worker node and we also have an API server in control plane 
Now using pod we will create a service and will restrict to either service ip mode  or nodepod mod.

Ingress resource will route the traffic inside the cluster. 

ingress.yaml - this is just the route mapping for each service. 

So we will write that any incoming request must map to a service and then from service it will be mapped to a pod. And we will deploy ingress resource using kubectl however we also need ingress controller. So what happens is that this controller will watch for ingress resource and will create a loadbalancer for the same type for example if we have ELB controller then it will spawn an ELB loadbalancer and if its nginx then nginx loadbancer. Typically we put this loadbalancer in public IP. 

To interact with kubernetes cluster we need some cli tools:

1. kubectl 
2. 