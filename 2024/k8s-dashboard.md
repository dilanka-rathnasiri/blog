## Deploying Kubernetes dashboard on a Kubernetes cluster

Kubernetes dashboard is a web user interface for getting overview of a kubernetes cluster.
It can be used for managing, monitoring, and troubleshooting a kubernetes cluster.
Kubernetes dashboard is one of the easiest way to have a dashboard for kubernetes cluster managing and monitoring.

## Install Kubernetes dashboard

Kubernetes dashboard can be install with [Helm](https://helm.sh).
Execute following commands in a terminal for installation.

```shell
# Add kubernetes-dashboard repository
helm repo add kubernetes-dashboard https://kubernetes.github.io/dashboard/
# Deploy a Helm Release named "kubernetes-dashboard" using the kubernetes-dashboard chart
helm upgrade --install kubernetes-dashboard kubernetes-dashboard/kubernetes-dashboard --create-namespace --namespace kubernetes-dashboard
```

## Create a user for accessing kubernetes dashboard

We need a user to access the kubernetes dashboard.
We can regulate the access control with Role-based access control (RBAC).
Reference [[5]](https://kubernetes.io/docs/reference/access-authn-authz/rbac/#role-and-clusterrole) provides a detailed explanation on RABC.

***Service Account:*** 
A service account provides a distinct identity in a Kubernetes cluster.
We can consider a service account is similar to a user.
But Service account and use account are two different things.
Reference [[6]](https://kubernetes.io/docs/concepts/security/service-accounts) provides a detailed explanation on service account.
We can use the following YAML configuration for creating a service account for kubernetes dashboard.

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: admin-user
  namespace: kubernetes-dashboard
```

***Cluster Role:*** 
Cluster role is a set of rules that provides specific permissions.
We only discuss four predefined roles for the simplicity.

They are,
- cluster-admin
  - Provides superuser access to do anything on any resource.
- admin
  - Provides permissions to do most of the actions on most of the resources.
  - It doesn't give the permissions for followings,
    - write access to the resource quota or to the namespace itself
    - write access to EndpointSlices (or Endpoints)
- edit
  - Provides permission to read and write access for most of the resources
  - It doesn't give the permissions for followings,
    - viewing or modifying roles or role bindings
    - permissions weren't provided by admin cluster role
- view
  - Provides read-only access for most of the resources
  - It doesn't give the permissions for followings,
    - viewing roles or role bindings
    - viewing Secrets
    - permissions weren't provided by view cluster role

Since great power comes with great responsibility, it is good to use minimum permission as possible.
![Uncle Ben advice](https://i.giphy.com/media/v1.Y2lkPTc5MGI3NjExdXByam4wdWpsZzFydnlkaXg1ejNsZHR0b2MzdTd2enViZnVoY2N2OCZlcD12MV9pbnRlcm5hbF9naWZfYnlfaWQmY3Q9Zw/10KIsXhwdoerHW/giphy.gif)

In this example kubernetes configuration yaml files, we use admin cluster role.

***Cluster Role Binding:*** 
We can use the following YAML configuration for creating a cluster role binding for the created user.

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: admin-user
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: admin-user
  namespace: kubernetes-dashboard
```

## Generating token for accessing Kubernetes dashboard

```shell
kubectl -n kubernetes-dashboard create token admin-user
```

## Create long-lived Bearer Token for accessing Kubernetes dashboard

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: admin-user
  namespace: kubernetes-dashboard
  annotations:
    kubernetes.io/service-account.name: "admin-user"   
type: kubernetes.io/service-account-token  
```

```shell
kubectl get secret admin-user -n kubernetes-dashboard -o jsonpath={".data.token"} | base64 -d
```

## Pulumi Infrastructure as Code for Kubernetes dashboard deployment

## Summary

## References
1. https://kubernetes.io/docs/tasks/access-application-cluster/web-ui-dashboard
2. https://github.com/kubernetes/dashboard
3. https://github.com/kubernetes/dashboard/blob/master/docs/user/accessing-dashboard/README.md
4. https://github.com/kubernetes/dashboard/blob/master/docs/user/access-control/creating-sample-user.md
5. https://kubernetes.io/docs/reference/access-authn-authz/rbac/#role-and-clusterrole
6. https://kubernetes.io/docs/concepts/security/service-accounts
