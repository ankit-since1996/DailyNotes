---
description: Container Management System
---

# Amazon EKS - Knowledge Badge

[Reference URL](https://skillbuilder.aws/learning-plan/FRN6Z7KMRY/amazon-eks--knowledge-badge-readiness-path)

* Container Basics
* Why AWS EKS for Kubernetes Workloads
* Introduction to Kubernetes Core Concepts
* Introduction to AWS EKS Cluster
* EKS Security
* EKS Networking
* Observabilty
* Autoscaling and Cost Optimization
* GitOps





### Why AWS EKS for Kubernetes Workloads

Why run your Kubernetes workloads on Amazon Elastic Kubernetes Service (EKS)?

Access to the vast ecosystem, Karpenter for AutoScaling

***

### Introduction to Kubernetes Core Concepts

The lesson covers fundamental concepts including **clusters**, **control plane nodes**, **worker nodes,** and **pods**, demonstrating how these components work together to provide scalability and fault tolerance.

Through a practical three-tier web application example, we will explore how traditional application hosting evolves into a containerized Kubernetes environment.

* Front-End Application Layer
* Backend Application Logic Layer
* DataBase  Layer

Then come containers, so that we can bundle our application code and its dependencies together.

**Kubernetes Objects: Persistent** entities in Kubernetes that represent the state of your cluster\
These objects tell Kubernetes what we want in our cluster, like how our cluster should look: \
what applications you want to run\
what kind of resources they will require, &\
how they should behave.

\
We define these objects in **YAML files** and submit them to the Kubernetes API Server, and then K8s decides how to provision/configure and manage them.

\
**eg:** \
Workload Objects: These objects manage your containers and ensure they stay running: Pod, Deployment, StatefulSet, DaemonSets, Jobs & CronJobs.  \
Networking Objects (Connecting & Exposing Apps): Because Pods are temporary and their IP addresses change whenever they restart, Kubernetes uses networking objects to provide stable endpoints: Service and Ingress\
Configuration & Storage Objects (Data & Settings): These keep your configuration and data separate from your container images. Secret, Configmap, PV/PVC\
\
etc...

***

**Cluster:** a group of machines (called nodes) that work together to run containerized applications. Two main components:\
**Control Plane Nodes**: Kubernetes Orchestration (management) Unit.\
Control plane nodes are responsible for orchestration-related tasks such as scaling, updating, and launching applications, essentially serving as the "**brain**" of the cluster.\
Running Components in Control Plane Nodes:\
• ETCD: the cluster's database storing all cluster state in key-value pairs. Stateful.\
• kube-API Server: handles all API operations. Exposes Kubernetes APIs.\
• kube-Controller Manager: runs controller processes. eg: node controller, deployment controller, etc.\
• kube-Scheduler: responsible for scheduling pods on nodes

\
&#x20;

<div align="left"><figure><img src=".gitbook/assets/image (30).png" alt="" width="563"><figcaption></figcaption></figure></div>

\
**Worker Nodes**: Kubernetes workload unit. The application lives here.\
Nodes/Machines where the actual application workloads run, hosting the pods that contain your containerized applications.

<div align="left"><figure><img src=".gitbook/assets/image (26).png" alt="" width="563"><figcaption></figcaption></figure></div>

<div align="left"><figure><img src=".gitbook/assets/image (28).png" alt="" width="375"><figcaption><p>Fault Tolerant + Hogh Availabilty</p></figcaption></figure></div>

POD:  A **Pod** is the smallest deployable unit in Kubernetes that wraps one or more containers.\
While it's technically possible to have multiple containers in a pod, the best practice/recommended approach is to keep one container per pod for non-tightly coupled microservices. This design principle aligns with the single responsibility concept and makes applications easier to manage, scale, and troubleshoot.

A **Pod** is a group of Linux processes (containers) that share the same isolated Linux namespaces—specifically the **network namespace**, **IPC namespace**, and **storage volumes**—so they behave as if they are running inside the same virtual machine.

**IMPORTANT**: Each Pod is allocated its own single IP address. \
If an application needs its own unique IP address, then it needs its own POD.

* **Shared within Pods** (across containers): Containers on the same Pod can communicate with each other as if they are on the same machine but still isolated from each other.
  * **Network:** All containers share one IP address and port space, meaning they talk to each other via **`localhost`** (and cannot bind to the same port number). i.e in **multi-container pods**, they behave as if they are in a single network and share the same IP address.
  * **Storage:** Mounted volumes (directories) can be read and written by all containers in the Pod.
  * **IPC (Inter-Process Communication):** Processes can communicate via System V IPC or POSIX message queues.
* **Isolated per Container:**
  * **Filesystem (Mount Namespace):** Each container still has its own independent root filesystem from its container image (e.g., an Ubuntu base vs. an Alpine base).
  * **Process Tree (PID Namespace):** By default, each container only sees its own processes (though PID sharing can be enabled).
  * **Resource Limits (cgroups):** CPU and memory limits are enforced individually per container process.

So, in case containers are **tightly packed** and need to communicate with each other very much, with shared storage requirements, we can have multiple containers in a single pod.

**NOTE**: Because PODS are the smallest deployable unit in K8s and are scaled up and down as a unit, any number of containers it holds will also scale accordingly.\
\
By placing each microservice in its own pod, you gain several advantages: \
• independent scaling (each service can scale based on its own demands), \
• simpler resource management (clearer resource allocation per service), \
• easier updates and rollbacks (changes to one service don't affect others),\
• better fault isolation (if one service fails, it doesn't impact the other), and \
• Easier monitoring and debugging as metrics and logs are cleanly separated per service.



**IMPORTANT**: Pods are **immutable**: Once a Pod has been created, it cannot be changed.\
If anything related to a Pod needs to be changed/updated after its launch, it needs to be replaced by new Pods.

&#x20;

<div align="left" data-with-frame="true"><figure><img src=".gitbook/assets/image (25).png" alt="" width="300"><figcaption><p>Node -> POD -> Container -> Image</p></figcaption></figure></div>

Node: **A physical** or virtual machine that provides the computing hardware to run those Pods.\
**One-to-Many Relationship:** A single **Node** can host multiple **Pods** simultaneously, depending on its CPU and RAM capacity.

For **fault tolerance** and **high availability**, at least start with min 2 Nodes or more, depending on the workload. In case a Node fails, any Pods running on that node will also go down.  Node and Pod scheduling is automatically handled by Kubernetes itself due to its self-healing nature. Cluster Control Plane is highly responsible for managing all this.

\
**INFO**: We can configure it to run certain Pods on certain Nodes using **Taints** and **Tolerations**.



#### Namespaces and Labels: Cluster Object Organization

A namespace is essentially a "**virtual cluster**" inside your physical cluster that **groups** and **isolates** resources.&#x20;

* **Use a Namespace when you need boundaries:**
  * Separating environments (`dev`, `staging`, `prod`) or different teams (`team-payments`, `team-search`).
  * Applying security permissions (e.g., Team A can only deploy inside `namespace: team-a`).

\
Resources in a namespace need to be **uniquely** named. Not across the whole cluster.

By default, 4 K8s namespaces are created:\
**kube-system:** objects created directly by Kubernetes live in this namespace; kube-proxy, kube-dns \
kube-public\
kube-node-lease\
**default**: everything else

**Recommendation**: Do not use default namespaces.

Unlike namespaces, which **create hard boundaries** and are better suited for multi-team or multi-tenant scenarios, \
labels provide a lightweight way to organize and query resources without adding unnecessary complexity. You can easily add multiple labels to the same resource.

Labels: \
Add l**ogical** key-value **metadata** tags to Kubernetes objects like services, deployments, pods, etc. \
Can be used to group/organize objects/resources under a single identity depending upon the use-case/logic they implement, and \
can be used later for **filtering** and finding specific objects using **label Selector**<br>

#### ReplicaSets and Deployment Sets: Deploying Applications

**ReplicaSets:** Using ReplicaSets, define how many **identical** pods should always be running; i.e., they maintain a stable set of replica/identical Pods.\
ReplicaSets continuously monitor the state of pods and automatically create or delete pods to maintain the desired count. Providing high availability\
ReplicaSets by themselves do not update the Pods in case of updates to the containers or Pods<br>

While **Deployments** are often used to **manage** ReplicaSets, for simple **pod count maintenance**, **ReplicaSets** are the fundamental building block.\
Unlike directly editing a ReplicaSet, which doesn't automatically update existing pods, Deployments ensure zero-downtime updates by gradually replacing old pods with new ones. This approach maintains application availability while updating, and if issues arise, you can easily roll back to the previous version

\
**Deployments** manage ReplicaSets to provide declarative updates and advanced deployment strategies like rolling updates, scaling, and version control.\
Describes the number of **desired identical pod replicas** and the preferred update strategy to use when updating. Rolling update is its default strategy. \
Rolling updates, Rollbacks, and pausing the deployment/updates.

Kubernetes creates a new ReplicaSet each time a new Deployment config is deployed and also keeps the old ReplicaSet, so we can roll back to the previous state with the old ReplicaSet.

<figure><img src=".gitbook/assets/image (32).png" alt="" width="383"><figcaption></figcaption></figure>

#### Services: Network Connectivity&#x20;

**Why are services needed?**

1. How do pods communicate:  Through IP or DNS name
2. Pods are **ephemeral**. When they die, a new pod takes their place with a **new IP** address and a new name. If a deployment controller manages them, each pod will have a **random hash** appended to the name of the pod
3. So we cannot rely on the pod IP/DNS names, as they change whenever a pod dies, and a new one replaces it with a completely new identity.
4. How do we **load balance** the incoming requests among various pod replicas? If there are 3 back-end pod replicas, how do we ensure requests are distributed to avoid overloading a single pod?
5. How to **expose the pods outside the cluster** to make the application available for public use?



Groups identical pods together and exposes them over the network using a single fixed source or endpoint, and also provides load balancing to the pods in the group.&#x20;

Services are not the running components; they are just like other objects, like pods. It is simply a logical configuration defined using template files.

**Dynamic Grouping via Labels:** A Service doesn't keep a hardcoded list of Pod IPs. Instead, it uses a **Label Selector** (e.g., `app: backend`). Any Pod with that label automatically gets added to the Service's routing pool.

**Single Fixed Endpoint:** The Service gets a stable internal **ClusterIP** and a permanent **DNS name.**

**Built-in Load Balancing:** By default, `kube-proxy` on every Node configures Linux networking rules (`iptables` or `IPVS`) to distribute incoming requests across all healthy matching Pods (typically round-robin or random).

<div align="left"><figure><img src=".gitbook/assets/image (33).png" alt="" width="563"><figcaption><p>Without Services</p></figcaption></figure></div>

<div align="left"><figure><img src=".gitbook/assets/image (34).png" alt="" width="563"><figcaption><p>With Services</p></figcaption></figure></div>

A Service only looks at **IP addresses and TCP/UDP ports**. It forwards raw network packets to Pods without inspecting the HTTP data inside.

4 main Types:

• **ClusterIP**:  Exposes the Service on an internal IP, only reachable _inside_ the cluster (great for databases or internal microservices).\
\
• **NodePort**:  Opens a static port (between `30000–32767`) on _every_ Worker Node's IP address, allowing external traffic via `<NodeIP>:<NodePort>`.\
\
• **LoadBalancer**: Provisions an external cloud load balancer (like AWS ELB or GCP Load Balancer) with a public IP that routes internet traffic into your cluster.\
\
Services can route traffic to backends outside the cluster (like an external database, legacy VM, or third-party API):

**ExternalName**: Use this when your **external service is managed via a domain name** whose **IP might change** (like an AWS RDS database or SaaS API). It creates a DNS `CNAME` alias inside CoreDNS instead of proxying traffic through `kube-proxy`.

**Use Selector-less Service + EndpointSlice:** if you have **IP addresses**, need to **remap ports** (e.g., internal port `80` -> external port `8080`), or want Kubernetes to **load-balance across multiple external servers**.



<br>

Normally, they are used for Pod communication within the cluster. A Service exists only within the **namespace** it is created in. Tied to the namespace.

```
Service endpoint: my-service.dev.svc.cluster.local

Here:
my-service → Service name
dev → Namespace
svc.cluster.local → Kubernetes service domain
```

\
**A** standard Kubernetes **Service** is a **Layer 4 (TCP/UDP) load balancer** (like an AWS **NLB** - Network Load Balancer), whereas \
An **AWS ALB** (Application Load Balancer) operates at **Layer 7 (HTTP/HTTPS)**—which in Kubernetes is the job of an **Ingress** (or Gateway API).



* **What it CANNOT do:**
  * Route `/api` to one set of Pods and `/login` to another.
  * Terminate SSL/TLS certificates (`https://`) on its own.
  * Inspect HTTP headers, cookies, or authentication tokens.

<div data-with-frame="true"><figure><img src=".gitbook/assets/image (36).png" alt=""><figcaption></figcaption></figure></div>

#### Ingress: Manage external access to services within a Kubernetes cluster.

By default, Kubernetes provides isolation between the cluster and the outside world.\
So, if we want to expose our Applications running inside the cluster to the outside world, we can use

a **Service** (specifically `NodePort` or `LoadBalancer`) \
**type: LoadBalancer:** Automatically provisions an external cloud load balancer (like AWS ELB/NLB, GCP Load Balancer, or Azure Load Balancer) with a public IP address that routes external traffic straight to your Pods. Use for exposing a single app quickly.

**OR**

\
an **Ingress** (or **Gateway API**) resource.&#x20;

**Ingress**: Sits in front of internal `ClusterIP` Services to expose multiple web apps through a **single external IP address**, routing traffic based on **domain names** (`://example.com`), and **URL paths** (`/api`, `/login`) while **handling SSL/TLS certificates.** \
This is the standard production choice for web applications and APIs.

Two **core** **components** of Ingress:\
The object spec(Ingress):  for defining traffic rules\
The controller:  for implementing them. \
An **Ingress** is a Kubernetes YAML configuration object that defines HTTP/HTTPS routing rules (like which URL path or domain goes to which Service), while \
an **Ingress Controller** is the actual reverse-proxy application (like NGINX, Traefik, or AWS ALB) running inside your cluster that reads those rules and routes the live network traffic.

An **Ingress** is only a YAML file with routing rules (e.g. "`/api` goes to service A"). It does nothing by itself. Something has to read those rules and enforce them, and that something is the **Ingress Controller**.

**Model 1:** In-cluster controller (NGINX, Traefik, HAProxy): The controller is a Pod, so the proxy lives inside the cluster. To reach it from outside, you put a Service in front of it.

The difference is where the reverse proxy runs.

**Model 2:** AWS Load Balancer Controller (ALB): The ALB is the reverse proxy, and it lives outside the cluster as a managed AWS resource.

An **Ingress Controller** is just a regular Pod running inside your cluster, so on its own, it is still trapped behind Kubernetes' internal network isolation. To let the outside world reach that Ingress Controller Pod, you expose **the Ingress Controller itself** using a single **`Service` (typically `type: LoadBalancer` or `NodePort`)**, and then point your public **DNS (`A` or `CNAME` record)** to that single external IP!

```
Internet → Cloud LB (NLB/AlB) → NodePort → NGINX Pod → Service → App Pods
```

<figure><img src=".gitbook/assets/image (37).png" alt=""><figcaption></figcaption></figure>





**Security**: Applications in private subnets only, while the Cloud Load Balancer is in Public.\
**Cost Efficiency:** Instead of paying for 20 separate Cloud Load Balancers for 20 microservices, you pay for **1 Cloud Load Balancer** that feeds into your Ingress Controller.



Cloud-Managed Ingress Controller (AWS Load Balancer Controller, GKE Ingress):  As it is a controller, it will sit in the Control plane.<br>

Here, the **AWS Load Balancer Controller** runs as a lightweight management Pod inside your cluster, **but it does NOT proxy any user traffic itself**. Instead, whenever you create an `Ingress` YAML, it calls the AWS API to configure a real **AWS Application Load Balancer (ALB)** living directly in your public subnets.

\
Ingress operates at Layer 7 of the OSI model, using host-based and path-based routing to direct traffic to appropriate backend services.\
Path-based routing allows you to direct traffic based on URL paths (like /api or /admin), while host-based routing enables routing based on different domain names.

_**Fun fact:**_ \
_When you deploy an **Ingress** on AWS EKS using the AWS Load Balancer Controller, Kubernetes literally provisions a real **AWS ALB** under the hood!_



Every Kubernetes YAML file follows the same **4-part skeleton** \
(`apiVersion`, `kind`, `metadata`, `spec`)&#x20;

```

apiVersion: apps/v1        # 1. A - Which version of the K8s API to use
kind: Deployment           # 2. K - What OBJECT you want to create (Pod, Service, Deployment)
metadata:                  # 3. M - Data that identifies this object (name, labels)
  name: my-app
  labels:
    app: my-app
    
spec:                      # 4. S - Desired state (what you actually want K8s to build)
  # Everything specific to the "kind" goes here

```



### Kubectl

* **Kubectl** is the command-line utility used to **interact with a Kubernetes cluster**, like a client.
* Kubernetes is **fully controlled through its REST API**. Every Kubernetes operation is exposed as an **API endpoint** and can be executed via **HTTP requests**.
* `kubectl` Interacts with the cluster by consuming these API endpoints.
* The process involves taking human-readable YAML/JSON **manifest files(**&#x75;sed to define the objects in Kubernetes that need to be created), and converting them into JSON payloads that are then sent securely over HTTPS to the API server. This architecture ensures **consistent** and **secure communication** while allowing users to work with more human-friendly YAML files for resource definitions.&#x20;

```
kubectl get pods/replicasets/deploymentsets
kubectl create -f manifest.yaml
kubectl describe pod <name>
kubectl delete pod <name>
```

***

### Introduction to AWS EKS Cluster

Cluster IAM role\
To access other AWS services and resources your Kubernetes control plane needs an IAM role.

**The control plane IAM role** is only for managing cluster infrastructure (like creating load balancers or network interfaces), whereas **your application Pods on Worker Nodes** use a separate IAM role (via **IRSA** or **EKS Pod Identity**) to access AWS services like S3.\
**Cluster IAM Role** (for the Control Plane) is strictly for **Kubernetes infrastructure operations**, not your business logic:

* Provisioning AWS Load Balancers when you create a `Service` of `type: LoadBalancer`.
* Managing Elastic Network Interfaces (ENIs) and security groups for Pods.
* Sending cluster logs to CloudWatch and pulling metrics.



_Your application Pods never inherit or use this Control Plane IAM role._

What Happens When Your **App** Needs to Access S3 (Worker Nodes / Pods): Your application code runs inside a **Pod on a Worker Node**, so the **Pod** makes the API call directly to S3. There are two ways it gets AWS permissions:

The Modern Best Practice: Pod-Level IAM (**EKS Pod Identity** or **IRSA**)\
Instead of giving permissions to the entire Worker Node (which would let _every_ Pod on that server read your S3 bucket), you attach an IAM Role directly to a specific Kubernetes **ServiceAccount** used by your Pod:

1. You create an **IAM Role** with `s3:GetObject` permission for your bucket.
2. You link that IAM Role to a Kubernetes **ServiceAccount** (using **EKS Pod Identity** or **IRSA** — _IAM Roles for Service Accounts_).
3. In your Pod/Deployment YAML, you specify `serviceAccountName: my-s3-reader`.
4. **Under the hood:** EKS automatically injects temporary, short-lived AWS credentials (rotated automatically) into that Pod's environment variables. When your app's AWS SDK calls `s3.getObject()`, it uses those temporary credentials to fetch the file directly from S3 over HTTPS—**the Control Plane isn't involved at all.**

The Old / Less Secure Way: Worker Node **Instance Profile:** Every EC2 Worker Node also has an **EC2 Node IAM Role** (used by `kubelet` to pull Docker images from ECR and join the cluster).

If you don't set up Pod Identity/IRSA, any Pod on that Worker Node falls back to using the **Worker Node's EC2 IAM Role**.
