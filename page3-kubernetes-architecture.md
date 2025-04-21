# Page3: Kubernetes Architecture

<figure><img src=".gitbook/assets/image (1).png" alt=""><figcaption><p><a href="https://jvns.ca/blog/2017/06/04/learning-about-kubernetes/">https://jvns.ca/blog/2017/06/04/learning-about-kubernetes/</a></p></figcaption></figure>

<figure><img src=".gitbook/assets/image (2).png" alt=""><figcaption><p>K8s Architecture Control Plane</p></figcaption></figure>

### **Master Node & Control Plane**

<figure><img src=".gitbook/assets/image (5).png" alt=""><figcaption></figcaption></figure>

#### Key Points:

* **Master is responsible for managing the complete cluster.**
* **Accessing the master node:** Can be done via **CLI, GUI, or API**.
* **Responsibilities of the master:**
  * Monitors the nodes in the cluster.
  * Orchestrates containers on the worker nodes.
* **High Availability:** Multiple master nodes can be configured to achieve **fault tolerance**.
* **Access Point:** Master node acts as the access point for admins and users to **schedule** and **deploy** containers.
* **Components of the Control Plane:**
  * **ETCD**
  * **Scheduler**
  * **Controller**
  * **API Server**

These four components together are known as the **Control Plane**.



#### **API Server**

* Exposes **Kubernetes APIs** and acts as the **frontend for the control plane**.
* Listens to updates or queries via CLI tools like **kubectl**.
* **Kubectl communicates** with the API Server to inform what actions need to be performed, like **creating or deleting pods**.
* **Validates requests** and then forwards them to the appropriate components.
* **All requests must go through the API Server**; direct access to the cluster is not allowed.

***

#### **Kube-Controller Manager**

* **Acts as the brain** behind orchestration.
* Responsible for **monitoring and reacting** when nodes, containers, or endpoints go down.
* Makes decisions to **bring up new containers** when needed.
* Runs **control loops** to maintain the cluster’s desired state (deployments, replicas, node status, etc.).

**Examples of controllers:**

* **Node controller**
* **Replication controller**
* **Endpoint controller**
* **Job controller**
* **Service Account and Token controllers**, etc.

## The Controller Manager's Role in Maintaining Kubernetes Cluster State

The Controller Manager plays a crucial role in Kubernetes' self-healing architecture. Let me walk through what happens when it detects a deviation between the actual and desired state of the cluster.

### The Reconciliation Loop Process

When the Controller Manager detects a deviation from the desired state, it initiates what's known as a reconciliation loop. This is the fundamental pattern that drives Kubernetes' declarative model of operation.

#### Step 1: Detection of State Deviation

The Controller Manager doesn't directly observe the cluster state. Instead, each controller within the Controller Manager watches the API Server for changes to specific resources they're responsible for. This detection happens through:

* **Watch Mechanisms**: Controllers establish watch connections to the API Server, receiving near real-time notifications when resources change.
* **Periodic Resync**: Even without changes, controllers periodically fetch all resources they care about to ensure nothing is missed.

For example, the ReplicaSet controller continuously monitors the current pod count against the desired count specified in the ReplicaSet definition.

#### Step 2: Information Flow

When a deviation is detected, the Controller Manager does not inform any other component directly. Instead:

1. The specific controller (like Deployment Controller or ReplicaSet Controller) reads the current state from the API Server.
2. It compares this with the desired state defined in the resource specification.
3. It calculates what changes are needed to converge toward the desired state.

#### Step 3: Taking Corrective Actions

The Controller Manager itself doesn't make direct changes to the cluster infrastructure. Instead:

1. It creates, updates, or deletes resource objects through the API Server.
2. The API Server validates these changes, persists them to etcd, and notifies relevant components.
3. Other components (like the Scheduler or kubelet) respond to these changes.

For example, if a ReplicaSet should have 5 pods but only 3 are running:

1. The ReplicaSet controller creates 2 new Pod objects via the API Server.
2. The API Server stores these in etcd.
3. The Scheduler notices the unscheduled pods and assigns them to nodes.
4. The kubelet on the selected nodes creates the actual containers.

#### Step 4: Handling Failures

If the Controller Manager encounters issues during reconciliation:

1. It typically uses exponential backoff to retry failed operations.
2. It logs events that administrators can view (using `kubectl describe` or in the events stream).
3. It may mark resources with specific status conditions to indicate problems.

