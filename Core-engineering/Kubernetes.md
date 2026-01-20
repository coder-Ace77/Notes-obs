
---

Kubernetes is open sourcse tool for customizing deployment, scaling and management of containerized application. Follows master slave architecture.

1. Master: Control plane -> 
	1. kube scheduler
	2. kube-api-server
	3. kube control manager
	4. etcd
2. Worker node ->
	1. Kube proxy
	2. kubelet
	3. container pods

### Working of kubernetes at high level:

At the heart of every Kubernetes cluster lies the **control plane**, a set of components that make global decisions about the cluster, like scheduling, monitoring, and managing the cluster's state. The control plane ensures that the desired state of the cluster is always maintained and that any changes or failures are handled automatically.

![[Pasted image 20250828093202.png]]

1. Kube api-server: The **Kube-API Server** is the gateway to all interactions with the Kubernetes cluster. It exposes the Kubernetes API, which is the primary interface through which both internal components and external users communicate with the cluster. Whether it's creating, updating, or deleting resources like pods, services, or deployments, all requests go through the kube-api server.
   As Kubernetes is a RESTful system, every request to the API server is either a GET, POST, PUT, or DELETE operation. The API server validates these requests, processes them, and then stores the updated state in **etcd**, a distributed key-value store. The API server also serves as the central hub for monitoring the health and status of the cluster, ensuring that all components are in sync and running as expected.

2. Kube scheduler: The **Kube-Scheduler** is tasked with making crucial decisions about where the containers (in the form of pods) will run within the cluster. When a pod is created, the scheduler assesses the cluster's available resources and determines the best worker node to run the pod. It considers factors like CPU and memory availability, affinity rules (which dictate which pods should be co-located), taints, tolerations, and node-specific labels to decide where a pod should run.

	This decision-making process ensures that workloads are distributed optimally across the cluster, helping prevent resource bottlenecks and ensuring high availability.
	
3. The **Kube-Control Manager** is a collection of controllers that ensure the cluster's desired state is constantly maintained. These controllers are responsible for specific tasks, such as:

	- **Replicas Controller**: Ensures that the desired number of pod replicas are running.
	- **Deployment Controller**: Manages the deployment of new versions of applications while maintaining availability.
	-  **Node Controller**: Monitors the health of nodes and performs actions like replacing failed nodes.
	- **Job Controller**: Ensures that jobs (which run tasks to completion) are executed successfully.

	If any resource in the cluster falls out of its desired state (for example, if a pod crashes or a node goes down), the kube-control manager will take corrective actions, like spinning up a new pod or rescheduling workloads to healthy nodes.

4. Etcd: The **etcd** component is a distributed key-value store that holds the entire configuration and state of the Kubernetes cluster. It’s where the control plane stores information about the state of the cluster, such as pod specifications, node status, and the desired configuration of services. Etcd acts as the “source of truth,” meaning that it contains the definitive record of the system’s state, and other components (like the API server and kube-scheduler) rely on it to make decisions.

	The highly available and fault-tolerant nature of etcd ensures that the cluster's state is persistent, even in the event of failure or restarts.


### The Worker Nodes

While the control plane is responsible for orchestration and management, **worker nodes** are the backbone of the Kubernetes system. Worker nodes are where your applications and workloads run in containers. These nodes execute the tasks assigned by the control plane, ensuring that containers are running and communicating with each other as needed.

#### 1. **Kubelet**

The **Kubelet** is an agent that runs on every worker node and is responsible for ensuring that the containers in its respective node are running and healthy. The kubelet interacts with the node’s container runtime (such as Docker or containerd) to launch, manage, and stop containers as directed by the control plane. It continuously monitors the health of the pods and ensures they are running in the correct state, reporting back to the API server about the status of the containers.

If the kubelet detects that a container has failed or isn’t running as expected, it will take action to restart it or report the failure to the control plane, prompting it to take corrective measures.

#### 2. **Kube-Proxy**

The **Kube-Proxy** is responsible for maintaining network rules on each worker node, allowing communication between services and pods within the cluster. Kube-Proxy routes traffic to the correct pod, based on service definitions, ensuring that requests to a service are load-balanced across multiple pods if needed.

Kube-Proxy uses either **iptables** or **IPVS** (IP Virtual Server) to configure the network rules that enable communication between pods, services, and external traffic. In essence, it enables seamless network connectivity in a microservices architecture by abstracting away the complexity of networking between containers.

#### 3. **Container Pods**

At the core of Kubernetes are **pods**—the smallest and most basic units of deployment. A **pod** is a logical host for one or more containers. Containers within a pod share the same network namespace, meaning they can communicate with each other using localhost, and they can share storage volumes to exchange data. Pods are designed to be ephemeral, and their lifecycle is managed by the Kubernetes scheduler.

When you create an application in Kubernetes, you typically define it as a pod specification, which includes the number of replicas, resource limits, and other configurations. The control plane schedules these pods onto the worker nodes, where they are executed. Pods provide a layer of abstraction, allowing the containers within them to be deployed and managed more easily.


---

Why kubernetes works so well is because not only worker nodes are distributed but the orchestration tool which is kubernetes is also distributed in nature. In fact all the components of control plane itself are distributed that is why we need etcd which acts as single source of truth.

The control plane consists of several components—such as the **Kube-API Server**, **Kube-Scheduler**, **Kube-Controller Manager**, and **etcd**—that can be distributed across multiple nodes to avoid a single point of failure and ensure high availability

### In depth about availability:

The **Kube-API Server** is the central communication hub in the control plane, receiving and processing all API requests. In a highly available setup, multiple instances of the API server can run on different nodes. These instances are configured to be **stateless**.



---

## In depth:

### etcd: 

It is a consistent and highly available key-value store used as kubernetes backing store for all cluster data. Any information about cluster must be stored here for example load etc.

The name “etcd” comes from a naming convention within the Linux directory structure: In UNIX, all system configuration files for a single system are contained in a folder called “/etc;” “d” stands for “distributed.

When client sends any request to create/update object API server authenticates and validates req and is written into etcd. 

What info it stores:

1. Mainly metadata about nodes, pods and services. Internally infromation is 

