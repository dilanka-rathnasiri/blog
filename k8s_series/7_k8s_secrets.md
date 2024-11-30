# Secrets

## What is a Kubernetes Secret?

* A Kubernetes secret is an object that contains sensitive data
* Secrets reduce the risk of exposing the sensitive data when deploying apps on Kubernetes
* Secrets are stored in Kubernetes etcd data store
* Secrets can't exceed 1 MB
* Secrets are encoded not encrypted

## Types of Secrets

* Opaque secrets
  * Store arbitrary user defined data
  * Default secret type
* Service account token secrets
  * Store token credentials that identify service accounts
* Docker config secrets
  * Store credentials for accessing container image registries
* Basic authentication secrets
  * Store credentials for basic authentication
  * Must contain at least one of the following keys
    * username
    * password
* SSH authentication secrets
  * Store SSH authentication data
* TLS secrets
  * Store TLS certificates and associated keys
* Bootstrap token secrets
  * Store bootstrap token data during the node bootstrap process
  * Typically,
    * In `kube-system` namespace
    * Name is in `bootstrap-token-{{token-id}}` format

## Base64 Encoding with the Linux terminal

```shell
echo "lamborghini" | base64
```

## Base64 Decoding with the Linux terminal

```shell
base64 -d <<< bGFtYm9yZ2hpbmkK
```

## Creating a Secret With a Manifest File

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: car-secret
type: Opaque
data:
  ford: bXVzdGFuZwo=
  nissan: c2t5bGluZQo=
stringData:
  dodge: "challenger"
  bugatti: "tourbillon"
```

* We can create the secret with kubectl using one of the following commands
```shell
kubectl create -f secret.yaml
```
or
```shell
kubectl apply -f secret.yaml
```

* The above manifest file creates secret `car-secret`

## Key Fields in the Secret Manifest File

* `type`
  * Type of the secret
  * Default is `Opaque`
* `data`
  * Contains key-value pairs
  * Contains base64 encoded strings
* `stringData`
  * Contains key-value pairs
  * Values are clear text strings

* Both `data` and `stringData` fields are optional
* One secret can contain both `data` and `stringData` fields
* Same key can't be in both `data` and `stringData` fields
  * If same key in both fields, then the `stringData` value will take precedence
* Also, same key can't duplicate in one of `data` or `stringData` fields according to the YAML specification
* Keys of key-values pairs under the `data` and `stringData` can only contain alphanumeric characters, `-`, `_`, `.`

## View values in a Secret with Kubectl

* We can't view the values in the secrets using `kubectl describe`

  Command :
```shell
kubectl describe secret car-secret
```

Output:
```text
Name:         car-secret
Namespace:    default
Labels:       <none>
Annotations:  <none>

Type:  Opaque

Data
====
bugatti:  10 bytes
dodge:    10 bytes
ford:     8 bytes
nissan:   8 bytes
```

* We can view the encoded values with the `kubectl get` command
```shell
kubectl get secrets car-secret -o jsonpath="{.data}"
```

Output:
```text
{"bugatti":"dG91cmJpbGxvbg==","dodge":"Y2hhbGxlbmdlcg==","ford":"bXVzdGFuZwo=","nissan":"c2t5bGluZQo="}
```

* We can use linux terminal `base64 -d` command to decode the above output
```shell
kubectl get secrets car-secret -o jsonpath="{.data.nissan}" | base64 -d
```

Output:
```text
skyline
```
## How to use a Secret?

* As environmental variables
* Mounting as volumes into pods

## Using as environmental variables

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
        - secretRef:
            name: car-secret
```

* We can inspect the pod's logs using kubectl with the following command
```shell
kubectl logs car-pod
```

* In the output log, we can see pod has used Secret's data as environment variables
* We can see, we get the decoded values of the secret data as environmental variables

```text
....
dodge=challenger
bugatti=tourbillon
ford=mustang
nissan=skyline
....
```

* Also, we can select individual keys from a Secret's data as follows.

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
        - name: NISSAN_GTR # here we can use a different key
          valueFrom:
            secretKeyRef:
              name: car-secret
              key: nissan
        - name: dodge
          valueFrom:
            secretKeyRef:
              name: car-secret
              key: dodge
```

* Pod's logs:

```text
....
dodge=challenger
NISSAN_GTR=skyline
....
```

* Also, note that the pod's environmental variable key can be changed from the Secret's data key

## Mounting as volumes into pods

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: car-pod
spec:
  containers:
    - name: app
      image: busybox:latest
      command: [ "ls", "/etc/app-secret" ]
      volumeMounts:
        - name: secret
          mountPath: "/etc/app-secret"
          readOnly: true
  volumes:
    - name: secret
      secret:
        secretName: car-secret
```

* In the above pod, 4 files are created for each key in `/etc/app-config`
* Each file for each key contains its decoded value

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
      command: [ "ls", "/etc/app-secret" ]
      volumeMounts:
        - name: secret
          mountPath: "/etc/app-secret"
          readOnly: true
  volumes:
    - name: secret
      secret:
        secretName: car-secret
        items:
          - key: ford
            path: mustang
          - key: nissan
            path: nissan
```

* In the above pod, 2 files are created in path `/etc/app-secret/`
* Files are,
  * `mustang` for `ford` key with content `mustang`
  * `nissan` for `nissan` key with content `nissan`

## Deleting a Secret

* We can delete the secret with kubectl using one of the following commands
```shell
kubectl delete -f secret.yaml
```
or
```shell

kubectl delete secret demo-secret
```

## Optional Secrets

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
            secretKeyRef:
              name: car-secret
              key: ferari
              optional: true # mark the variable as optional
```

* In the above pod, the variable `FERARI` won't exist if
  * The Secret `car-secret` doesn't exist or
  * The key `ferari` doesn't exist
* Otherwise, the variable `FERARI` will be set to the value of key `ferari`

## Immutable Secrets

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: car-secret
type: Opaque
data:
  ford: bXVzdGFuZwo=
  nissan: c2t5bGluZQo=
immutable: true # mark the Secret as immutable
```

* So, we can't edit the Secret `car-secret` later
* Immutable Secrets are useful for
  * Use the same set of secrets for many apps
  * Prevent accidental modifications

## References
- https://kubernetes.io/docs/concepts/configuration/secret
- https://spacelift.io/blog/kubernetes-secrets