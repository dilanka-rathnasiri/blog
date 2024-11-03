# Kubernetes Pods

## Introduction

## What are Kubernetes Objects?

- Kubernetes Objects are entities of the Kubernetes System
- We can consider Kubernetes objects as the build blocks of a Kubernetes cluster
- Each Kubernetes object has its own specific attributes and responsibilities
- There are many Kubernetes objects

## What is a Pod?

- In Kubernetes, pods are the smallest deployable units of computing
- Kubernetes pods are one object of the Kubernetes objects
- A pod can be one container or a group of containers
- So, pods can be considered as the abstraction to represent one container or a group of containers
- In most use cases, pods have single containers

## Manifest file for Kubernetes pods

Example pod manifest file:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: nginx:latest
    ports:
    - containerPort: 80
```

Then we can create a pod using [kubectl](https://kubernetes.io/docs/reference/kubectl) with the following command
```shell
kubectl apply -f nginx.yaml
```

- The above manifest file creates a pod with a single nginx container
- In most use cases, we don't need to directly create a pod using a manifest file
- We will use [workload resources](https://kubernetes.io/docs/concepts/workloads) for creating pods

## Multi-container pods

- Container required to work together can be encapsulated in to a single pod
- These containers are automatically co-located and co-located and co-scheduled in the same physical or virtual machine
- These containers can share resources and dependencies
- These containers can communicate and coordinate with each other
- In most cases, we use pods with single containers

Example multi-container pod manifest file:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: nginx:latest
    ports:
    - containerPort: 80
  - name: redis
    image: redis
    ports:
    - containerPort: 6379
```
- There are several types of multi-container pods that we need to discuss
  - Sidecar Containers
  - Ambassador Containers
  - Adapter Containers

## Sidecar Containers

- Sidecar containers are the secondary containers that run along with the main application container within the same Pod
- They provide additional services or functionalities such as logging, monitoring

## Init Container

- Init container is a container that runs before the app container of the pod
- We can use init containers for tasks required to execute once at the pod startup
- Init containers are also regular containers
- Init containers always run to completion
- Init containers run only at the pod startup
- Pod can have multiple init containers

Example Kubernetes manifest file with 2 init containers
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app.kubernetes.io/name: MyApp
spec:
  containers:
  - name: myapp-container
    image: busybox:1.28
    command: ['sh', '-c', 'echo The app is running! && sleep 3600']
  initContainers:
  - name: init-myservice
    image: busybox:1.28
    command: ['sh', '-c', "until nslookup myservice.$(cat /var/run/secrets/kubernetes.io/serviceaccount/namespace).svc.cluster.local; do echo waiting for myservice; sleep 2; done"]
  - name: init-mydb
    image: busybox:1.28
    command: ['sh', '-c', "until nslookup mydb.$(cat /var/run/secrets/kubernetes.io/serviceaccount/namespace).svc.cluster.local; do echo waiting for mydb; sleep 2; done"]
```

- Init containers are defined inside `initContainers` section
- In the above example init-myservice and init-mydb are init containers
- Init container run sequentially in the order of `initContainers` section of the manifest file
- Only one init container runs at a time
- So, `init-myservice` container starts first and runs to the completion
- After successfully completing `init-myservice` init container, `init-mydb` init container starts and runs to the completion
- After successfully completing `init-mydb` init container, `myapp-container` container starts
- If an init container fails, it will be restarted until succeeded
- However, we can control the restart behavior by using `restartPolicy` of the pod
- If an init container fails, the whole pod will fail

## Summary

## References

1. https://kubernetes.io/docs/concepts/workloads/pods/
2. https://kubernetes.io/docs/concepts/overview/working-with-objects/
3. https://kubernetes.io/docs/concepts/workloads/pods/sidecar-containers/
4. https://kubernetes.io/docs/concepts/workloads/pods/init-containers/
