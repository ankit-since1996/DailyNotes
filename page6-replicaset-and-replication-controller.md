# Page6: ReplicaSet & Replication Controller

**In Kubernetes:**\
**The Desired State**

* The desired state is one of the core concepts of Kubernetes
* Through a declarative or an imperative API, we describe the state of the objects like Pod, Replicaset, Deployment, etc., in the cluster
* If, due to some failures, a **container stops running**, the <mark style="color:green;">Kubelet</mark> recreates the Pod based on the lines of the desired state stored in **etcd**
* **Kube controller managers** in the master are responsible for regulating the state of the system. If they **detect any drift** in the current state of the cluster, they **instruct the kubelets** component in the workers to spin up additional pods (depending on the desired state) to bring the cluster back to the desired state.
* So, Kubernetes strictly ensures that all the containers running across the cluster are always in the desired state

<figure><img src=".gitbook/assets/image (12).png" alt=""><figcaption></figcaption></figure>

Need for Replication:

* Scalability: Scale up and down based on load.
* Load balancing: Route traffic for better load balancing.
* Reliable: One goes down, others take care of the load in mean meantime.

<figure><img src=".gitbook/assets/image (13).png" alt=""><figcaption></figcaption></figure>

K8s allows us to automate deployments, scale, and manage containerized applications. Different ways to deploy our application(pods) on Kubernetes: The Latest two different resources that Kubernetes provides for deploying pods: **Deployments** and **StatefulSet**

**Replication Controller:** \
Legacy API for managing workloads that can scale horizontally. Superseded by the Deployment and ReplicaSet APIs.

**Note:**

A [`Deployment`](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/) that configures a [`ReplicaSet`](https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/) is now the recommended way to set up replication.

**Controller managers** are responsible for managing the desired state of the K8 cluster, so if you are running any resources in K8 without a Controller, and if those resources go down, K8 will not be able to adjust back to the desired state, as no controller is managing/monitoring this resource.

So, for managing the POD desired state, Kubernetes provides two controllers: **ReplicaSets**(set-based selectors) and **ReplicationController**(equity-based selector)

**IMP:**

1. ReplicationController(rc) manages the pods by replacing unhealthy pods and making sure the desired number of **pod replicas** are always running in the cluster.ie, it ensures that a specified number of pod replicas are running at any one time.
2. Also, ReplicationController <mark style="color:red;">manages the load between the pod replicas</mark> that it creates.
3.  These controllers use **Labels/Selectors** for them to know the specific pod that goes down, which is for a particular application/use case. That means the controller will get information via etcd that the pod is down, but when it has to launch the pod, it must know which type of pod has gone down, so that it can launch that specific type of pod.

    &#x20;             &#x20;

    <figure><img src=".gitbook/assets/image (14).png" alt="" width="563"><figcaption></figcaption></figure>

**Note\*: RC** helps even in the case of a single running pod application, by bringing a new pod in case your current pod goes down.&#x20;

Isolating Pod from ReplicationController **Using Labels:**

<figure><img src=".gitbook/assets/image (15).png" alt=""><figcaption></figcaption></figure>

**Scaling Replication Controller:**

```
Scale Up Replicas: using ad-hoc commands
    kubectl scale rc/rc_name --replicas=5
    
Using Manifest file: using manifest files.
    kubectl scale -f rc.yaml --replicas=5
    
Delete controller:
    kubectl delete rc rc_name 
    
```

<figure><img src=".gitbook/assets/image (7).png" alt=""><figcaption></figcaption></figure>

The replication controller deploys pod replicas that have specific labels attached to them. We can use those labels as selectors in the controller, therefore making this controller t**rack/monitor** all pods with these specific labels, which helps in pods' auto-healing.

A replication controller **does not support scheduling policies**, meaning you cannot provide rules for choosing cluster nodes to run pods from the managed set.



**Note: Situational Question:**

If we have let a controller create Pod replicas with some specific labels, and after that, we create some pods separately with those labels in them. What will happen? How replication controller behave in that case?

<mark style="color:red;">**IMP:**</mark>

1. Replication/Deployment Controller or ReplicaSet will have a **template** section in specs, where we define a template for the pod, which gets created by the controller. This template contains information like metadata, container specs, etc.

```yaml
// Some code
template:
    metadata:
      name: nginx  #create pods with the name nginx, can we have pods with same name?NO
      labels:
        app: nginx    #all three pods have the label "app:nginx" on them
    spec:
      containers:
      - name: nginx    
        image: nginx
        ports:
        - containerPort: 80
      - name: busybox
        image: busybox
```

2. Another important thing is the **selector,** which helps the controller to **watch/monitor** the specific pods in the cluster using labels. This selector will use labels defined on the labels of the pods.

```
// Some code
selector:
    app: nginx
```

{% code fullWidth="true" %}
```yaml
apiVersion: v1
kind: ReplicationController

metadata:
  name: myreplica-controller  #this goes into the pod's name.
  annotations:
    owner: anku
  labels:
    env: test
  namespace: default

spec:
  replicas: 3
  
  selector:
    app: nginx

  template:
    metadata:
      name: nginx
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
      - name: busybox
        image: busybox


## Commands ##
kubectl apply -f rc.yaml
kubectl delete -f rc.yaml

kubectl get rc
kubectl get pods

kubectl describe rc
kubectl describe pods
kubectl events

kubectl get pods --show-labels
kubectl get rc --show-labels  #if no label provided, RC will use pod label as the default label
```
{% endcode %}

\*: Create a separate pod with the same labels as the one mentioned in ReplicationController:

<div><figure><img src=".gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure> <figure><img src=".gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure></div>

This is what happens here: "**If there are too many pods, the ReplicationController terminates the extra pods**" ie, as we have already defined 3 at the max limit of pod replicas, so when we start a separate pod with same label as being used by controller selector, it will terminte that pod as the max replica condifition is already met.                                                                                                                                   &#x20;

&#x20;**\***&#x41;ll these points are valid only if we are in the **same namespace.** We can create pods with the same label in different namespaces, and they will get created.

<figure><img src=".gitbook/assets/image (6).png" alt=""><figcaption><p><br></p></figcaption></figure>

<mark style="color:red;">**VERY IMP:**</mark> The scale-up and down of pod replicas are random in the case of these three controllers. But StatefulSet Controllers scale up and down in an ordered manner. Pod scheduling order is being maintained&#x20;

**Stateless and Stateful:**

**Stateless** means there is no memory of the past. Every transaction is performed as if it were being done for the very first time.

**Stateful** means that there is memory of the past. Previous transactions are remembered and may affect the current transaction.

```
Stateless:
// The state is derived by what is passed into the function.

function int addOne(int number)
{
return number + 1;
}

Stateful:
// The state is maintained by the function

private int _number = 0; //initially zero
function int addOne()
{
_number++;
return _number;
}
```

[**https://www.baeldung.com/ops/kubernetes-deployment-vs-statefulsets**](https://www.baeldung.com/ops/kubernetes-deployment-vs-statefulsets)

Deployment vs StatefulSets:

[https://stackoverflow.com/questions/41583672/kubernetes-deployments-vs-statefulsets](https://stackoverflow.com/questions/41583672/kubernetes-deployments-vs-statefulsets)

[https://www.reddit.com/r/kubernetes/comments/gxto93/kubernetes\_statefulset\_simply\_explained/](https://www.reddit.com/r/kubernetes/comments/gxto93/kubernetes_statefulset_simply_explained/)