### Concrete Example: Node Failure Scenario

Let's trace through what happens when a node fails:

1. **Detection Phase**:
   * The Node Controller (part of Controller Manager) notices a node hasn't sent heartbeats.
   * After a grace period (default 40 seconds), it marks the node as "NotReady".
   * After another period (default 5 minutes), it begins to evict pods from the failed node.
2. **Communication Phase**:
   * The Node Controller updates the Node object's status in the API Server.
   * It adds "eviction" taints to the Node object.
   * It terminates pods on the unreachable node by updating their status.
3. **Remediation Phase**:
   * Other controllers (like the ReplicaSet Controller) notice their pods are terminated.
   * These controllers create replacement pods via the API Server.
   * The Scheduler assigns these new pods to healthy nodes.
   * Kubelets on those nodes create the replacement containers.

### Different Controllers and Their Specific Actions

The Controller Manager consists of many individual controllers, each handling specific resources:

* **ReplicaSet Controller**: Ensures the correct number of pod replicas exist
* **Deployment Controller**: Manages rollouts and rollbacks of application versions
* **StatefulSet Controller**: Maintains stateful applications with stable network identities
* **Node Controller**: Monitors node health and responds to node failures
* **Endpoint Controller**: Creates Endpoint objects that connect Services to Pods
* **Namespace Controller**: Cleans up resources when namespaces are deleted
* **Job Controller**: Handles one-time or batch execution tasks

Each controller follows the same fundamental pattern but takes different specific actions to reconcile their resources.

### The Greater Control Loop Architecture

The entire Kubernetes control plane operates as a series of asynchronous, eventually consistent control loops:

1. **API Server**: The central coordination point receiving all state changes
2. **Controller Manager**: Watching and reconciling resource states
3. **Scheduler**: Assigning pods to nodes
4. **kubelet**: Running containers on nodes

This decentralized architecture allows Kubernetes to scale and maintain resilience even when parts of the system fail temporarily.

The remarkable aspect of this design is that there is no central orchestrator. Each component reacts to changes in its domain, and together, they move the entire system toward the desired state.



#### **ETCD**

* ETCD is a **distributed, reliable key-value store** used by Kubernetes to **store all data** used to manage the cluster.
* When you have **multiple nodes and multiple masters** in your cluster, etcd stores all that information across all nodes in a **distributed manner**.
* Information in etcd is typically formatted in **human-readable YAML**.

Every component in Kubernetes (the API server, the scheduler, the kubelet, the controller manager, whatever) is stateless.

## Understanding "Stateless" Components in Kubernetes Architecture

When we say Kubernetes components are "stateless," we're highlighting a fundamental architectural principle that shapes how the entire system works. Let me explain what this means and why it matters.

### What "Stateless" Means in Kubernetes

In the context of Kubernetes, "stateless" means that components like the API server, scheduler, controller manager, and kubelet don't persist their operational data between restarts. They don't "remember" what they were doing if they crash and restart.

Think of these components as workers who start each day with a clean slate. If a worker gets sick and needs a replacement, the new worker can step in immediately because all the information about what needs to be done is stored elsewhere (in etcd).

### Why etcd Is Different

Etcd is deliberately designed as the central "memory" of the cluster. It's the only component that is **stateful** by design. It's a distributed, reliable key-value store that:

1. Persistently stores all cluster configuration
2. Records the desired state of all objects in the cluster
3. Maintains the current observed state of those objects
4. Provides a consistent way for components to watch for changes

### How This Architecture Works in Practice

Let me walk through an example that illustrates this stateless design:

Imagine you have a Deployment that should maintain 3 replicas of an application. Here's what happens:

1. When you create this Deployment, the API server validates it and stores it in etcd.
2. The API server itself doesn't "remember" this Deployment in its memory.
3. The controller manager has a deployment controller that watches etcd for Deployments.
4. This controller sees the new Deployment in etcd and creates a ReplicaSet object in etcd.
5. The ReplicaSet controller (also part of the controller manager) watches etcd for ReplicaSets.
6. It sees the new ReplicaSet and creates 3 Pod objects in etcd.
7. The scheduler watches etcd for unscheduled Pods.
8. It assigns each Pod to a node and updates the Pod objects in etcd.
9. Each node's kubelet watches etcd for Pods assigned to it.
10. When a kubelet sees a Pod assigned to its node, it creates the containers.

