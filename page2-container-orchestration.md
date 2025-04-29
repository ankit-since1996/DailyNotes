# Page2: Container Orchestration

### **What is Orchestration**

Orchestration in **technical** terms refers to the automated coordination and management of complex computer systems, services, and middleware. It involves aligning and managing multiple computerized tasks and workflows to ensure systems operate smoothly, efficiently, and as intended. The coordination and management of multiple computer systems, applications, and/or services, stringing together multiple tasks to execute a larger workflow or process

This process can include provisioning resources, managing dependencies, and facilitating communication between disparate systems, often leveraging tools to streamline and optimize these processes.



### What is Container Orchestration

Container orchestration is the automated <mark style="color:red;">**process of managing the lifecycle, deployment, scaling, networking, and availability of containerized applications across cluster**</mark>s.&#x20;

It ensures that containerized applications run smoothly by controlling and automating tasks like resource allocation, load balancing, and service discovery. Tools like **Kubernetes**, **Docker Swarm**, and **Apache Mesos** are commonly used for container orchestration.

```
Container orchestration is used to automate the following tasks at scale
```

{% code overflow="wrap" lineNumbers="true" fullWidth="false" %}
```
 Configuring and scheduling containers using schedulers
 Provisioning and deployment of containers
 Redundancy and availability of containers: Desired vs Current state
 Scaling up or removing containers to spread application load evenly across the host infrastructure
 Movement of containers from one host to another if there is a shortage of resources in a host, or if a host dies
 Allocation of resources between containers
 External exposure of services running in a container to the outside world
 Load balancing of service discovery between containers
 Health monitoring of containers and hosts
```
{% endcode %}

## References:

Kubernetes Overview:

[https://kubernetes.io/docs/concepts/overview/](https://kubernetes.io/docs/concepts/overview/)



