# ConfigMaps

## What Is a ConfigMap?

* A ConfigMap is an object to store non-confidential configurations in key-value pairs format for other objects' usage
* So, configMap:
    * a key-value pairs storage
    * for non-confidential data
    * Other kubernetes objects use the data stored in the ConfigMap
* A ConfigMap allows us to separate configuration data from the application code
* The data stored in a ConfigMap can't exceed 1 MiB

## Creating a ConfigMap With a Manifest File

* To understand how ConfigMaps work, let’s start with the manifest file
* Here’s an example of a ConfigMap manifest file:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: car-configmap
data:
  # property-like keys: each key maps to a simple value
  porsche: "deutschland"
  bmw: "deutschland"
  toyota: "japan"

  # file-like keys
  car.brands: |
    ford=usa
    chevy=usa
    audi=deutschland
    pagani=italy
    lamborghini=italy
binaryData:
  car-bin-data: bGV4dXMsIHRveW90YSwgYm13LCBhdWRpLCBiZW56
```

We can create this ConfigMap using [kubectl](https://kubernetes.io/docs/reference/kubectl) with one of the following commands
```shell
kubectl create -f car-configmap.yaml
```
or
```shell
kubectl apply -f car-configmap.yaml
```

The above manifest file creates ConfigMap `car-configmap`

## Key Fields in the ConfigMap Manifest File

* `data`:
  * Contains key-value pairs
  * Values can be,
    * property-like keys
    * file-like keys
  * Contains UTF-8 strings
* `binaryData`:
  * Contains key-value pairs
  * Contains binary data as base64 encoded strings

* Both `data` and `binaryData` fields are optional
* One ConfigMap can contain both `data` and `binaryData` fields
* Same key can't be in both `data` and `binaryData` fields
* Also, same key can't duplicate in one of `data` or `binaryData` fields according to the YAML specification
* Keys of key-values pairs under the `data` and `binaryData` can only contain alphanumeric characters, `-`, `_`, `.`

## View values in a ConfigMap with Kubectl

* Using `kubectl describe`

Command:
```shell
kubectl describe configmaps car-configmap
```

Output:
```text
Name:         car-configmap
Namespace:    default
Labels:       <none>
Annotations:  <none>

Data
====
toyota:
----
japan
bmw:
----
deutschland
car.brands:
----
ford=usa
chevy=usa
audi=deutschland
pagani=italy
lamborghini=italy

porsche:
----
deutschland

BinaryData
====
car-bin-data: 30 bytes

Events:  <none>
```

* Using `kubectl get`

Command:
```shell
kubectl get configmaps car-configmap -o yaml
```

Output:
```text
apiVersion: v1
binaryData:
  car-bin-data: bGV4dXMsIHRveW90YSwgYm13LCBhdWRpLCBiZW56
data:
  bmw: deutschland
  car.brands: |
    ford=usa
    chevy=usa
    audi=deutschland
    pagani=italy
    lamborghini=italy
  porsche: deutschland
  toyota: japan
kind: ConfigMap
metadata:
  creationTimestamp: "2024-11-24T15:03:33Z"
  name: car-configmap
  namespace: default
  resourceVersion: "514"
  uid: bdeecd21-5599-4c99-97c0-4b539158ad3b
```

## How to use a ConfigMap?

4 ways to use data inside a ConfigMap:
* As environmental variables
* Inside a container commands and arguments
* Mounting as volumes into pods
* Using Kubernetes API

## Using as environmental variables

e.g.:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: car-pod
spec:
  containers:
    - name: app
      image: busybox:latest
      command: ["/bin/sh", "-c", "printenv"]
      envFrom:
        - configMapRef:
            name: car-configmap
```

We can inspect the pod's logs using kubectl with the following command
```shell
kubectl logs car-pod
```

In the output log, we can see pod has used ConfigMap's data as environment variables