At each step, components read from etcd to determine what they should do, take action, and write results back to etcd. They don't maintain their durable record of what exists or what actions they've taken.

**Basically, everything in Kubernetes works by watching etcd for stuff it has to do, doing it, and then writing the new state back into etcd.**

***

#### **Scheduler**

* The **scheduler is responsible** for **distributing work or containers** across multiple nodes.
* It looks for **newly created containers** and schedules them for the available nodes.
* **Scheduling decisions** are based on factors like:
  * **Pod resource requirements**
  * **Hardware/software/policy constraints**
  * **Affinity and anti-affinity rules**
  * **Taints and tolerations**, etc.

The Kubernetes scheduler is in charge of scheduling pods onto nodes. It works like this:

1. You create a pod
2. The scheduler notices that the new pod you created doesn’t have a node assigned to it
3. The scheduler assigns a node to the pod

#### How[ the scheduler works, in English](https://jvns.ca/blog/2017/07/27/how-does-the-kubernetes-scheduler-work/#how-the-scheduler-works-in-english) <a href="#how-the-scheduler-works-in-english" id="how-the-scheduler-works-in-english"></a>

1. At the beginning, every pod that needs scheduling gets added to a queue
2. When new pods are created, they also get added to the queue
3. The scheduler continuously takes pods off that queue and schedules them
4. That’s it

One interesting thing here is that, if for whatever reason a pod **fails** to get scheduled, there’s nothing in here yet that would make the scheduler retry. It’d get taken off the queue, it fails scheduling, and that’s it. It lost its only chance! (unless you restart the scheduler, in which case everything will get added to the pod queue again)

Of course, the scheduler is smarter than that – when a pod fails to schedule, in general, it calls an error handler, like this:

```go
host, err := sched.config.Algorithm.Schedule(pod, sched.config.NodeLister)
if err != nil {
	glog.V(1).Infof("Failed to schedule pod: %v/%v", pod.Namespace, pod.Name)
	sched.config.Error(pod, err)
```

This `sched.config.Error` function call adds the pod back to the queue of things that need to be scheduled, and so it tries again.



**Worker Nodes:**

#### **Kubelet**

* Worker nodes have the **kubelet** agent, which is responsible for interacting with the master.
* Carry out actions requested by the master on the worker nodes.
* The kubelet takes a set of **PodSpecs** that are provided through various mechanisms and ensures that the containers described in those PodSpecs are running and healthy.
* Reporting the health status of the node and each pod/container.
* The kubelet doesn't manage containers that were not created by Kubernetes.\
  &#xNAN;_&#x45;x: Containers created using Docker commands_



### Kubernetes Services: The Networking Abstraction

**Services** abstract the complexity of dynamic pods and networking.

Use **Services** for all pod-to-pod or external communications.

**CoreDNS** resolves service names to service IPs.

A Service in Kubernetes is a fundamental abstraction that solves a critical problem in container orchestration: **how to reliably connect to pods that are ephemeral and dynamically scheduled.** Let me walk you through what Services are and how they work.

#### The Core Purpose of Services

Pods in Kubernetes are designed to be temporary. As pods are **ephemeral**, they can be created, destroyed, or rescheduled at any time. **They're assigned IP addresses dynamically, which means a pod's IP address isn't reliable for long-term connectivity.** This presents a challenge: how do other components reliably communicate with these constantly changing pods?

Services solve this problem by providing a **stable endpoint for a set of pods that perform the same function.** Think of a Service as a **persistent front door/load balancer** with a fixed address that directs traffic to whichever pods are currently running your application.

#### Key Characteristics of Services

A Service in Kubernetes:

1. **Provides a stable virtual IP (ClusterIP)** that never changes throughout the Service's lifetime
2. **Maintains a consistent DNS name** within the cluster
3. **Load balances traffic** across all pods that match its selector
4. **Automatically updates its endpoints** as pods come and go

```
apiVersion: v1
kind: Service
metadata:
  name: my-application
spec:
  selector:
    app: my-app
  ports:
    - port: 80
      targetPort: 8080
  type: ClusterIP
  
This Service:

Creates a stable endpoint named "my-application"
Selects all pods with the label app: my-app
Forward traffic from port 80 on its virtual IP to port 8080 on the pods
It is only accessible within the cluster (ClusterIP type)
```

#### **Kube Proxy**

