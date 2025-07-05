# Page4: Minikube

**Minikube:**

A tool that makes it easy to run a l**ocal, single-node, complete Kubernetes cluster** on your machine

**Single Node Cluster**, ie, only has a master node, and we deploy our workload/application pods in that master node. Unlike kubeadmn, where the master node is tainted and we need to have separate worker nodes for the workload, here the **master node is untainted.**

&#x20;                                                 [**https://minikube.sigs.k8s.io/docs/**](https://minikube.sigs.k8s.io/docs/)

&#x20;                            [https://docs.aws.amazon.com/eks/latest/userguide/install-kubectl.html](https://docs.aws.amazon.com/eks/latest/userguide/install-kubectl.html)



### Kubectl



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

&#x20;                                                  ![](<.gitbook/assets/image (4) (1).png>)





### Kubeconfig File:
