# Page11: Ingress Controller

What is reverse Proxy: Nginx, Apache, HAProxy

Ingress Controller vs Ingress:\
Ingress is the set of rules, and the Ingress Controller is the program that enforces those rules.

The Ingress Controller is the actual software that runs inside your Kubernetes cluster. It's a specialized load balancer and reverse proxy that continuously watches for newly created or updated Ingress resources. When it detects a new or changed Ingress, it reads the rules defined in the Ingress object and configures itself to implement those rules. Without an Ingress Controller, an Ingress resource is useless. Common Ingress Controllers include **NGINX, HAProxy, and Traefik.**

#### Ingress

An Ingress is a Kubernetes API object that defines the rules for routing external HTTP and HTTPS traffic to services within the cluster. It's a declarative way to tell Kubernetes how to handle incoming requests, such as:

* Which hostname (e.g., `www.example.com`) should be directed to which service?
* Which URL path (e.g., `/api` or `/blog`) should be sent to a specific backend service.
* Handling SSL/TLS termination for secure connections.
* Providing load balancing across multiple services.

You create an Ingress using a YAML file, similar to other Kubernetes objects like Pods or Services. It is a set of specifications, but it doesn't have any functionality on its own.







Ingress Controller vs Reverse Proxy in K8 Cluster Environment.&#x20;

**Ingress Controller**

* **K**ubernetes-native component, **Ingress** is probably the most powerful way to expose the services outside the cluster
* It exposes multiple services under the same IP address
* Note, an **Ingress Controller** typically <mark style="color:$danger;">**doesn’t**</mark> eliminate the need for an external load balancer — the ingress controller adds a layer of routing and control behind the load balancer
* You only pay for one load balancer IP if you are using any cloud native load balancer, and Ingress is smart enough to route various requests using simple **Host** or **URL-based** routing
* Both **app1.com** and **app2.com** point to the same load balancer IP. The Ingress controller routes the traffic from **app1.com** to its corresponding service inside the cluster. The same is true for **app2.com**. This smart routing is made possible through **Ingress rules**
* This way, we have avoided having multiple IPs for each exposed service.

<mark style="color:$danger;">**Note:**</mark> <mark style="color:$danger;"></mark><mark style="color:$danger;">We can map more than one domain to one IP address of the Load Balancer.</mark>

### Multiple Domains to One IP

When you map multiple domains to the same load balancer IP, the load balancer uses **Host header inspection** to determine which backend service to route the request to. For example:

* `api.example.com` → Backend API servers
* `www.example.com` → Web frontend servers
* `admin.example.com` → Admin panel servers

All these domains can point to the same load balancer IP (e.g., `203.0.113.10`), and the load balancer examines the HTTP Host header to make routing decisions.

>
>
> **Cloud Load Balancers:**
>
> * **Application Load Balancers (ALB)** in AWS typically have multiple IP addresses behind a single DNS name for redundancy
> * **Network Load Balancers (NLB)** can have one static IP per availability zone
> * **Google Cloud Load Balancer** can have multiple IPs depending on the type (global vs regional)
>
> **Physical/On-premises Load Balancers:**
>
> * Usually configured with at least 2 IPs (primary + secondary for failover)
>
> Even if your load balancer has multiple IPs internally, you typically expose one primary IP or use DNS-based routing to present a single entry point for your domains.

<figure><img src=".gitbook/assets/image (24).png" alt="" width="271"><figcaption></figcaption></figure>

Ingress Controller Flow:

* **User** accessing websites → `www.app1.com` and `www.app2.com`
* Both go through a **Cloud LB (Load Balancer)**
* LB forwards traffic to **192.168.0.100** (cluster entry IP)
* Inside cluster → **Ingress** routes traffic:
  * `app1.com` → **Service-1** → **Pod-1** (`10.248.0.22:5000`)
  * `app2.com` → **Service-2** → **Pod-2** (`10.248.0.23:3000`)

**Ingress Controller**

* Ingress controllers don’t come with the standard Kubernetes binary; they must be deployed separately.
* They are generally implemented by a third-party **proxy** that can read the **Ingress rules** and adjust its configuration accordingly.
* There are many types of Ingress controllers like **AWS ALB Controller, Traefik, Nginx, HAProxy, Contour, Istio,** etc.
* There are also plugins for Ingress controllers, like **cert-manager**, that can automatically provision SSL certificates for the services.
* Ingress controllers also provide features such as **SSL** and **Auth** straight out of the box.
* If you are running the cluster on-prem, Ingress controllers are to be exposed via a **NodePort** and use a **proxy** between the DNS server and the Ingress controller, unless we have other solutions like **MetalLB**.

An ingress controller is a **Kubernetes wrapper** around a reverse proxy that makes it speak "Kubernetes language" and automatically configures the underlying proxy based on Kubernetes resources.

