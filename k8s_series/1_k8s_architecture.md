# Kubernetes Cluster Architecture

> cover image: Photo by <a href="https://unsplash.com/@gwundrig?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash">Manuel Nägeli</a> on <a href="https://unsplash.com/photos/orchestra-playing-their-piece-7CcPLtywRso?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash">Unsplash</a>

[Kubernetes](https://kubernetes.io) is the most popular container orchestration platform. It's open-source, production-ready, and extensible. Kubernetes has emerged as the de facto standard for container orchestration, empowering organizations to scale and manage containerized applications effortlessly. Its open-source nature, coupled with its robust ecosystem, makes it a go-to platform for automating deployment, scaling, and operations in the cloud-native world. In this article, we will talk about the architecture of Kubernetes. Understanding the architecture and components of Kubernetes will be a good starting point for the Kubernetes journey.

## Kubernetes Cluster Architecture

![App Platorm](https://kubernetes.io/images/docs/kubernetes-cluster-architecture.svg)
*Figure 1: Basic Kubernetes cluster architecture; Image from <a href="https://kubernetes.io/docs/concepts/architecture/">Kubernetes docs</a>*

Figure 1 shows the architecture of a basic Kubernetes cluster. More components can be added to the cluster depending on the use cases.

There are 2 main parts in the Kubernetes cluster.
- Control plane
- Worker nodes

## Control plane
Control plane controls the cluster. It acts as the brain of the cluster. It makes decisions about the cluster. Also, It detects and responds to cluster events.

Components of the control plane:
- kube-apiserver
- etcd
- kube-scheduler
- kube-controller-manager
- cloud-controller-manager

## Worker nodes
Worker nodes do the actual workload of applications. They act as the muscles of the cluster. There will be one or more worker nodes.

Components of the worker node:
- kubelet
- kube-proxy
- Container runtime

Each worker node consists of the above node components.

## What is a pod?

Understanding the pods will be more helpful for talking about Kubernetes cluster architecture.

In Kubernetes, pods are the smallest deployable units of computing. A pod can be one container or a group of containers.

## Control plane components

### kube-apiserver

Kube-apiserver exposes Kubernetes APIs (Application Programming Interface) for interacting with the Kubernetes cluster. It acts as the front-end of the control plane. Also, kube-apiserver can be scaled horizontally.

We can access Kubernetes APIs via:
- REST API calls
- CLI tools such as [kubectl](https://kubernetes.io/docs/reference/kubectl/), [kubeadm](https://kubernetes.io/docs/reference/setup-tools/kubeadm/)


### etcd

Etcd is a key value store for all cluster data. It is an [etcd](https://etcd.io/) data store. So, It is highly available, reliable, and distributed.

Data stored in the etcd:
- Cluster configuration data
- Information about Kubernetes objects' states

### kube-scheduler

Kube-scheduler maintains the health of the cluster. It watches the pods. Also, it schedules pods to an appropriate node. Various factors are considered when scheduling. The following are some of them:
- individual and collective resource requirements
- hardware/software/policy constraints
- affinity and anti-affinity specifications

### kube-controller-manager

Kube-controller-manager consists of multiple controllers. These controllers make the decisions about the cluster and maintain the cluster. Logically, each controller can be considered as a separate process. To reduce the complexity, all the controller processes are compiled into a single binary and run as a single process. The following are some of them:
- Node Controller: Noticing and responding when nodes go down
- Job Controller: Watches the job objects and executes them
- EndpointSlice Controller: Populate EndpointSlice objects
- ServiceAccount Controller: Create default ServiceAccounts for new namespaces

We actually don't need to know in-depth about these controllers to use a Kubernetes cluster. However, in-depth knowledge will be valuable for Kubernetes administrators.

### cloud-controller-manager

Cloud-controller-manager contains cloud service provider's (e.g., AWS, GCP, etc.) specific controller logics. It links the cluster with the cloud provider's APIs.

Similar to the kube-controller-manager, cloud-controller-manager consists of multiple control logics compiled into a single binary and run as a single process.

If the Kubernetes cluster is on-premise or in a learning environment (e.g., Minikube, kind, etc.), there's no cloud-controller-manager.

## Worker nodes components

### Container Runtime

Container runtime empowers Kubernetes to run containers effectively. It manages the execution and lifecycle of containers.

Kubernetes supports container runtimes that are any implementation of [Kubernetes CRI (Container Runtime Interface)](https://kubernetes.io/docs/concepts/architecture/cri/). The following are some of the Container runtimes:
- [Docker](https://docs.docker.com/)
- [Containerd](https://containerd.io/)
- [CRI-O](https://cri-o.io/)

### kubelet

Kubelet is an application that communicates with the control plane. It acts as an agent of the control plane. It executes the decisions made by the control plane. It takes a set of PodSpecs and ensures that the containers described in the PodSpecs are running and healthy.

### kube-proxy

Kube-proxy is a network proxy for facilitating Kubernetes networking services. It handles communication inside or outside the cluster.

If there is a network plugin that has similar functionalities to the kube-proxy, kube-proxy won't be necessary.

## Production Kubernetes Cluster

Creating a production Kubernetes cluster from scratch is tedious. Also, I don't recommend it at all.

We can use a deployment tool to create a production Kubernetes cluster. The following are some of the deployment tools:
- [kubeadm](https://kubernetes.io/docs/reference/setup-tools/kubeadm)
- [kops](https://kops.sigs.k8s.io)

The easiest way to have a production-grade Kubernetes cluster is by using a managed Kubernetes service. The following are some of the managed Kubernetes services:
- [GCP](https://cloud.google.com/kubernetes-engine/?hl=en)
- [EKS](https://aws.amazon.com/eks)
- [AKS](https://azure.microsoft.com/en-us/products/kubernetes-service)

## My thoughts

![Baby yoda eating](https://i.giphy.com/media/v1.Y2lkPTc5MGI3NjExN2sxNTAzenlqMWo1bWRyc2E0bGtjY2pjd3ZvaTIya3hjZHBtOTlxcyZlcD12MV9pbnRlcm5hbF9naWZfYnlfaWQmY3Q9Zw/2wGXK84nfEtR1JHe1H/giphy.gif)

Since I use AWS as my day-to-day cloud provider, I personally have used EKS only. Honestly, my experience with EKS is awful. Although AWS EKS is a managed service, I'm encountering many issues with it. Also, managing AWS EKS cluster with IaC (Infrastructure as Code) such as Terraform or Pulumi is very challenging. The only reason I use EKS is that my current organization's whole system is in AWS.

> [!IMPORTANT]
> If you're planning to use AWS EKS, I highly recommend using [eksctl](https://eksctl.io).

But keep in mind as of 6th October 2024, eksctl doesn't have a stable major release yet. Its latest version is still 0.191.0.

I recommend reading [this medium article](https://medium.com/@elliotgraebert/comparing-the-top-eight-managed-kubernetes-providers-2ae39662391b) for having a comparison of managed Kubernetes services.

In my opinion, for a simple production system, Kubernetes is unnecessary. But for a complex large-distributed system, Kubernetes is the best solution.

## Summary
In this article, we explored the architecture of Kubernetes. Understanding the architecture and components of Kubernetes will be valuable for a solid foundation of Kubernetes. Also, I encourage you to get hands-on experience with Kubernetes. [MiniKube](https://minikube.sigs.k8s.io/docs) will be a good starting point for that.

## References
- https://kubernetes.io/docs/concepts/architecture
- https://www.redhat.com/en/topics/containers/kubernetes-architecture
- https://kubernetes.io/docs/setup/production-environment
