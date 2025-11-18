# Page10: Kubernetes Services

**Why** is it needed, and **What** are these?

**Why are services needed?**

1. How do pods communicate? Through IP or DNS name?
2. Pods are **ephemeral**. When they die, a new pod takes its place with a **new IP** address and new name. If a deployment controller manages them, each pod will have a **random hash** appended to the name of the pod
3. How do we rely on IP/DNS names of the pods when they change every time a pod dies and a new one replaces it with an entirely new identity?
4. How do we **load balance** the requests among various pod replicas? If there are 3 back-end pod replicas, how do we ensure that the requests are distributed to avoid overloading a specific pod?
5. How to **expose the pods outside the cluster** to make the application available for public use?

<div><figure><img src=".gitbook/assets/image (1).png" alt="" width="370"><figcaption></figcaption></figure> <figure><img src=".gitbook/assets/Screenshot-20250810153957-1372x663.png" alt="" width="375"><figcaption></figcaption></figure></div>

What: Services are not the running components, just like other objects, like pods. It is simply a configuration defined using template files.

* **Kubernetes Service** is an abstraction which defines a logical set of Pods running somewhere in your cluster, that all provide the same functionality
* Each Service is assigned a unique IP address at the time of creation.
* This address is tied to the lifespan of the Service, and will not change while the Service is alive.
* Pods can be configured to forward traffic to the Service, and the Service will automatically forward that traffic to any of its member pods in a load-balanced way
* Kubernetes Service provides the <mark style="color:$success;">IP Address,</mark> a single <mark style="color:$danger;">DNS name</mark>, and a <mark style="color:orange;">Load Balancer</mark> to a set of Pods
* <mark style="color:$info;">A Service identifies its member Pods with a</mark> <mark style="color:$info;"></mark><mark style="color:$info;">**selector**</mark><mark style="color:$info;">. For a Pod to be a member of the Service, the Pod must have all of the</mark> <mark style="color:$info;"></mark><mark style="color:$info;">**labels**</mark> <mark style="color:$info;"></mark><mark style="color:$info;">specified in the selector of the service</mark>
* A **label** is an arbitrary **key/value pair** that is attached to an object.
* Service automatically removes unhealthy pods from its load balancing.
*   **Services are namespaced objects in K8s:**&#x20;

    That means:

    * A Service exists **only within the namespace** it is created in.
    * If you create a Service called `my-service` in the `dev` namespace, it will not be visible in the `prod` namespace unless you explicitly expose it across namespaces.
    *   DNS names for services include the namespace:

        ```
        my-service.dev.svc.cluster.local
        ```

        Here:

        * `my-service` → Service name
        * `dev` → Namespace
        * `svc.cluster.local` → Kubernetes service domain

        <figure><img src=".gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

    **Implication:**

    * A Pod in one namespace can’t access a Service in another namespace by just using its short name (`my-service`) — it must use the full DNS name or be given special network rules.
    * This helps with isolation between environments (e.g., dev, staging, prod).



<div><figure><img src=".gitbook/assets/image (2).png" alt="" width="563"><figcaption></figcaption></figure> <figure><img src=".gitbook/assets/Screenshot-20250810154644-1074x424.png" alt=""><figcaption></figcaption></figure> <figure><img src=".gitbook/assets/Screenshot-20250810155210-1092x538.png" alt=""><figcaption></figcaption></figure></div>

Each of these Pods will have a label during its deployment. Service will use these labels to forward the traffic. So now, in case a new pod is being generated inside the same deployment, it will have a label attached to it, which will help the service to find these new pods and traffic the load to them.

**Service will have its unique IP Address.**  <mark style="background-color:$primary;">But PodA application talks to the service with the service name instead of its IP address</mark>, so even if we create a new service with the same name, it will get a new IP address, but it will work the same way as the old one if the configuration is are same. These service IP addresses are provided by the kube-api server
