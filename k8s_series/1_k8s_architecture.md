# Kubernetes Architecture

[Kubernetes](https://kubernetes.io) is the most popular container orchestration platform.
It's opensource, production-ready, and extensible.
In this article, we will talk about the architecture of Kubernetes.
Understanding the architecture and components of Kubernetes will be a good starting point for the Kubernetes journey.

## Kubernetes Cluster Architecture

![App Platorm](https://kubernetes.io/images/docs/kubernetes-cluster-architecture.svg)
*Figure 1: Basic Kubernetes cluster architecture; Image from <a href="https://kubernetes.io/docs/concepts/architecture/">Kubernetes docs</a>*

Figure 1 shows the architecture of a basic Kubernetes cluster.
Depending on the use cases, more components can be added to the cluster.

There are 2 main parts in the Kubernetes cluster.
- Control plane
- Worker nodes

## Control plane
- Control plane controls the cluster
- We can consider the control plane as the brain of the cluster
- It makes decisions about the cluster
- It detects and responds to cluster events
- Components of the control plane:
  - kube-apiserver
  - etcd
  - kube-scheduler
  - kube-controller-manager
  - cloud-controller-manager

## Worker nodes
- Worker nodes do the actual workload of applications
- We can consider worker nodes as the muscles of the cluster
- Components of the worker node:
  - kubelet
  - kube-proxy
  - Container runtime

## Control plane components

#### kube-apiserver

- kube-apiserver exposes Kubernetes APIs (Application Programming Interface) for interacting with the Kubernetes cluster
- kube-apiserver is the front-end control plane
- We can access Kubernetes APIs via REST API calls, CLI tools such as [kubectl](https://kubernetes.io/docs/reference/kubectl/), [kubeadm](https://kubernetes.io/docs/reference/setup-tools/kubeadm/)
- kube-apiserver can be scaled horizontally

#### etcd

- Key value store for all cluster data
  - Cluster configuration data
  - Information about Kubernetes objects' states
- This is an [etcd](https://etcd.io/) data store
- Highly available, reliable, and distributed

#### kube-scheduler

- kube-scheduler maintains the health of the cluster
- It watches the pods in the cluster
- kube-scheduler schedule a pods to an appropriate node 
- Various factors are considered when scheduling. The following are some of them.
  - individual and collective resource requirements
  - hardware/software/policy constraints
  - affinity and anti-affinity specifications

#### kube-controller-manager

- kube-controller-manager consists of multiple controllers
- These controllers take the decisions about the cluster and maintain the cluster
- Logically, each controller can be considered as a separate process
- To reduce the complexity, all the controller processes are compiled into a single binary and run as a single process
- The following are some of them
  - Node Controller: Noticing and responding when nodes go down
  - Job Controller: Watches the job objects and execute them
  - EndpointSlice Controller: Populate EndpointSlice objects
  - ServiceAccount Controller: Create default ServiceAccounts for new namespaces
- We actually don't need to know about these controllers in-depth for maintaining a Kubernetes cluster. But in-depth knowledge will be valuable for Kubernetes administrators 

#### cloud-controller-manager

- cloud-controller-manager contains cloud service providers (e.g., AWS, GCP, etc.) specific controller logics
- It links the cluster with the cloud provider's APIs
- Similar to the kube-controller-manager, cloud-controller-manager consists of multiple control logics compiled to a single binary and run as a single process
- If the Kubernetes cluster is on-premise or in a learning environment (e.g., Minikube, kind, etc.), there won’t be a cloud-controller-manager

## Worker nodes components

#### kubelet
#### kube-proxy
#### Container runtime


## Summary
In this article, we discussed the architecture of Kubernetes.
Understanding the architecture and components of Kubernetes will be a good starting point for the Kubernetes journey.

## Reference
- https://kubernetes.io/docs/concepts/architecture
- https://www.redhat.com/en/topics/containers/kubernetes-architecture