> Most ingress controllers actually **use a reverse proxy as their data plane**:
>
> * **Nginx Ingress Controller** → Uses Nginx as reverse proxy
> * **Traefik Ingress Controller** → Uses Traefik as reverse proxy
> * **HAProxy Ingress Controller** → Uses HAProxy as reverse proxy
> * **Istio Gateway** → Uses Envoy proxy
>
> ### Example Architecture
>
> ```
> Internet → LoadBalancer → Ingress Controller (Nginx) → Services → Pods
>                               ↓
>                          Reverse Proxy
>                          (Nginx process)
> ```

>
>
> ### Configuration Comparison
>
> **Traditional Reverse Proxy (Nginx):**
>
> nginx
>
> ```nginx
> upstream backend {
>     server 10.0.1.10:8080;
>     server 10.0.1.11:8080;
> }
>
> server {
>     location /api {
>         proxy_pass http://backend;
>     }
> }
> ```
>
> **Ingress Controller (K8s):**
>
> yaml
>
> ```yaml
> apiVersion: networking.k8s.io/v1
> kind: Ingress
> metadata:
>   name: api-ingress
> spec:
>   rules:
>   - host: api.example.com
>     http:
>       paths:
>       - path: /api
>         backend:
>           service:
>             name: api-service
>             port:
>               number: 8080
> ```

\ <mark style="color:$danger;">**Most Important Difference:**</mark> \
**Ingress Controllers do automatic service discovery.**\
[**https://claude.ai/chat/2b45dedd-564a-4e0a-a0d5-e155416342fb#:\~:text=How%20Ingress%20Controllers%20Discover%20Services**](https://claude.ai/chat/2b45dedd-564a-4e0a-a0d5-e155416342fb)

> #### Service Discovery vs Traditional Load Balancers
>
> **Traditional Load Balancer:**
>
> nginx
>
> ```nginx
> # Manual configuration - static
> upstream backend {
>     server 10.0.1.10:8080;
>     server 10.0.1.11:8080;
>     # Must manually add/remove servers
> }
> ```
>
> **Kubernetes Ingress Controller:**
>
> * Automatically discovers all pods behind a service
> * Updates when pods are added/removed
> * Handles rolling deployments seamlessly
> * Respects pod readiness probes

> #### Real-World Example Flow
>
> yaml
>
> ```yaml
> # 1. Service definition
> apiVersion: v1
> kind: Service
> metadata:
>   name: web-service
> spec:
>   selector:
>     app: web
>   ports:
>   - port: 80
>     targetPort: 8080
>
> ---
> # 2. Ingress points to service
> apiVersion: networking.k8s.io/v1
> kind: Ingress
> metadata:
>   name: web-ingress
> spec:
>   rules:
>   - host: web.example.com
>     http:
>       paths:
>       - path: /
>         backend:
>           service:
>             name: web-service
>             port: 
>               number: 80
> ```

**What the ingress controller does:**

1. Sees the Ingress resource
2. Queries `web-service` → finds ClusterIP `10.96.0.100:80`
3. Queries endpoints for `web-service` → finds pod IPs `192.168.1.10:8080`, `192.168.1.11:8080`
4. Configures a reverse proxy to route `web.example.com` → those pod IPs
5. Continuously watches for pod changes

#### Dynamic Updates

When pods scale or change:

```bash
bash

# Scale up
kubectl scale deployment web --replicas=5
```

**Ingress controller automatically:**

1. Detects new endpoints via API watch
2. Updates reverse proxy config
3. Reloads/reconfigures without dropping connections









References:

[https://kubernetes.io/docs/concepts/services-networking/ingress/](https://kubernetes.io/docs/concepts/services-networking/ingress/)\
[https://kubernetes.io/docs/concepts/services-networking/ingress-controllers/](https://kubernetes.io/docs/concepts/services-networking/ingress-controllers/)\
[https://onairotich.medium.com/services-and-ingress-7afc517f2ec6](https://onairotich.medium.com/services-and-ingress-7afc517f2ec6)\
[https://kubernetes.github.io/ingress-nginx/user-guide/basic-usage/](https://kubernetes.github.io/ingress-nginx/user-guide/basic-usage/)\
[https://engineering.clearroute.io/complete-guide-using-kubernetes-secrets-store-csi-driver-with-hashicorp-vault-1a6d104e9e5b](https://engineering.clearroute.io/complete-guide-using-kubernetes-secrets-store-csi-driver-with-hashicorp-vault-1a6d104e9e5b)\
[https://www.ashokit.in/assets/upload-notes/01-Docker-K8S/Docker-Notes.txt](https://www.ashokit.in/assets/upload-notes/01-Docker-K8S/Docker-Notes.txt)\
[https://www.interviewbit.com/docker-interview-questions/#docker-namespace](https://www.interviewbit.com/docker-interview-questions/#docker-namespace)\
[https://medium.com/@techsuneel99/docker-from-beginner-to-expert-a-comprehensive-tutorial-5efec10c82ab](https://medium.com/@techsuneel99/docker-from-beginner-to-expert-a-comprehensive-tutorial-5efec10c82ab)