* **kube-proxy** is a network proxy that runs on each node in your cluster, implementing part of the Kubernetes Service concept. It's responsible for implementing the Service concept at the network level. Without kube-proxy, Services would just be abstract concepts with no actual networking implementation.
* **Kube-proxy** maintains forwarding rules on each node.
* It routes traffic coming into a node from the service and forwards requests to the correct pods.
*   #### How Kube-Proxy Works

    kube-proxy continuously watches the **Kubernetes API server** for changes to **Service and Endpoint objects**. When it detects changes, it updates the node's network rules to properly route traffic destined for Services.
* A Kubernetes service manages a collection of pods, and the service gets an IP address.\
  **Kube-dns** resolves Kubernetes service DNS names to their IP addresses.\
  kube-proxy sets up iptables rules in the host in such a way that the traffic coming to services gets forwarded to the pods in a random load-balancing fashion.
*   Kube-Proxy Modes: Userspace Mode: Older version, not used much.

    * **iptables mode** (default):
      * Uses Linux kernel's iptables to implement network routing rules
      * More efficient than userspace mode, as packets are processed in kernel space
      * Creates a randomized list of backend pods for load balancing
    *   **IPVS mode** (enhanced):

        * Uses Linux kernel's IP Virtual Server for more efficient load balancing
        * Supports more load balancing algorithms than iptables
        * Scales better for large numbers of Services



    #### The Flow of Traffic Through kube-proxy

    When a client pod attempts to connect to a Service, here's what happens:

    1. The client requests the Service's ClusterIP and port
    2. The node's network stack (configured by kube-proxy) intercepts this traffic
    3. kube-proxy's rules redirect the traffic to one of the backend pods
    4. The return traffic follows the established connection back to the client

    This all happens transparently to both the client and the server pods. The client only needs to know the Service's IP and port, not the details of which pods are implementing the Service or where they're running.

#### A Concrete Example of kube-proxy in Action

Imagine a Service named "web" that selects three pods running a web application. Here's what kube-proxy does behind the scenes:

1. kube-proxy notices the "web" Service has been created with ClusterIP 10.96.0.10
2. It also sees that this Service has three endpoint pods with IPs 10.244.1.2, 10.244.2.3, and 10.244.3.4
3. In iptables mode, kube-proxy creates iptables rules on every node that:
   * Intercept traffic destined for 10.96.0.10:80
   * Use probability-based rules to redirect to one of the three endpoint IPs
   * Rewrite the destination IP from the Service IP to the selected pod IP

When a pod requests 10.96.0.10:80, the node's kernel networking stack, configured by these iptables rules, transparently redirects the packet to one of the endpoint pods.

#### kube-proxy Visualized in the Cluster Architecture

kube-proxy sits alongside the kubelet on every node in the cluster:

1. The API server stores Service and Endpoint information in etcd
2. kube-proxy watches the API server for changes to this information
3. kube-proxy updates the node's networking rules based on these changes
4. Node networking rules (iptables/IPVS) handle the actual packet routing

Together, Services and kube-proxy create a powerful abstraction that hides all the complexity of routing traffic in a dynamic environment:

1. Services define the logical grouping and stable endpoint
2. Endpoints track which pods are currently members of each Service
3. kube-proxy ensures traffic reaches the right pods by programming the node's network stack
4. DNS provides simple service discovery for pods

This mechanism is fundamental to Kubernetes networking, allowing applications to communicate reliably despite the dynamic nature of container orchestration.

#### **P Address Assignment**

* IP addresses are assigned as follows:
  * **Pods:** Assigned from a **CIDR block** via a **CNI plugin** (Container Network Interface).
  * **Services:** Assigned by the **API Server**.
  * **Nodes:** Assigned by the **Cloud Controller Manager**.



**CNI:**&#x20;

* Kubernetes clusters require to allocate non-overlapping IP addresses for Pods, Services and Nodes, from a range of available addresses.
* The kube-apiserver is configured to assign IP addresses to Services.
* The kubelet is configured to assign IP addresses to Nodes.
* The network plugin (**CNI**) is configured to assign IP addresses to Pods when they are scheduled.
* CNI Plugin is focusing on building up an overlay network, without which Pods can't communicate with each other.



<figure><img src=".gitbook/assets/Screenshot from 2025-04-20 12-50-03.png" alt=""><figcaption></figcaption></figure>

#### **Container Runtime Environment**

