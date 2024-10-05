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
#### etcd
#### kube-scheduler
#### kube-controller-manager
#### cloud-controller-manager

## Worker nodes components

#### kubelet
#### kube-proxy
#### Container runtime


## Summary
In this article, we discussed the architecture of Kubernetes.
Understanding the architecture and components of Kubernetes will be a good starting point for the Kubernetes journey.

## Reference
[1] https://kubernetes.io/docs/concepts/architecture
[2] https://www.redhat.com/en/topics/containers/kubernetes-architecture

