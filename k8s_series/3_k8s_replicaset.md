# Kubernetes ReplicaSet

## Introduction

## What is a ReplicaSet?

* A ReplicaSet is a Kubernetes object that maintains a specified number of pods of a given template are running at any
  given time
* A ReplicaSet guarantees the availability of a specified number of identical pods
* We don't directly manage ReplicaSets
* Instead, we use [Deployments](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/) for managing ReplicaSet

## Creating a ReplicaSet with a Manifest File

* Understanding the manifest file of the ReplicaSet is helpful for understanding how ReplicaSet works
* Here’s an example of a ReplicaSet manifest file:

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: my-replicaset
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
        - name: my-container
          image: nginx
          ports:
            - containerPort: 80
```

Then we can create this ReplicaSet using [kubectl](https://kubernetes.io/docs/reference/kubectl) with one of the following commands
```shell
kubectl create -f my-replicaset.yaml
```
or
```shell
kubectl apply -f my-replicaset.yaml
```

* The above manifest file creates `my-replicaset` ReplicaSet
* Let's discuss important fields in the above manifest file

* `.spec.replicas`:
  * Number of pods managed by the ReplicaSet
  * In the above ReplicaSet, we have 3 pods
  * If we don't specify a value, then the default value is 1
* `.spec.template`:
  * Actual pods are created from this pod template
* `.spec.template.metadata.labels` =>
  * labels of the pods created from the pod template
  * These labels must match `.spec.selector.matchLabels` labels
  * In the above ReplicaSet, the labels of the pods are `app: my-app`
* `.spec.selector.matchLabels` =>
  * Replicaset identifies the pods required to manage by the ReplicaSet from these labels
  * If there are pods that aren't created by the above pod template and have all the labels matches with labels in `.spec.selector.matchLabels`, then those pods will be also managed by ReplicaSet `my-replicaset`
  * So, if we already have 2 pods with the labels `app: my-app`, then only 1 pod will be newly created by `my-replicaset` ReplicaSet
  * But all the 3 pods with the labels `app: my-app` will be managed by ReplicaSet `my-replicaset`
  * Also, we can use [label selectors](https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/) other than matchLabels such as `matchExpressions`
* A ReplicaSet is linked to its pods via the `metadata.ownerReferences` field of the pods
* Field `metadata.ownerReferences` specifies the owner of the object
* Following is the output YAML of a running pod which is managed by `my-replicaset` ReplicaSet

```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: "2024-11-10T07:30:35Z"
  generateName: my-replicaset-
  labels:
    app: my-app
  name: my-replicaset-dxzzg
  namespace: default
  ownerReferences:
    - apiVersion: apps/v1
      blockOwnerDeletion: true
      controller: true
      kind: ReplicaSet
      name: my-replicaset
      uid: a93e48a6-9ec6-46e6-b92f-c61ccd9a2d40
  resourceVersion: "472"
  uid: 6cf20e31-a73a-4846-ad8d-a4b41e7711ac
spec:
  containers:
    - image: nginx
      imagePullPolicy: Always
      name: my-container
      ports:
        - containerPort: 80
          protocol: TCP
      resources: {}
      terminationMessagePath: /dev/termination-log
      terminationMessagePolicy: File

```

We can delete the above ReplicaSet using kubectl with one of the following commands
```shell
kubectl delete -f my-replicaset.yaml
```
or
```shell
kubectl delete rs my-replicaset
```

## Replicaset vs. Replication Controller

* ReplicaSet is the next generation of Replication Controller
* So, we don't use Replication Controller anymore

## Summary

## References

- https://kubernetes.io/docs/concepts/workloads/controllers/replicaset
- https://kubernetes.io/docs/concepts/workloads/controllers/replicationcontroller