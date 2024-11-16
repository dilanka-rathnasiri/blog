# Namespaces

## What is a Namespace?

* Namespace is a logically isolated group of resources with in a single cluster
* Names of resources in a namespace must be unique within the namespace
* But the same name can exist for resources in different namespaces
* Most Kubernetes resources are in some namespaces
  * eg: pods, services, deployments, etc.
* But some Kubernetes resources aren't in any namespace
  * eg: nodes, persistent volumes, namespace, etc.

## Initial Namespaces

Kubernetes starts with 4 initial namespaces

* default -> This is the default namespace
* kube-node-lease
* kube-public 
* kube-system -> Objects created by the Kubernetes system are in this namespace

* The best practice is to avoid using the default namespace
* Instead, create a new namespace and use it
* Also, we shouldn't create namespaces with prefix `kube-`
* Because it's reserved for Kubernetes system namespaces

## Create a Namespace with a manifest file

* Here's an example of a Namespace manifest file:

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: my-namespace
```

Then we can create this Namespace using [kubectl](https://kubernetes.io/docs/reference/kubectl) with one of the following commands
```shell
kubectl create -f my-namespace.yaml
```
or
```shell
kubectl apply -f my-namespace.yaml
```

* The above manifest file creates namespace `my-namespace`
* Also, we can create a namespace without a manifest file using the following kubectl command
```shell
kubectl create namespace my-namespace
```

We can delete the above namespace using kubectl with one of the following commands
```shell
kubectl delete -f my-namespace.yaml
```
or
```shell
kubectl delete namespace my-namespace
```

> [!WARNING] 
> Deleting a namespace deletes all the resources in the namespace 

## References

* https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/
* https://kubernetes.io/docs/tasks/administer-cluster/namespaces/