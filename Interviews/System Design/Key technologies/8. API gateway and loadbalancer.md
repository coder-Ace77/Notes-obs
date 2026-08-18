
---

Especially in a microservice architecture, an API gateway sits in front of your system and is responsible for routing incoming requests to the appropriate backend service.The gateway is also typically responsible for handling cross-cutting concerns like authentication, rate limiting, and logging.

![[Pasted image 20251013114410.png]]

Most common api gateways are AWS API gateway or nginx.

Most system design problems will require you to design a system that can handle a large amount of traffic. When you have a large amount of traffic, you will need to distribute that traffic across multiple machines (called horizontal scaling) to avoid overloading any single machine or creating a hotspot. This is where a load balancer comes in. For the purposes of an interview, you can assume that your load balancer is a black box that will distribute work across your system.

The reality is that you need a load balancer wherever you have multiple machines capable of handling the same request.

![[Pasted image 20251013114953.png]]

Note that sometimes you'll need to have specific features from your load balancer, like sticky sessions or persistent connections. The most common decision to make is whether to use an L4 (layer 4) or L7 (layer 7) load balancer.

You can somewhat shortcut this decision with a simple rule of thumb: if you have persistent connections like websockets, you'll likely want to use an L4 load balancer. Otherwise, an L7 load balancer offers great flexibility in routing traffic to different services while minimizing the connection load downstream.

The most common load balancers are AWS Elastic Load Balancer (a hosted offering from AWS), NGINX (an open-source webserver frequently used as a load balancer), and HAProxy(a popular open-source load balancer)

- The **load balancer** decides **which gateway or backend instance** gets the request.
- The **API gateway** decides **which microservice endpoint** should handle it, and **how** (auth, rate limits, etc.).

Load balancers maintain a list of instances and uses algorithm like round robin or least connections. However it needs to constantly do a health check

### Request flow:

Suppose we are having microservice architectiure and a reuqest comes first it will be resolved by dns and will reach the api gateway gateway will perform stuff like auth etc and will route it to the **load balancer** in front of Microservice. Load balancer will decide which instance to give the request to.

In kubernetes Each microservice has a **Kubernetes Service** object that acts as an internal **load balancer + service discovery**. The API Gateway (Ingress Controller) routes external requests into the cluster.

Now api gateway can perform many tasks - 

### Request validation

This validation includes checking that the request URL is valid, required headers are present, and the request body (if any) matches the expected format.

This early validation is important because it helps catch obvious issues before they reach your backend services. For example, if a mobile app sends a malformed JSON payload or forgets to include a required API key, there's no point in routing that request further into your system. 

### Middleware task

API gateway can handle many middleware tasks like

- Authenticate requests using JWT tokens
- Limit request rates to prevent abuse
- Terminate SSL connections
- Log and monitor traffic
- Compress responses
- Handle CORS headers
- Whitelist/blacklist IPs
- Validate request sizes
- Handle response timeouts
- Version APIs
- Throttle traffic
- Integrate with service discovery

### Routing 

The gateway maintains a routing table that maps incoming requests to backend services. This mapping is typically based on a combination of:

- URL paths (e.g., /users/* routes to the user service)
- HTTP methods (e.g., GET, POST, etc.)
- Query parameters
- Request headers

### Backend communication

While most services communicate via HTTP, in some cases your backend services might use a different protocol like gRPC for internal communication. When this happens, the API Gateway can handle translating between protocols, though this is relatively uncommon in practice.

The gateway would, thus, transform the request into the appropriate protocol before sending it to the backend service. This is nice because it allows your services to use whatever protocol or format is most efficient without clients needing to know about it.

### Caching 

Before sending the response back to the client, the gateway can optionally cache the response. This is useful for frequently accessed data that doesn't change often and, importantly, is not user specific. If your expectation is that a given API request will return the same result for a given input, caching it makes sense.

### Scaling api gateway

When discussing API Gateway scaling in interviews, there are two main dimensions to consider: handling increased load and managing global distribution.

### Horizontal scaling

The most straightforward approach to handling increased load is horizontal scaling. API Gateways are typically stateless, making them ideal candidates for horizontal scaling. You can add more gateway instances behind a load balancer to distribute incoming requests.

Another option that works well particularly for large applications with users spread across the globe is to deploy API Gateways closer to your users, similar to how you would deploy a CDN. This typically involves:

1. **Regional Deployments**: Deploy gateway instances in multiple geographic regions
2. **DNS-based Routing**: Use GeoDNS to route users to the nearest gateway
3. **Configuration Synchronization**: Ensure routing rules and policies are consistent across regions


With a microservices architecture, an API Gateway becomes almost essential. Without one, clients would need to know about and communicate with multiple services directly, leading to tighter coupling and more complex client code. The gateway provides a clean separation between your internal service architecture and your external API surface.

The most straightforward approach to handling increased load is horizontal scaling. API Gateways are typically stateless, making them ideal candidates for horizontal scaling. You can add more gateway instances behind a load balancer to distribute incoming requests.

While API Gateways are primarily known for routing and middleware functionality, they often include load balancing capabilities. However, it's important to understand the distinction:

- **Client-to-Gateway Load Balancing**: This is typically handled by a dedicated load balancer in front of your API Gateway instances (like AWS ELB or NGINX).
- **Gateway-to-Service Load Balancing**: The API Gateway itself can perform load balancing across multiple instances of backend services.

This can typically be abstracted away during an interview. Drawing a single box to handle "API Gateway and Load Balancer" is usually sufficient. You don't want to get bogged down in the details of your entry points as they're more likely to be a distraction from the core functionality of your system.

There are mainly two types of load balancing associated with api gateways.
 
 1) Client-to-Gateway Load Balancing (typically handled by a dedicated load balancer in front of gateway instances),
 2) Gateway-to-Service Load Balancing (the gateway itself balancing across backend service instances).