* Referring to the software layer that enables containers to run on an operating system
* Containers are [not first-class objects ](#user-content-fn-1)[^1]in the Linux kernel
* Containers are fundamentally composed of several underlying kernel primitives: **namespaces** (who you are allowed to talk to), **cgroups** (the number of resources you are allowed to use), and LSMs (Linux Security Modules—what you are allowed to do). Together, these kernel primitives allow us to set up secure, isolated, and metered execution environments for our processes
* Creating these environments manually each time we want to create a new isolated process would be tiresome and error-prone
* To avoid this, all the components have been bundled together in a concept called a **container**
* The **container runtime** is the software that is responsible for running these containers
* The runtime executes the container, telling the kernel to assign resource limits, create isolation layers (for processes, networking, and filesystems), and so on, using a cocktail of mechanisms like control groups (cgroups), namespaces, capabilities, SELinux, etc

### Kubectl

#### 🔹 Text Summary:

* **Kubectl** is the command-line utility used to **interact with a Kubernetes cluster**, like a client.
* Kubernetes is **fully controlled through its REST API**.
* Every Kubernetes operation is exposed as an **API endpoint** and can be executed via **HTTP requests**.
* `kubectl` interacts with the cluster by consuming these API endpoints.

```shell
kubectl run nginx                  # Deploy a pod to the cluster
kubectl cluster-info              # View information about the cluster
kubectl get nodes                 # List all the nodes that are part of the cluster
kubectl get componentstatuses     # Get health status of control plane components


```

&#x20;                                                  ![](<.gitbook/assets/image (4).png>)



**Kubernetes Objects:**

```
The following command successfully displays all Kubernetes objects

kubectl api-resources
```



## References:

K8s

[https://jvns.ca/blog/2017/06/04/learning-about-kubernetes/](https://jvns.ca/blog/2017/06/04/learning-about-kubernetes/)

Controller:

[https://github.com/kubernetes/community/blob/8decfe4/contributors/devel/controllers.md](https://github.com/kubernetes/community/blob/8decfe4/contributors/devel/controllers.md)

API Server:

[https://kamalmarhubi.com/blog/2015/09/06/kubernetes-from-the-ground-up-the-api-server/](https://kamalmarhubi.com/blog/2015/09/06/kubernetes-from-the-ground-up-the-api-server/)

Scheduler:

[https://jvns.ca/blog/2017/07/27/how-does-the-kubernetes-scheduler-work/](https://jvns.ca/blog/2017/07/27/how-does-the-kubernetes-scheduler-work/)

[https://kamalmarhubi.com/blog/2015/11/17/kubernetes-from-the-ground-up-the-scheduler/](https://kamalmarhubi.com/blog/2015/11/17/kubernetes-from-the-ground-up-the-scheduler/)

Kubelet:

[https://kamalmarhubi.com/blog/2015/08/27/what-even-is-a-kubelet/](https://kamalmarhubi.com/blog/2015/08/27/what-even-is-a-kubelet/)

Services:

[https://iximiuz.com/en/posts/service-discovery-in-kubernetes/](https://iximiuz.com/en/posts/service-discovery-in-kubernetes/)

[https://kubernetes.io/docs/concepts/services-networking/service/](https://kubernetes.io/docs/concepts/services-networking/service/)

[https://www.baeldung.com/ops/kubernetes-service-types](https://www.baeldung.com/ops/kubernetes-service-types)

K8 Objects:

[https://kubernetes.io/docs/concepts/overview/working-with-objects/](https://kubernetes.io/docs/concepts/overview/working-with-objects/)

[https://kodekloud.com/blog/kubernetes-objects/](https://kodekloud.com/blog/kubernetes-objects/)

Claude:

[https://claude.ai/share/e25517b9-b749-4bfc-a37b-ea4b66022072](https://claude.ai/share/e25517b9-b749-4bfc-a37b-ea4b66022072)

[^1]: It means that:

    * **Containers are not a native feature** of the Linux kernel in the way that, say, processes, threads, or filesystems are.
    * The **Linux kernel does not have a single object or API** called "container" that you can directly interact with.

    Instead, **containers are built by combining existing kernel features**, such as:

    * **Namespaces** – for isolation (process, network, user, etc.)
    * **cgroups** – for resource control (CPU, memory, etc.)
    * **Capabilities, SELinux, AppArmor, seccomp**, etc. – for security

    So, the **Linux kernel provides the building blocks**, and **container runtimes (like Docker or containerd)** assemble those into what we know as containers.
