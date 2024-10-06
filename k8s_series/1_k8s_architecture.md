# Kubernetes Architecture

> cover image: Photo by <a href="https://unsplash.com/@ylearchives?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash">Yle Archives</a> on <a href="https://unsplash.com/photos/a-group-of-men-playing-musical-instruments-in-a-room-5SjR0aKp2rU?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash">Unsplash</a>

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
- There will be one or more worker nodes
- Components of the worker node:
  - kubelet
  - kube-proxy
  - Container runtime
- Each worker node consists of the above node components

## What is a pod?

- Understanding the Kubernetes pods will be more helpful for talking about Kubernetes architecture
- In Kubernetes, pods are the smallest deployable units of computing
- A pod can be one container or a group of containers

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

#### Container runtime

- Container runtime empowers Kubernetes to run containers effectively
- It manages the execution and lifecycle of containers
- Kubernetes supports container runtimes that are any implementation of [Kubernetes CRI (Container Runtime Interface)](https://kubernetes.io/docs/concepts/architecture/cri/)
- The following are some of the Container runtimes
  - [Docker](https://docs.docker.com/)
  - [Containerd](https://containerd.io/)
  - [CRI-O](https://cri-o.io/)

#### kubelet

- Kubelet is an application that communicates with the control plane
- It works as an agent of control plane
- It executes the decisions made by control plane
- It takes a set of PodSpecs and ensures that the containers described in the PodSpecs are running and healthy 

#### kube-proxy

- Kube-proxy is a network proxy for facilitating Kubernetes networking services
- It handles communication inside or outside the cluster
- If there is a network-plugin that has similar functionalities to the kube-proxy, kube-proxy won't be necessary

## Production Kubernetes Cluster

- Creating a production Kubernetes cluster is tedious if we're going to start from scratch
- We can use a deployment tool for creating a production Kubernetes cluster. The following are some of the deployment tools
  - [kubeadm](https://kubernetes.io/docs/reference/setup-tools/kubeadm)
  - [kops](https://kops.sigs.k8s.io)
- The easiest way to have a production grade Kubernetes cluster is using a managed Kubernetes service. The following are some of the managed Kubernetes services
  - [GCP](https://cloud.google.com/kubernetes-engine/?hl=en)
  - [EKS](https://aws.amazon.com/eks)
  - [AKS](https://azure.microsoft.com/en-us/products/kubernetes-service)

## My thoughts

![minions hurray](https://i.giphy.com/media/v1.Y2lkPTc5MGI3NjExN2sxNTAzenlqMWo1bWRyc2E0bGtjY2pjd3ZvaTIya3hjZHBtOTlxcyZlcD12MV9pbnRlcm5hbF9naWZfYnlfaWQmY3Q9Zw/2wGXK84nfEtR1JHe1H/giphy.gif)

Since I use AWS as my day-to-day cloud provider, I personally have used only EKS.
So, if you're planning to use AWS EKS, I highly recommend using [eksctl](https://eksctl.io).
I recommend reading 
["Comparing the Top Eight Managed Kubernetes Providers"](https://medium.com/@elliotgraebert/comparing-the-top-eight-managed-kubernetes-providers-2ae39662391b) 
medium article for having a comparison of managed Kubernetes services.

In my opinion, for a simple production system, Kubernetes is unnecessary.
But for a complex large-distributed system, Kubernetes is the best solution.

## Summary
In this article, we discussed the architecture of Kubernetes.
Understanding the architecture and components of Kubernetes will be a good starting point for the Kubernetes journey.

## Reference
- https://kubernetes.io/docs/concepts/architecture
- https://www.redhat.com/en/topics/containers/kubernetes-architecture
- https://kubernetes.io/docs/setup/production-environment

