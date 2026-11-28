---
description: Container Management System
---

# Amazon EKS - Knowledge Badge

[Reference URL](https://skillbuilder.aws/learning-plan/FRN6Z7KMRY/amazon-eks--knowledge-badge-readiness-path)

* Container Basics
* Why AWS EKS for Kubernetes Workloads
* Introduction to Kubernetes Core Concepts
* Introduction to AWS EKS Cluster



* Deploying Microservices to Amazon EKS
* EKS Security
* EKS Networking
* Observabilty
* Autoscaling and Cost Optimization
* GitOps

***

### Why AWS EKS for Kubernetes Workloads

Why run your Kubernetes workloads on Amazon Elastic Kubernetes Service (EKS)?

Access to the vast ecosystem, Karpenter for Auto Scaling

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
\
**Control Plane Nodes**: Kubernetes Orchestration (management) Unit.\
Control plane nodes are responsible for orchestration-related tasks such as scaling, updating, and launching applications, essentially serving as the "**brain**" of the cluster.\
Running Components in Control Plane Nodes:\
• **ETCD:** the cluster's database storing all cluster state in key-value pairs. Stateful.\
• **kube-API Server**: handles all API operations. Exposes Kubernetes APIs. Front door to the Kubernetes control plane. All API clients, whether they're external tools like kubectl or internal components like the scheduler or controller manager, communicate with the cluster through the API Server. Central management entity that processes and validates all REST requests. Responsible for persisting the cluster's state in etcd. Acts as a central communication hub.\
• **kube-Controller Manager**: runs controller processes. eg node controller, deployment controller, etc. Provides **self-healing** ability to the cluster. Controllers continuously monitor the cluster's state, and when they detect a deviation from the desired state (such as when a pod fails, or when there aren't enough replicas running), they automatically take corrective action\
• **kube-Scheduler:** responsible for scheduling pods on nodes.

\
&#x20;

<div align="left"><figure><img src=".gitbook/assets/image (30).png" alt="" width="563"><figcaption></figcaption></figure></div>

\
**Worker Nodes**: Kubernetes workload units. The application lives here.\
Nodes/Machines where the actual application workloads run, hosting the pods that contain your containerized applications.

\
Management component running in data Plane Nodes(Worker nodes):\
• **kubelet**: Listens for instructions from the control plane and performs the requested actions. It's responsible for starting, stopping, and maintaining application containers as directed by the control plane. By having a kubelet on each node, Kubernetes ensures that every node can be managed centrally.\
• **kube-proxy:** Handles networking in worker nodes\
• **Container Runtime Engine**: like containerd, Docker Engine.

<br>

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

<div align="left"><figure><img src=".gitbook/assets/image (34).png" alt="" width="563"><figcaption><p>With Services: Stable Network Abstraction layer in front of Pods</p></figcaption></figure></div>

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
An **Ingress** (or **Gateway API**) resource.&#x20;

**Ingress**: Sits in front of internal `ClusterIP` Services to expose multiple web apps through a **single external IP address**, routing traffic based on **domain names** (`://example.com`), and **URL paths** (`/api`, `/login`) while **handling SSL/TLS certificates.** \
This is the standard production choice for web applications and APIs.

Two **core** **components** of Ingress:\
The object spec(Ingress):  for defining traffic rules\
The controller:  for implementing them. \
An **Ingress** is a Kubernetes YAML configuration object that defines HTTP/HTTPS routing rules (like which URL path or domain goes to which Service), while \
an **Ingress Controller** is the actual reverse-proxy application (like NGINX, Traefik, or AWS ALB) running inside your cluster that reads those rules and routes the live network traffic.

An **Ingress** is only a YAML file with routing rules (e.g.,"`/api` goes to service A"). It does nothing by itself. Something has to read those rules and enforce them, and that something is the **Ingress Controller**.

**Model 1:** In-cluster controller (NGINX, Traefik, HAProxy): The controller is a Pod, so the proxy lives inside the cluster. To reach it from outside, you put a Service in front of it.

The difference is where the reverse proxy runs.

**Model 2:** AWS Load Balancer Controller (ALB): The ALB is the reverse proxy, and it lives outside the cluster as a managed AWS resource.

An **Ingress Controller** is just a regular Pod running inside your cluster, so on its own, it is still trapped behind Kubernetes' internal network isolation. To let the outside world reach that Ingress Controller Pod, you expose **the Ingress Controller itself** using a single **`Service` (typically `type: LoadBalancer` or `NodePort`)**, and then point your public **DNS (`A` or `CNAME` record)** to that single external IP!

```
Internet → Cloud LB (NLB/AlB) → NodePort → NGINX Pod → Service → App Pods
```

<figure><img src=".gitbook/assets/image (37).png" alt=""><figcaption></figcaption></figure>





**Security**: Applications in private subnets only, while the Cloud Load Balancer is public.\
**Cost Efficiency:** Instead of paying for 20 separate Cloud Load Balancers for 20 microservices, you pay for **1 Cloud Load Balancer** that feeds into your Ingress Controller.



Cloud-Managed Ingress Controller (AWS Load Balancer Controller, GKE Ingress):  As it is a controller, it will sit in the Control plane.<br>

Here, the **AWS Load Balancer Controller** runs as a lightweight management Pod inside your cluster, **but it does NOT proxy any user traffic itself**. Instead, whenever you create an `Ingress` YAML, it calls the AWS API to configure a real **AWS Application Load Balancer (ALB)** living directly in your public subnets.

\
Ingress operates at Layer 7 of the OSI model, using host-based and path-based routing to direct traffic to appropriate backend services.\
Path-based routing allows you to direct traffic based on URL paths (like /api or /admin), while host-based routing enables routing based on different domain names.

_**Fun fact:**_ \
_When you deploy an **Ingress** on AWS EKS using the AWS Load Balancer Controller, Kubernetes literally provisions a real **AWS ALB** under the hood!_



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

### Introduction to AWS EKS

AWS-managed service; manages the Control Plane. EKS removes the operational burden of maintaining Kubernetes infrastructure by automatically handling high availability, scaling, and security patches across multiple availability zones for the Control Plane Nodes.

#### EKS's architecture: Single-tenant control

<figure><img src=".gitbook/assets/image (41).png" alt=""><figcaption><p>All this is abstractedn form us, amanged by AWS solely.</p></figcaption></figure>

Node Group Types: (based on **control** and **convenience**)\
Managed Node Group:  EKS creates and manages the EC2 instances for us(including OS/AMI)\
Self-managed Node Group: We create & manage EC2, select our preferred Instance type, OS/AMI.\
Fargate: Serverless, on-demand. We don't have control over our EC2.

**Cluster DNS:** Entry Point to the Kubernetes cluster. It points to the API server.

**Launch AWS EKS Cluster:**

**Cluster IAM Role/Cluster Service Role**:  \
An IAM role that allows the EKS service to create and manage AWS resources necessary for operating your Kubernetes cluster, such as EC2 instances or load balancers. Without this role, EKS would not be able to interact with other AWS services, severely limiting its functionality.

**The Control Plane IAM Role** is only for managing cluster infrastructure (like creating load balancers or network interfaces),     whereas \
Y**our application Pods on Worker Nodes** use a separate IAM role (via **IRSA** or **EKS Pod Identity**) to access AWS services like S3.\
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



### EKS Compute&#x20;

#### Node group

A node group is a group of EC2 instances that supply compute capacity to your Amazon EKS cluster. You can add multiple node groups to your cluster.

**Node IAM role**\
To access other AWS services and resources, your nodes need an IAM role. Choose a role with the right permissions for your Nodes.\
The node IAM role **gives permissions to the `kubelet`** running on the node to make calls to the Kubernetes API and other AWS APIs on your behalf. This includes permissions to access **container registries** where your application containers are stored.

When you create a managed node group, Amazon **EKS will create a service-linked role** for Amazon EKS to use to create and manage the managed node groups.

Trusted Identity: AWS Services (Services that can assume this Role)\
Use Case: EC2\
\
Policy: \
AmazonEKSWorkerNodePolicy: Allows AWS Amazon EKS worker nodes to connect to Amazon EKS Clusters.\
AmazonEC2ContainerRegistryReadOnly: Provides read-only access to Amazon EC2 Container Registry repositories.\
AmazonElasticContainerRegistryPublicReadOnly: Provides read-only access to Amazon ECR Public repositories.\
Amazon\_EKS\_CNI\_Policy: This policy provides the Amazon VPC CNI Plugin (amazon-vpc-cni-k8s) the permissions it requires to modify the IP address configuration on your EKS worker nodes. This permission set allows the CNI to list, describe, and modify Elastic Network Interfaces on your behalf. https://github.com/aws/amazon-vpc-cni-k8s<br>

**NOTE**:  **Bottlerocket** is an open-source operating system(AMI) developed by AWS specifically for running containers. It's designed with security, operations, and manageability in mind, featuring a minimalist design that reduces the attack surface and improves resource utilization.



#### Service Account:

#### Fargate Profile

Fargate profiles let you use Fargate to run Kubernetes pods.<br>

#### Kubeconfig File

**`kubectl`** is the command-line tool you use to talk to a Kubernetes cluster, and \
**`kubeconfig`** is the configuration file (typically stored at **`~/.kube/config`**) that \
tells `kubectl` _which_ cluster to connect to, _where_ its API server is located, & _**how**_**&#x20;to authenticate you.**

A `kubeconfig` file is just a YAML file organized into three main sections:

* **`clusters`:** The connection details for each Kubernetes cluster you have access to (the API Server's HTTPS URL and its Certificate Authority data to verify TLS).
* **`users`:** Your login credentials (client certificates, bearer tokens, or an `exec` plugin command like `aws eks get-token` that fetches temporary credentials).
* **`contexts`:** The glue that pairs a **Cluster** + a **User** + a default **Namespace** under a friendly nickname (e.g., `dev-cluster` vs. `prod-cluster`), so you can switch between multiple clusters with one command.

1\. How They Work Together. \
Every time you type a command like `kubectl get pods`:

1. **`kubectl`** reads your **`kubeconfig`** file to find your currently active **context**.
2. It grabs the **API Server URL** and your **authentication credentials** (like a certificate or AWS IAM token) from that context.
3. It sends an HTTPS REST API request to your cluster's Control Plane and displays the result in your terminal.

```bash
#Generating / Updating kubeconfig on AWS EKS
#Once your EKS cluster finishes creating, you don't write the kubeconfig file by hand
#—the AWS CLI generates and merges it into ~/.kube/config automatically:

aws eks update-kubeconfig --region ap-south-1 --name my-cluster-name

kubectl config view
kubectl config get-contexts

```

#### EKS Cluster Version Upgrade

Kubernetes is a very fast-paced, open-source, ongoing project. Because EKS is based on it, AWS also updates EKS regularly.  So we have to also keep our EKS cluster updated.

3-step Process:\
**Upgrade the Control Plane:**\
Each major update brings a lot of changes. Make sure to double-check these features in the AWS EKS docs and Kubernetes docs, and fail-safe your application before updating the EKS cluster by using a **rolling update strategy** to **minimize downtime** and **maintain high availability** during the upgrade process.\
It updates the Kubernetes version running on the A**WS-managed Control plane** using a rolling update strategy.&#x20;

**Upgrade the Data Plane (Worker Node Group)**\
No upgrade to the **Data Plane.** ie K8s components working on Worker Nodes are still using the previous version components. Kubernetes allows one backward version compatibility for Worker Node Groups compared to the Control Plane. The **one-version difference** allowance provides **flexibility** in the upgrade process, enabling you to upgrade the control plane first and then bring the nodes up to date. You must upgrade this Worker Node group version manually after the control plane version upgrade. \
It offers **Rolling Update** as well as **Force Update**. Once updated, Old Nodes running the previous version of K8s will be replaced by New Nodes with the upgraded version. A **rolling update strategy gradually** replaces old nodes with new ones, ensuring that there's always sufficient capacity to run your workloads. This approach **minimizes downtime** and risk, as it allows you to carefully control the pace of the update and quickly roll back if issues are detected.

**Upgrade the add-ons:**\
Add-ons need to be upgraded separately to match the current Cluster version.



***

## Deploying Microservices to Amazon EKS

#### Imperative vs Declarative

**Imperative** means telling Kubernetes _how_ to do something step-by-step using direct `kubectl` commands, whereas \
**Declarative** means defining _what_ final state you want inside YAML files and letting Kubernetes figure out how to achieve it. i.e **YAML Manifests + `kubectl apply`**

**Pro Tip**: Combine Both (The Hybrid Workflow): \
Use **imperative commands** to generate clean YAML files quickly, then manage them **declaratively**:

```bash
# Generate YAML imperatively without creating anything yet
kubectl create deployment my-app --image=nginx --dry-run=client -o yaml > app.yaml

# Deploy and manage declaratively from now on
kubectl apply -f app.yaml

```



Every Kubernetes YAML file follows the same **4-part skeleton** \
&#x20;**A-K-M-S:**  `apiVersion`, `kind`, `metadata`, `spec`

```yaml
apiVersion: apps/v1        # 1. A - Which version of the K8s API to use
kind: Deployment           # 2. K - What OBJECT you want to create (Pod, Service, Deployment)
metadata:                  # 3. M - Data that identifies this object (name, labels)
  name: my-app
  labels:
    app: my-app
    
spec:                      # 4. S - Desired state (what you actually want K8s to build)
  # Everything specific to the "kind" goes here

```

**Boilerplate** (Deployment + Service)    **A.K.M.S**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-app
spec:
  replicas: 2                    # How many copies of the Pod to run
  selector:
    matchLabels:
      app: web-app               # Must match the template labels below!
      
  template:                      # <-- This entire section is just a Pod blueprint
    metadata:
      labels:
        app: web-app
    spec:
      containers:
      - name: nginx              # Note the hyphen (-) for list items
        image: nginx:latest
        ports:
        - containerPort: 80
        
---

apiVersion: v1
kind: Service
metadata:
  name: web-app-service
spec:
  type: ClusterIP                # Use LoadBalancer or NodePort for external access
  selector:
    app: web-app                 # Routes traffic to Pods with this label
  ports:
  - port: 80                     # Port exposed by the Service
    targetPort: 80               # Port your container is listening on

```

**Annotations** are non-identifying metadata key-value pairs attached to Kubernetes objects to pass **extra configuration instructions to external controllers, tools, or cloud providers** (like the AWS Load Balancer Controller) that don't fit inside Kubernetes' standard `spec:` fields.

**Labels vs. Annotations** (How to Know Which One to Use)\
Both live under `metadata:` and look like `key: value` pairs, so people often mix them up. Use this simple rule:

* **Labels = For Kubernetes to select & group objects** (e.g., matching a Service to Pods, or filtering with `kubectl get pods -l app=orders`).
* **Annotations = For external controllers & humans to read extra settings** (e.g., telling AWS how to configure an ALB, attaching an SSL certificate ARN, or storing Git commit info). \
  _&#x4B;ubernetes itself ignores annotations—only the controller watching that object reads them._

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: alb-ingress
  namespace: ecommerce
  labels:
    app.kubernetes.io/name: alb-ingress
  annotations:
    kubernetes.io/ingress.class: alb
    alb.ingress.kubernetes.io/scheme: internet-facing 
    alb.ingress.kubernetes.io/target-type: ip 
    alb.ingress.kubernetes.io/listen-ports: '[{"HTTP": 80}]'


#Because Kubernetes' built-in Ingress spec only knows about basic host and path rules, 
#the AWS Load Balancer Controller uses alb.ingress.kubernetes.io/... annotations to know how to build the AWS ALB:
• kubernetes.io/ingress.class: alb (Note: In modern K8s networking.k8s.io/v1, spec.ingressClassName: alb is the newer replacement for this annotation, though AWS still supports both). Tells the AWS Load Balancer Controller: "Hey, this Ingress belongs to you—don't let NGINX touch it."
• alb.ingress.kubernetes.io/scheme: internet-facing: Tells AWS to create a public ALB in your public subnets with a public DNS name (as opposed to internal, which creates a private VPC-only ALB).
• alb.ingress.kubernetes.io/target-type: ip: Tells the AWS ALB to register Pod IPs directly as targets (required for Fargate and best for EKS VPC CNI), rather than routing through EC2 NodePorts (instance).
• alb.ingress.kubernetes.io/listen-ports: '[{"HTTP": 80}]': Tells the ALB which ports to open on its listener (e.g., HTTP on 80, or '[{"HTTP": 80}, {"HTTPS": 443}]' when adding SSL).

```

**3 situations when you need an annotation**:

1. **You are configuring a cloud-specific feature that doesn't exist in standard Kubernetes:**
   * _Examples:_ Making an AWS ALB `internet-facing`, attaching an AWS ACM SSL certificate (`alb.ingress.kubernetes.io/certificate-arn`), or requesting an internal AWS NLB on a `Service`.
   * _How to find it:_ Search `"AWS Load Balancer Controller annotations"`—every single one starts with `alb.ingress.kubernetes.io/` (for Ingress) or `service.beta.kubernetes.io/aws-load-balancer-` (for Services).
2. **You are configuring a third-party Ingress Controller's behavior:**
   * _Examples:_ URL rewriting, rate-limiting, or max body upload size in NGINX (`nginx.ingress.kubernetes.io/rewrite-target`).
3. **You are integrating an add-on tool (like `cert-manager` or `external-dns`):**
   * _Examples:_ Telling `cert-manager` to auto-generate a Let's Encrypt SSL certificate (`cert-manager.io/cluster-issuer: letsencrypt-prod`).



### Stateful Workload:

Generally, Kubernetes is associated with running **stateless** applications.

Three main purposes for preserving data in microservices: \
persistence, \
streaming, and \
batch processing&#x20;

Databases are the most common example for preserving data. Session Data another.

\
\- Three types of storage:\
File: \
Block: EBS\
Object: S3<br>

Pods are ephemeral/temporary in nature, and so is their data, which is in containers(also ephemeral). So, to make their data persist beyond their lifecycle, we need to store data to disk. \
Enter **Kubernetes Volumes.** Volumes allow pods/containers to store data on disk that will persist regardless/independent of the container/pod.

Volumes solve the critical problem of data persistence in containerized environments - while containers are ephemeral and lose their data when they restart, volumes provide a way to store data that persists beyond the container's lifecycle. They act as a **bridge** between the ephemeral nature of containers and the need for persistent data storage. This is crucial for applications like **databases**, **file storage systems**, or any application that needs to maintain state.

<div data-with-frame="true"><figure><img src=".gitbook/assets/image (38).png" alt="" width="367"><figcaption><p>Kubernetes Volumes</p></figcaption></figure></div>

\
Kubernetes Volumes components:\
• Persistent Volumes (PVs)\
• Persistent Volume Claims (PVCs) \
• Storage Classes

Persistent Volumes define the actual storage resources available for the cluster - they're like an inventory of available storage. PVCs, on the other hand, are requests for storage by users/applications,\
PVs can be created in two ways: statically and dynamically. \
**Static provisioning:** explicitly creating storage using a manifest file. Here, a PVC binds to an entire PV, not a "segment" or slice of it.

The 1-to-1 Binding Rule (The Crucial Nuance): \
When you use **Static Provisioning**, if an administrator manually creates a `100Gi` PV and your application's PVC only asks for `10Gi`Kubernetes will bind that PVC to the **entire `100Gi` PV**.

* The remaining `90Gi` cannot be claimed by another PVC—it is effectively wasted for other applications.
* Because of this 1-to-1 relationship, static PVs must be sized carefully to match the expected PVC requests.

\
**Dynamic Provisioning:** Don't have to explicitly create the PVs. No manual `PersistentVolume` YAML is needed ahead of time. The developer creates a `PersistentVolumeClaim` pointing to a **StorageClass (SC)**.\
The StorageClass automatically talks to the storage provider (like AWS EBS, GCP PD, or Azure Disk), provisions a disk of the _**exact**_**&#x20;size** requested by the PVC, generates the PV object behind the scenes, and binds them together.

<figure><img src=".gitbook/assets/image (39).png" alt="" width="563"><figcaption></figcaption></figure>

A PV is the actual piece of physical or cloud storage in the cluster. While Pods and PVCs belong to a specific namespace, PVs exist at the cluster level.

* **How it gets created:** Either an administrator creates it manually ahead of time (Static Provisioning), or the StorageClass automatically generates it the moment a PVC asks for space (Dynamic Provisioning).
* **Lifecycle:** Once a PV is matched (or **bound**) to a PVC, it belongs exclusively to that claim until the PVC is deleted.

A PVC is a request for storage made by a developer/application. Your **Pod mounts the PVC**, not the underlying disk directly.

A PVC and PV bind **one-to-one** and exclusively - once bound, the pairing holds until the PVC is deleted. When a PVC is deleted, the PV moves to `Released` and its **reclaim policy** decides what happens: `Delete` removes the backing volume; `Retain` keeps the PV (and data) but it will not rebind automatically - an admin must clean it and remove the old `claimRef` first. A `Released` PV that "won't bind" to a new PVC is expected behavior, not a bug.

You attach a PVC to your Pod configuration so your app has a place to save files.\
**What it defines:** How much space the app needs (e.g., `10Gi`), which StorageClass tier it wants, and how the volume will be accessed (`AccessModes`):&#x20;

* **ReadWriteOnce (RWO):** Can be mounted as read-write by a _single_ node (standard for most databases).
* **ReadOnlyMany (ROX):** Can be mounted as read-only by _many_ nodes simultaneously.
* **ReadWriteMany (RWX):** Can be mounted as read-write by _many_ nodes at the same time (common for shared file uploads using NFS or EFS).

StorageClass: A template that tells Kubernetes how to dynamically create storage on the fly when a PVC is requested. \
**What it defines:** The storage provider (e.g., AWS EBS, Google Persistent Disk, Azure Disk, or local NFS), \
performance tiers (SSD vs. HDD), **and** \
what happens to the data when a user is done with it (`ReclaimPolicy`: either `Delete` the disk or `Retain` the data).



### StatefulSets and ReplicaSets

Headless services: A Headless Service (`clusterIP: None`) is a K8s Service that does not allocate a single virtual IP and **does not load-balance traffic.**  No proxying; the client connects directly to the Pod IP.\
**Instead**, it returns the individual DNS records (IPs) of each backing Pod so clients can connect to specific Pods directly.

Unlike Deployments or ReplicaSets where pods get random identifiers, StatefulSets maintain a stable, predictable identity for each pod, even after rescheduling. This stable identity is crucial for stateful applications because it ensures consistent network identities, stable persistent storage, and ordered deployment and scaling.\
**Each pod** in a StatefulSet gets its **own dedicated PVC**, ensuring that even if a pod is rescheduled to a different node, it will still have access to its own unique persistent storage. This is vital for stateful applications where each instance needs its own stable storage

StatefulSets are for workloads that need stable identity and persistent storage per replica. Typical examples include databases, queue brokers, and clustered stateful systems.

• Stable name: Stable pod ordinal names: `app-0`, `app-1`, `app-2`

• Stable DNS address: Stable DNS names through a headless service

• Stable volume: One PersistentVolumeClaim per pod from `volumeClaimTemplates`

• Ordered rollout and scale behavior by default

A StatefulSet also provides an important safety property: **at most one pod per identity**. \
Before replacing `app-1`, the controller must be certain the old `app-1` is gone - two processes writing the same volume or claiming the same cluster membership would corrupt data. \
This is why StatefulSet pods on a failed node are not replaced automatically until the node object is deleted or the pod is force-deleted:





<br>

<br>

### Required Companion: Headless Service <a href="#required-companion-headless-service" id="required-companion-headless-service"></a>

StatefulSets rely on a headless Service for stable DNS records.

**Why StatefulSets Need a Headless Service**\
Imagine a 3-node database cluster (`db-0` is primary, `db-1` and `db-2` are read replicas):

* With a **regular Service**, hitting `db-svc` randomly routes a write query to `db-1` (a read-only replica), causing errors.
* With a **Headless Service** (`serviceName: db-headless`), Kubernetes creates predictable per-Pod DNS entries:
  * `db-0.db-headless.default.svc.cluster.local` → points directly to `db-0`'s Pod IP
  * `db-1.db-headless.default.svc.cluster.local` → points directly to `db-1`'s Pod IP

When `db-0` dies and gets rescheduled with a **new IP**, CoreDNS automatically updates `db-0.db-headless...` to point to the new IP.



**StatefulSets**: Kubernetes resource that allows you to manage stateful workloads using stable naming and state. Stateful apps (Databases, Kafka, ZooKeeper, Elasticsearch)

Similar to ReplicaSets in that both manage the lifecycle of an identical set of Pods. **BUT**,\
StatefulSets make sure Pods **retain their identity(in what sense?)** even after they are rescheduled, & it manage Pods with ordered indexing; like if we have two replica Pods, they will have a suffix in an ordered approach, unlike ReplicaSets, which have unique naming.

ReplicaSets share PVCs across all their Pods, whereas StatefulSets create PVCs per pod replica(again, confused??)

_Does a Headless Service use `kube-proxy`?_\
**Answer:** No. Because there is no `ClusterIP`, `kube-proxy` creates no `iptables`/`IPVS` rules for it. Service discovery happens purely through **CoreDNS**.



<div data-with-frame="true"><figure><img src=".gitbook/assets/image (40).png" alt="" width="563"><figcaption></figcaption></figure></div>

Why does a regular Service break stateful apps??\
**Answer**: A regular Service breaks stateful apps because it uses random load balancing behind a single IP, treating all Pods as identical when stateful Pods actually have distinct roles and data. \
A regular Service assumes Pods are interchangeable and stateless, so it hides their identities behind a load balancer. But StatefulSet Pods have unique roles and data, requiring a Headless Service so clients and peers can address specific Pods directly by DNS name.

{% code expandable="true" %}
```
/Interview Answer:
1. Asymmetric Roles (Primary vs. Replica)

• The problem: In a database cluster (like PostgreSQL or MySQL), db-0 is the Primary (handles writes) and db-1/db-2 are Replicas (read-only).
• What breaks: A regular ClusterIP Service blindly round-robins traffic across all three Pods. A WRITE request sent to the Service has a 66% chance of landing on a read-only replica and failing.

2. Peer-to-Peer Discovery & Leader Election

• The problem: Distributed systems (like Kafka, ZooKeeper, etcd, or Cassandra) need nodes to talk directly to each other to sync data, form a quorum, and elect a leader.
• What breaks: Nodes must know the exact list of peers (e.g., "I am node 0, talking to node 1 and node 2"). A regular Service hides individual nodes behind one IP, so a node might accidentally talk to itself or fail to build a cluster membership list.

3. Data Sharding / Partitioning

• The problem: In systems like Redis Cluster or Elasticsearch, data is split across shards—for example, shard-0 holds users A–M, and shard-1 holds users N–Z.
• What breaks: If a client queries a regular Service for User Z, the Service might route the request to shard-0, which doesn't have that data. Clients need direct addressing to hit the exact Pod holding that shard.

4. Broken Replication & Split-Brain Risks

• The problem: Replica Pods (db-1) need a stable target to pull transaction logs from (db-0).
• What breaks: If db-1 tries to replicate from a regular Service IP, it could connect to another replica or itself instead of the primary, halting replication or causing cluster inconsistency.

```
{% endcode %}

Headless services, StatefulSets, storage classes, and the Amazon EBS CSI driver for managing persistent storage in EKS.

If the workload is replaceable and does not require replica identity, use Deployment for simpler operations. Do not use StatefulSet just because an app writes logs or temp files.

<pre class="language-yaml"><code class="lang-yaml"># StatefulSets must be in the manifest YAML file:

specs:
<strong>  ServiceName: &#x3C;headless-service-name>        #Headless service reference
</strong><strong>  ...
</strong><strong>  ...
</strong><strong>  template:
</strong><strong>    specs:
</strong><strong>    ...
</strong><strong>    ...
</strong><strong>    volumeMounts:
</strong><strong>    -  name: 
</strong><strong>       mountPath:
</strong><strong>   
</strong><strong>  volumeClaimTemplates: 
</strong><strong>  -  metadata: 
</strong><strong>       name:
</strong><strong>     specs:
</strong>       storageClassname: 
       accessModes: 
       resources:
         requests: 
           storage: 2Gi
</code></pre>

**CSI Driver Controller:**\
A CSI (Container Storage Interface) driver controller in Amazon EKS manages volume lifecycle operations—such as provisioning, attaching/detaching, and snapshotting—for Kubernetes workloads across EBS, EFS, and S3.&#x20;

EBS CSI Driver Controller (`ebs.csi.aws.com`)

* **Role:** Runs as a Kubernetes Deployment (`ebs-csi-controller`) using a service account `ebs-csi-controller-sa` with IAM permissions (e.g., `AmazonEBSCSIDriverPolicyV2`) to create, delete, attach, detach, and snapshot EBS volumes.&#x20;
* **Storage Type & Access:** Block storage (`ReadWriteOnce`), bound to a **single Availability Zone** (AZ).&#x20;
* **Best For:** **Single-pod stateful** workloads and **databases** (PostgreSQL, MySQL) needing low-latency block I/O.
* **Key Notes:** Runs on EC2 nodes (not supported on Fargate or EKS Hybrid Nodes); \
  Volume attachment/detachment can add startup latency across pod rescheduling.

EFS CSI Driver Controller (`efs.csi.aws.com`)

* **Role:** Deploys `efs-csi-controller` (using `efs-csi-controller-sa`) to provision volumes/access points, paired with the `efs-csi-node` DaemonSet to mount NFS file systems onto nodes.
* **Storage Type & Access:** Elastic NFS file system supporting `ReadWriteMany` across **multiple AZs** and pods simultaneously. (EFS CSI driver v3.0.0+ also supports mounting Amazon S3 file systems).
* **Best For:** **Shared multi-pod**, **multi-AZ** file storage, CMS platforms, and POSIX-compliant workloads.
* **Key Notes:** Requires EFS mount targets in worker subnets and security group rules allowing NFS traffic.

S3 CSI Driver Controller (`s3.csi.aws.com`)

* **Role:** Integrates Amazon S3 object storage with Kubernetes PersistentVolumes/PersistentVolumeClaims via Mountpoint for Amazon S3.
* **Storage Type & Access:** Object storage exposed as a file system interface (not POSIX-compliant, eventually consistent; traditionally static provisioning, with v2 mountpoint pods requiring capacity/headroom planning).
* **Best For:** **Read-heavy workloads**, data lakes, **analytics**, and **large object/file access**.



**Other Notables:**\
**Amazon EC2 Instance Store CSI Driver (`lis.csi.aws.com`).**\
**Amazon FSx for Lustre (`fsx.csi.aws.com`):**  High-performance parallel file system geared toward **HPC**, **AI/ML** training, and massive batch data processing rather than transactional databases.\
**Longhorn:** Cloud-agnostic, Lightweight, CNCF-backed distributed **block storage** that replicates volumes across nodes and backs up snapshots to S3



#### Kubernetes StatefulSets, Replication & AWS DevOps Architecture&#x20;

1\. **Why Databases Need Separate Storage Per Pod.** When running databases in Kubernetes `StatefulSet`, each pod gets its own dedicated `PersistentVolumeClaim` (`volumeClaimTemplates`) rather than sharing a single volume:

* **Data Corruption Prevention:** Standard databases cache data in RAM (Buffer Pools). If a secondary pod reads disk files while the primary modifies them in memory, it reads incomplete writes ("torn pages") and corrupted indexes.
* **Split-Brain Protection:** If a network glitch promotes a secondary pod while the primary is still alive, shared storage causes both primaries to overwrite the exact same disk blocks simultaneously, permanently destroying the database.
* **No Single Point of Failure (SPOF):** If one shared volume or availability zone fails, all replicas go down together. Separate disks ensure independent, full data copies.
* **IOPS Isolation:** Shared volumes force read replicas to compete with the primary for disk I/O, bottlenecking critical write operations.
* **Storage Protocol Limits:** Fast cloud block storage (AWS EBS) is **ReadWriteOnce (RWO)** (single-node attachment only). Sharing across nodes requires network storage like NFS (**ReadWriteMany / RWX**), adding unacceptable latency.

2\. **How Replicas Stay Updated** (Application-Level Replication): Instead of disk sharing, databases replicate over the network via **Transaction Log Streaming** (**WAL** in Postgres, **Binlog** in MySQL, **Oplog** in MongoDB):

1. Client sends a write to the **Primary** pod.
2. Primary commits the change to its own transaction log and dedicated disk.
3. Primary streams that tiny log entry over the network to **Read Replicas**.
4. Replicas apply the log entry to their **own separate disks** and memory caches.

3\. Replication Modes & The "Zero Lag" Myth. True physical zero lag is impossible due to network transit and disk write times, managed via two modes:

| Feature            | Asynchronous _(Default)_                                                                     | Synchronous _(Near-Zero Lag)_                                                                        |
| ------------------ | -------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------- |
| **Mechanism**      | Primary commits locally, confirms "Success" immediately, and streams logs in the background. | Primary sends logs and **waits** for replica confirmation before confirming "Success" to the client. |
| **Lag**            | **Milliseconds** (typically 5ms–50ms).                                                       | **Effectively zero** for committed data.                                                             |
| **Write Speed**    | **Fastest** (no network wait).                                                               | **Slower** (waits for network round-trip).                                                           |
| **Data Loss Risk** | Recent unstreamed writes can be lost on primary crash.                                       | **Zero data loss** on failover.                                                                      |

4\. Handling Replication Lag in Production

* **Read-Your-Own-Writes:** Route a user's reads to the **Primary** for a few seconds immediately after they make an update, while routing general traffic to Replicas.
* **Lag-Aware Proxies:** Tools like **ProxySQL** or **PgBouncer/HAProxy** monitor replica lag and temporarily remove lagging replicas from read rotation.
* **Critical Read Routing:** Route financial/auth reads to Primary; route analytics/browsing reads to Replicas.

5\. Why StatefulSets Are Still Essential

* **Stable Network IDs:** Predictable DNS names (`db-0`, `db-1`) allow reliable peer discovery.
* **Sticky Storage:** Re-attaches the exact `PersistentVolumeClaim` if a pod reschedules to another node.
* **Ordered Operations:** Creates and updates pods sequentially (`0 -> 1 -> 2`) to prevent simultaneous downtime.

***

6\. Industry Standard: Managed Cloud vs. Kubernetes Operators

* **Default Standard (90% of companies):** Keep stateless apps in **Kubernetes** and stateful data outside in **Managed Cloud Services** (AWS RDS, Aurora, DynamoDB, GCP AlloyDB).
  * _Why:_ Zero Day-2 maintenance (automated patching, backups, failovers), smaller blast radius during K8s upgrades, and custom disaggregated storage (like Aurora) that avoids EBS single-AZ attachment limits.
* **When Databases Run&#x20;**_**Inside**_**&#x20;Kubernetes (10% of companies):** Managed via **Kubernetes Operators** (e.g., **CloudNativePG** for Postgres, **Vitess** for MySQL) instead of raw StatefulSets.
  * _Used for:_ Massive hyper-scale cost savings, strict multi-cloud/on-prem requirements, custom DB extensions, or ephemeral CI/CD test environments.

7\. AWS DevOps Production Blueprint (EKS + Managed DB)

* **Architecture & Networking:** Stateless apps run in **Amazon EKS** (Private Subnets); **RDS / Aurora / DynamoDB** live outside EKS in **Isolated Private Subnets**. Security Groups allow DB port traffic (`5432`/`3306`) strictly from EKS worker node Security Groups.
* **Secretless / Secure Connections:**
  * **IAM Database Auth (Gold Standard):** Use **EKS Pod Identity** so pods fetch short-lived (15-min) AWS STS tokens—zero static passwords.
  * **Secret Rotation:** Store credentials in **AWS Secrets Manager** and sync them into EKS via **External Secrets Operator (ESO)**.
* **Connection Pooling:** Place **Amazon RDS Proxy** between EKS and RDS/Aurora so rapid pod auto-scaling does not exhaust the database, `max_connections` and failovers happen up to 66% faster.
* **IaC & CI/CD Separation:** Provision VPC, EKS, and RDS with **Terraform / OpenTofu**; deploy apps via **GitOps (ArgoCD)**; run schema migrations via **Kubernetes `Jobs`** (Flyway/Liquibase) as ArgoCD pre-sync hooks.

#### Helm Package Manager

A Kubernetes cluster can consist of a large number of resources depending on applications, and creating and managing so many manifest files individually (including for separate envs) for these applications can be a daunting and inflexible task. Impacts Productivity.\
Hard to standardize the resources across envs and can easily miss best practices.

HELM. It is a package manager for Kubernetes. As a package manager, Helm serves a similar purpose to tools like **apt** or **npm**, allowing you to package, distribute, and install Kubernetes applications as a single unit.&#x20;

And like other package managers, it:\
• Supports single-command installation for relevant resources.\
• Solves relevant dependencies for the installation package<br>

Templating Engine and Lifecycle Management Capabilities: It helps manage the release lifecycle of Kubernetes applications. Helm provides powerful capabilities for deploying, upgrading, rolling back, and managing the entire lifecycle of your Kubernetes applications. It also works as a template engine: it automatically generates manifest files from a set of given templates.

Standardizing Kubernetes deployments. Improves productivity by reducing repetitive configuration tasks and minimizing deployment errors across environments.

**Helm Charts**:  Used to package application resources and dependencies together.&#x20;

Helm Chart Structure:\
chart-name/            # Helm Chart Name \
\-  chart.yaml           # Data and MetaData regarding the application, including dependencies\
\-  values.yaml         # Configuration values for parameters in Helm chart templates\
\-  templates/           # Templates used by Helm to generate manifests.\
\-  charts/                 # Dependent charts to be used in this current chart

A chart contains all the resource definitions needed to run an application, including templates, values, dependencies, and metadata. This bundling makes it possible to version, share, and deploy complex applications as a single unit

<pre class="language-bash" data-expandable="true"><code class="lang-bash"><strong># Important Helm Commands
</strong><strong>helm create &#x3C;helm-chart-name>
</strong>
helm template orders . -f values.yaml > orders.yaml

helm install orders . -f values.yaml

helm upgrade orders . -f values.yaml

</code></pre>

#### &#x20;Kubernetes probes - Startup, Liveness, and Readiness

Health Checks for Kubernetes Application

**Startup:** Has the application in the container started running?

**Liveness**: Automatically checks whether the container application is in a healthy and available state or not. If not, it will trigger the restart of the container to try to resolve the issue. \
This is different from Readiness Probes in that Liveness Probes focus on detecting and recovering from application failures rather than managing traffic flow.

**Readiness**: Determines when a container application is ready to accept requests. \
<mark style="color:red;">**IMPORTANT**</mark>: Incoming traffic **will not be forwarded** to the particular Pod until its container has passed all the readiness checks defined by the readiness probe. Readiness Probes serve as a traffic control mechanism - if a pod fails its readiness check, it's removed from the service's endpoint list, ensuring that no traffic is routed to pods that aren't ready to handle requests.

```yaml
livenessProbe:
  tcpSocket:
    port: {{ .Values.containerPort }}
  initialDelaySeconds: 15
  periodSeconds: 20 

readinessProbe:
  tcpSocket:
    port: 3003
  initialDelaySeconds: 5
  periodSeconds: 10
```



### Using GitHub Actions to Deploy to EKS (OIDC)

```yaml
name: 'Deploy Orders Helm Chart to EKS'

on:
  push:
    branches:
      - main
    paths:
      - 'helm/orders/**'
      - '.github/workflows/main.yaml'
  workflow_dispatch:
    inputs:
      image_tag:
        description: 'Docker image tag to deploy (defaults to commit SHA if empty)'
        required: false
        default: ''

permissions:
  id-token: write   # Required for requesting the AWS OIDC JWT token
  contents: read    # Required for actions/checkout

env:
  AWS_REGION: ${{ secrets.EKS_REGION }}
  EKS_CLUSTER: ${{ secrets.EKS_CLUSTER }}
  HELM_VERSION: v3.16.3

jobs:
  deploy:
    name: Deploy Orders Service
    runs-on: ubuntu-latest
    environment: production
    steps:
      - name: Checkout helm directory
        uses: actions/checkout@v4
        with:
          sparse-checkout: |
            helm/orders

      - name: Configure AWS credentials via OIDC
        uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: ${{ secrets.AWS_IAM_ROLE_ARN }}
          aws-region: ${{ env.AWS_REGION }}

      - name: Install Helm
        uses: azure/setup-helm@v4
        with:
          version: ${{ env.HELM_VERSION }}

      - name: Connect to EKS cluster
        run: aws eks update-kubeconfig --region "$AWS_REGION" --name "$EKS_CLUSTER"

      - name: Deploy Orders Helm chart to EKS
        env:
          IMAGE_TAG: ${{ github.event.inputs.image_tag || github.sha }}
        run: |
          cd helm/orders
          helm lint . -f values.yaml
          helm upgrade --install orders . \
            --namespace ecommerce \
            --create-namespace \
            --atomic \
            --timeout 5m \
            -f values.yaml \
            --set image.tag="$IMAGE_TAG"

      - name: Verify deployment
        run: kubectl rollout status deployment/orders -n ecommerce --timeout=120s
```



#### EKS Networking

Kubernetes Networking Model: **IP-per-pod model**\
Each Pod gets its own **Unique Cluste-wide Private IP Address**.And, \
Containers in that Pod share that same IP Address. uses **localhost** for internal communication with each other in case of multi container pod.\
Any Pod can talk to any other Pod across any node without Network Address Translation (NAT).

<div data-with-frame="true"><figure><img src=".gitbook/assets/image (42).png" alt=""><figcaption></figcaption></figure></div>

Although, Kubernetes defines the networking model, but does not include the networking implementataion by itself. Kubernetes does not ship with built-in code to create virtual cables, assign IPs, or route packets between servers. Without an external tool installed, Pods cannot communicate across nodes.\
It uses a **pluggable** model where the admin have the option to choose.   To bridge the gap, Kubernetes uses a standard interface called the **Container Network Interface (CNI)**. \
Because different organizations run Kubernetes in vastly different environments (bare-metal data centers, AWS, local laptops), a single networking setup wouldn't fit everyone. Cluster administrators can pick and plug in whichever CNI plugin best fits their infrastructure like Cilium, calico or CloudNative like AWS VPC CNI/AzureCNI



"**Pods are first-class citizens in a VPC network**": Traditionally, container platforms hid containers behind the host machine's IP address using port mapping and NAT—making containers invisible to the outside network.\
When a cloud provider's CNI plugin (like the AWS VPC CNI or Azure CNI) is used, **Pods receive real, native IP addresses directly from your cloud Virtual Private Cloud (VPC) subnet**.

* **No translation needed.** To the cloud network, a Pod looks just like a regular Virtual Machine (EC2 instance or VM).
* **Direct integration.** Cloud firewalls (Security Groups), VPC routing tables, and VPC Flow Logs can interact with and monitor individual Pods directly by their IP address,

**Being a "first-class citizen" means an object has its own direct identity and native privileges within a system**, rather than hiding behind a parent host.

**Why Do We Need This?**\
Making Pods first-class citizens solves four major infrastructure headaches:

* **Faster Network Speed:** Without NAT, the host server doesn't have to constantly rewrite packet headers or wrap traffic inside extra virtual tunnels (overlays). Packets travel directly across the cloud network at native hardware speeds.
* **Easy Troubleshooting:** When traffic leaves a Pod, your cloud network logs (like VPC Flow Logs) record the Pod's exact IP address.
* **Direct Security Rules:** You can attach cloud firewalls (like AWS Security Groups) directly to a specific Pod.

The <mark style="color:red;">**biggest catch is IP exhaustion**</mark>. Because every single Pod eats up a real IP address from your cloud subnet, spinning up thousands of Pods can quickly drain all available IPs in a small VPC.



A **CNI (Container Network Interface)** <mark style="color:red;">**plugin**</mark> is the networking equivalent of CSI—while CSI manages **storage** for pods, CNI manages **networking** (assigning IP addresses to pods and enabling pod-to-pod and pod-to-network communication).

**Amazon VPC CNI** (Default in EKS): \
Designed by AWS speciafically for AWS EKS. Comes pre-installed with EKS cluster. Runs as DaemonSet(aws-node), ie AWS EKS ships with VPC CNI plug-in by default.

* **How it works:** Assigns real **AWS VPC IP addresses** directly to Kubernetes pods using Elastic Network Interfaces (ENIs) attached to your EC2 worker nodes.
* **Pros:** Native AWS performance (no overlay network encapsulation overhead), direct integration with VPC Security Groups (Security Groups for Pods), VPC Flow Logs, and AWS load balancers.
* **Cons / Caveats:** Can exhaust VPC subnet IPv4 addresses quickly on large clusters (<mark style="color:red;">mitigated</mark> using **Prefix Delegation** or IPv6), and does **not natively enforce Kubernetes `NetworkPolicy`** unless enabled via its built-in eBPF network policy engine.

Popular Third-Party Alternatives CNI Plugins for EKS. You can run these either in **overlay mode** (completely replacing the AWS VPC CNI to **save VPC IPs**) or in **chaining mode** (keeping AWS VPC CNI for IP allocation while adding advanced policy/observability features):  **Calico**/\
**Cilium**: Uses **eBPF** instead of `iptables` for high-performance routing, L3–L7 Network Policies.



```shellscript
#VPC CNI runs as DaemonSets in EKS cluster
kubectl get ds -A                            # -A for all namespaces

kubectl get node
kubectl get pod

kubectl describe node
kubectl describe pod -o wide    

```



Ref:\
[https://github.com/aws/amazon-vpc-cni-k8s](https://github.com/aws/amazon-vpc-cni-k8s)\
[https://docs.aws.amazon.com/eks/latest/userguide/pod-networking-use-cases.html](https://docs.aws.amazon.com/eks/latest/userguide/pod-networking-use-cases.html)\
[https://docs.aws.amazon.com/eks/latest/userguide/cni-iam-role.html](https://docs.aws.amazon.com/eks/latest/userguide/cni-iam-role.html)<br>

## Service Discovery:

What is Service in K8s: Network Abstraction layer\
• Exposes application over the network by providing a stable endpoint to ephemeral app Pods.\
• Selects Pods using a selector \
• Does the load balancing&#x20;

<mark style="color:red;">**IMPORTANT**</mark>: Service CIDR Block is different and unique from Pods CIDR Block.

<div data-with-frame="true"><figure><img src=".gitbook/assets/image (43).png" alt=""><figcaption></figcaption></figure></div>

#### **CoreDNS:**

Native K8s component that helps in service discovery.&#x20;