```text
....
car.brands=ford=usa
chevy=usa
audi=deutschland
pagani=italy
lamborghini=italy

porsche=deutschland
bmw=deutschland
toyota=japan
....
```

Also, we can select individual keys from a ConfigMap's data as follows.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: car-pod
spec:
  containers:
    - name: app
      image: busybox:latest
      command: ["/bin/sh", "-c", "printenv"]
      env:
        - name: porsche-taycan # here we can use a different key
          valueFrom:
            configMapKeyRef:
              name: car-configmap
              key: porsche
        - name: prius
          valueFrom:
            configMapKeyRef:
              name: car-configmap
              key: toyota

```

Pod's logs:

```text
....
prius=japan
porsche-taycan=deutschland
....
```

* Also, note that the pod's environmental variable key can be changed from the ConfigMap's data key

## Using inside container commands and arguments

e.g.:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: demo-pod
spec:
  containers:
    - name: app
      image: busybox:latest
      command: ["/bin/sh", "-c", "echo ${bmw}"]
      envFrom:
        - configMapRef:
            name: car-configmap
```

## Mounting as a volume into a pod

e.g.:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: car-pod
spec:
  containers:
    - name: app
      image: busybox:latest
      command: [ "ls", "/etc/app-config" ]
      volumeMounts:
        - name: config
          mountPath: "/etc/app-config"
          readOnly: true
  volumes:
    - name: config
      configMap:
        name: car-configmap
```

* In the above pod, 7 files are created for each key in `/etc/app-config`
* Each file for each key contains its value
* Also, we can specify the keys wanted to mount as follows

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: car-pod
spec:
  containers:
    - name: app
      image: busybox:latest
      command: [ "ls", "/etc/app-config" ]
      volumeMounts:
        - name: config
          mountPath: "/etc/app-config"
          readOnly: true
  volumes:
    - name: config
      configMap:
        name: car-configmap
        items:
          - key: porsche
            path: porsche911
          - key: bmw
            path: bmw
```

* In the above pod, 2 files are created in path `/etc/app-config/`
* Files are,
  * `porsche911` for `porsche` key
  * `bmw` for `bmw` key

## Optional ConfigMaps

e.g.:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: car-pod
spec:
  containers:
    - name: app
      image: busybox:latest
      command: ["/bin/sh", "-c", "printenv"]
      env:
        - name: FERARI
          valueFrom:
            configMapKeyRef:
              name: italy-configmap
              key: ferari
              optional: true # mark the variable as optional
```

* In the above pod, the variable `FERARI` will be empty
  * If the config map `italy-configmap` doesn't exist or
  * If the key `ferari` doesn't exist, 
* Otherwise, the variable `FERARI` will be set to the value of key `ferari`

## Deleting a ConfigMap

We can delete the ConfigMap using kubectl with one of the following commands
```shell
kubectl delete -f car-configmap.yaml
```
or
```shell
kubectl delete configmap car-configmap
```

## Immutable ConfigMaps

* We can mark a ConfigMap as immutable using the `immutable` field as follows

e.g.:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: car-configmap
data:
  porsche: "deutschland"
  bmw: "deutschland"
  toyota: "japan"
immutable: true # mark the ConfigMap as immutable
```

* So, we can't edit the ConfigMap `car-configmap` later
* Immutable ConfigMaps are useful for
  * Use the same set of configs for many apps
  * Prevent accidental modifications

## How does the updating ConfigMaps work?

* The worker node's kubelet worker process periodically checks for ConfigMaps changes
* When we update values in a ConfigMaps
  * ConfigMap's data used as environment variables or command line arguments in pods won't change
  * ConfigMap's data used as mounted volumes will eventually change
* Since the worker node skips checking for immutable ConfigMaps changes, immutable ConfigMaps have a performance advantage

## References

* https://kubernetes.io/docs/concepts/configuration/configmap
* https://spacelift.io/blog/kubernetes-configmap
* https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap