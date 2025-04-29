# Page4: Kubernetes Objects

### Kubernetes Objects

Kubernetes objects are **persistent entities** that represent the state of your cluster.

In Kubernetes, anything a user creates and **retains** is referred to as an "**Object**." These can include various items such as namespaces, [pods](https://devopscube.com/kubernetes-pod/), [deployments](https://devopscube.com/kubernetes-deployment-tutorial/), [DaemonSet](https://devopscube.com/kubernetes-daemonset/), volumes, or secrets.

<figure><img src=".gitbook/assets/image (7).png" alt=""><figcaption></figcaption></figure>

The **etcd** component plays an important role in persisting cluster objects, amongst other things.  Kubernetes objects are stored persistently in the cluster's etcd database. They don't disappear when pods restart or when the control plane components are restarted.

Each Kubernetes object includes two important nested field components:

* **Spec**: You provide this when creating objects, describing your desired state.
* **Status**: Supplied by Kubernetes, describing the current state of the object.

A Kubernetes object is a "record of intent" -once you **create the object,** the Kubernetes system will constantly **work to ensure that the object exists**. By creating an object, you're effectively telling the Kubernetes system what you want your cluster's workload to look like; this is your <mark style="color:orange;">cluster's</mark> <mark style="color:orange;"></mark>_<mark style="color:orange;">desired state,</mark>_ and it maintains it through its various **controllers**.

This persistence has two important aspects:

1. **Data persistence**: Once you create a Kubernetes object (like a Pod, Deployment, or Service), its configuration and state are stored in the Kubernetes etcd database. This information persists even if the control plane components restart.
2. **Desired state persistence**: Kubernetes objects represent your desired state for the cluster. The system continually works to ensure the actual state matches this desired state. If something changes (like a pod crashes), Kubernetes will detect this difference and work to reconcile it.

Kubernetes Deployment object persists even if all its Pods crash or all nodes fail. When the system recovers, Kubernetes will recreate the necessary resources to match the Deployment's specification.

\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

To **work** with Kubernetes objects—whether to create, modify, or delete them—you'll need to use the [Kubernetes API](https://kubernetes.io/docs/concepts/overview/kubernetes-api/). To create these objects, you describe **what you want (desired state)** in a **file** using either **YAML or JSON**. It is called an Object Specification.

For creating an object, we must provide the object spec that describes its desired state, as well as some basic information about the object (such as a name)

### Kubernetes Object

Important native **Kubernetes object types** are organized in categories.

| Category                       | Kubernetes Objects                                                                                                                                                             |
| ------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **Workload**                   | <p>1. Pods<br>2. ReplicaSets<br>3. Deployments<br>4. StatefulSets<br>5. DaemonSets<br>6. Jobs<br>7. CronJobs<br>8. Horizontal Pod Autoscaler<br>9. Vertical Pod Autoscaler</p> |
| **Service & Networking**       | <p>1. Services<br>2. Ingress<br>3. IngressClasses<br>4. Network Policies<br>5. Endpoints<br>6. EndpointSlices</p>                                                              |
| **Storage**                    | <p>1. PersistentVolumes<br>2. PersistentVolumeClaims<br>3. StorageClasses</p>                                                                                                  |
| **Configuration & Management** | <p>1. ConfigMaps<br>2. Namespaces<br>3. ResourceQuotas<br>4. LimitRanges<br>5. Pod Disruption Budgets (PDB)<br>6. Pod Priority and Preemption<br></p>                          |
| **Security**                   | <p>1. Secrets<br>2. ServiceAccounts (sa)<br>3. Roles<br>4. RoleBindings<br>5. ClusterRoles<br>6. ClusterRoleBindings</p>                                                       |
| **Metadata**                   | <p>1. Labels and Selectors<br>2. Annotations<br>3. Finalizers</p>                                                                                                              |

### Common Object Parameters <a href="#common-object-parameters" id="common-object-parameters"></a>

Common parameters for every object.

| Parameter      | Description                                                                                                                                                                                                                      |
| -------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **apiVersion** | <p>The Kubernetes API version for the object.<br>- Alpha<br>- Beta<br>- Stable</p>                                                                                                                                               |
| **kind**       | <p>The type of object being defined<br>- Pod<br>- Deployment<br>- Service<br>- Configmap, etc.</p>                                                                                                                               |
| **metadata**   | <p>metadata is used to uniquely identify and describe a Kubernetes object. Here are some of the common key metadata that can be added to an object<br>- labels<br>- name<br>- namespace<br>- annotations<br>- finalizers etc</p> |
| **spec**       | Under the 'spec' section of a Kubernetes object definition, we declare the desired state and characteristics of the object that we want to create.                                                                               |

### Kubernetes Manifests <a href="#kubenetes-manifests" id="kubenetes-manifests"></a>

Normally, we call [Kubernetes YAML ](https://devopscube.com/create-kubernetes-yaml/)files manifests. It includes specifications for one or more Kubernetes objects, such as Deployments, Services, ConfigMaps, Secrets, etc.

`kubectl` interacts with the cluster by consuming these API endpoints.



### PODS:

### Namespaces:







```

The following command successfully displays all Kubernetes objects

kubectl api-resources
```

## References:

K8 Objects:

[https://devopscube.com/kubernetes-objects-resources/](https://devopscube.com/kubernetes-objects-resources/)

[https://kubernetes.io/docs/concepts/overview/working-with-objects/](https://kubernetes.io/docs/concepts/overview/working-with-objects/)

[https://kodekloud.com/blog/kubernetes-objects/](https://kodekloud.com/blog/kubernetes-objects/)

[https://stackoverflow.com/questions/52309496/difference-between-kubernetes-objects-and-resources](https://stackoverflow.com/questions/52309496/difference-between-kubernetes-objects-and-resources)



