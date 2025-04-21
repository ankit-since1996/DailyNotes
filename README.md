---
description: Contents
---

# Page1: Introduction

<pre data-line-numbers><code> Container Orchestration
 Kubernetes Introduction
 Installation: Minikube &#x26; Kubeadm
 Kubernetes Architecture
 Understanding Kubeconfig file
 Pods vs Docker Containers
 Pods and Init Containers
 Commands and Arguments
 Multicontainer Patterns: Sidecar, Ambassador, Adapter
 Labels &#x26; Selectors
 Replica Sets &#x26; Deployments
 Deployment Patterns
 Namespaces
 Services: ClusterIP, NodePort, Load Balancer, Headless, Port forward
 Ingress Controllers and Resources: Path vs URL Based Routing
 Volumes: emptyDir, HostPath
 Persistent Volumes: Storage Classes, PV &#x26; PVC
 StatefulSets for Stateful Applications                 
 Manual Scheduling: Node/Pod Affinity, NodeName, and
 NodeSelector, Taints, and Tolerations
 Kubernetes Probes: Readiness, Liveness and Startup
 Resource Limits
 Static Pods &#x26; DaemonSets
 Jobs and Cronjobs
 Config Maps &#x26; Secrets
 RBAC: Authentication and Authorization
<strong> Cluster Maintenance and Upgrade                                       
</strong></code></pre>

> Q; What is POD? What is a container? Can we have more than one container in a Pod? In which cases can we use multi-container Pods?&#x20;
>
> As the pod is the smallest unit in the K8 cluster, we do not have a concept of deploying containers in a Cluster. It's always a Pod deployment in a Cluster. Inside that Pod, we can have containers, and one Pod can have multiple containers in it, depending on the use case.



*

