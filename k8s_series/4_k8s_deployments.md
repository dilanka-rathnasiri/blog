# Kubernetes Deployments

## What is a Kubernetes Deployment?

* A Deployment manages a stateless set of pods to run an application workload
* We can consider deployment as an abstraction layer for managing a set of pods
* We describe the desired state in a Deployment, then the Deployment Controller changes the actual state to the desired state at a controlled rate
* So, we indirectly manage ReplicaSets and Pods through the Deployments

## Creating a Deployment with a Manifest File

* Here’s an example of a Deployment manifest file:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-deployment
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
        - name: nginx
          image: nginx
          ports:
            - containerPort: 80
```

Then we can create this ReplicaSet using [kubectl](https://kubernetes.io/docs/reference/kubectl) with one of the following commands
```shell
kubectl create -f my-deployment.yaml
```
or
```shell
kubectl apply -f my-deployment.yaml
```

* The above manifest file creates Deployment `my-deployment`
* Let's discuss important fields in the above manifest file

## References

- https://kubernetes.io/docs/concepts/workloads/controllers/deployment/