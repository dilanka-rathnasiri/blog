# ConfigMaps

## What Is a ConfigMap?

* A ConfigMap is an object to store non-confidential configurations in key-value pairs format for other objects usage
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
  name: game-configmap
data:
  # property-like keys; each key maps to a simple value
  player_initial_lives: "3"
  ui_properties_file_name: "user-interface.properties"

  # file-like keys
  game.properties: |
    enemy.types=aliens,monsters
    player.maximum-lives=5
  user-interface.properties: |
    color.good=purple
    color.bad=yellow
    allow.textmode=true
binaryData:
  file_template: RGVtbwo=
```

We can create this ConfigMap using [kubectl](https://kubernetes.io/docs/reference/kubectl) with one of the following commands
```shell
kubectl create -f game-configmap.yaml
```
or
```shell
kubectl apply -f game-configmap.yaml
```

The above manifest file creates ConfigMap `game-configmap`

## Key Fields in the ConfigMap Manifest File

* `data`:
  * contains key-value pairs
  * contains UTF-8 strings
* `binaryData`:
  * contains key-value pairs
  * contains binary data as base64 encoded strings

* Both `data` and `binaryData` fields are optional
* One ConfigMap can contain both `data` and `binaryData` fields
* Same key can't be included in both `data` and `binaryData` fields <!-- todo: check this in minikube -->
* Key value of key-values pairs under the `data` and `binaryData` can only contain alphanumeric characters, `-`, `_`, `.`


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
  name: demo-pod
spec:
  containers:
    - name: app
      command: ["/bin/sh", "-c", "printenv"]
      image: busybox:latest
      envFrom:
        - configMapRef:
            name: game-configmap
```

We can inspect the pod's logs using kubectl with the following command
```shell
kubectl logs demo-pod
```

In the output log we can see ConfigMap's data has been used as environment variables in the pod

<!-- todo: check the output, specially for binaryData -->
```shell
player_initial_lives=3
ui_properties_file_name=user-interface.properties
game.properties=enemy.types=aliens,monsters
player.maximum-lives=5
color.good=purple
color.bad=yellow
allow.textmode=true
```

Also, we can select individual keys from a ConfigMap's data as follows. Also, note that the key of the pod's environmental variable doesn't need to be the same as ConfigMap's key. We can use a different value for the key name.

```shell
apiVersion: v1
kind: Pod
metadata:
  name: demo-pod
spec:
  containers:
    - name: app
      command: ["/bin/sh", "-c", "printenv"]
      image: busybox:latest
      env:
        - name: initial_val
          valueFrom:
            configMapKeyRef:
              name: game-config
              key: player_initial_lives
```

Then Pod's log is as follows

```shell
initial_val=3
```

## Using inside a container commands and arguments

e.g.:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: demo-pod
spec:
  containers:
    - name: app
      command: ["/bin/sh", "-c", "echo ${player_initial_lives}"]
      image: busybox:latest
      envFrom:
        - configMapRef:
            name: game-configmap
```

## Mounting as volumes into pods

e.g.:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: demo-pod
spec:
  containers:
    - name: app
      command: ["ls", "/etc/app-config"]
      image: demo-app:latest
      volumeMounts:
        - name: config
          mountPath: "/etc/app-config"
          # readOnly: true todo: check this
  volumes:
    - name: config
      configMap:
        name: demo-config
```

* In the above pod, 7 files are created for each key in `/etc/app-config`
* Also, we can specify the keys wanted to mount as follows

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: demo-pod
spec:
  containers:
    - name: app
      command: ["ls", "/etc/app-config"]
      image: demo-app:latest
      volumeMounts:
        - name: config
          mountPath: "/etc/app-config"
          # readOnly: true todo: check this
  volumes:
    - name: config
      configMap:
        name: demo-config
        items:
          - key: player_initial_lives
            path: player_initial_lives
```

* In the above pod, 1 file is created for key `player_initial_lives` in path `/etc/app-config/player_initial_lives`

## Optional ConfigMaps

e.g.:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: dapi-test-pod
spec:
  containers:
    - name: test-container
      image: gcr.io/google_containers/busybox
      command: ["/bin/sh", "-c", "env"]
      env:
        - name: SPECIAL_LEVEL_KEY
          valueFrom:
            configMapKeyRef:
              name: a-config
              key: akey
              optional: true # mark the variable as optional
```

* In the above pod, if the config map `a-config` doesn't exist or key `akey` doesn't exist, the variable `SPECIAL_LEVEL_KEY` will be empty
* Otherwise, the variable `SPECIAL_LEVEL_KEY` will be set to the value of key `akey`

## Deleting a ConfigMap

We can delete above ConfigMap using kubectl with one of the following commands
```shell
kubectl delete -f game-configmap.yaml
```
or
```shell
kubectl delete configmap game-configmap
```

## Immutable ConfigMaps

* We can mark a ConfigMap as immutable using the `immutable` field as follows

e.g.:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: game-configmap
  labels:
    app: game
data:
  player_initial_lives: "3"
  ui_properties_file_name: "user-interface.properties"
  game.properties: "enemy.types=aliens,monsters"
  player.maximum-lives: "5"
  color.good: "purple"
  color.bad: "yellow"
  allow.textmode: "true"
immutable: true # mark the ConfigMap as immutable
```

* So, we can't edit the ConfigMap `game-configmap` later
* Immutable ConfigMaps are useful for
  * Use configs in many apps
  * Prevent accidental modifications

## How do the ConfigMaps updates work?

* Worker nodes' kubelet worker processes periodically check for ConfigMaps changes
* When we update values in a ConfigMaps
  * ConfigMap's data used as environment variables or command line arguments in pods won't change
  * ConfigMap's data used as mounted volumes will eventually change
* Since worker nodes' skip checking for immutable ConfigMaps changes, immutable ConfigMaps have a performance advantage

## References

* https://kubernetes.io/docs/concepts/configuration/configmap
* https://spacelift.io/blog/kubernetes-configmap
* https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap