## Deploying Kubernetes dashboard on a Kubernetes cluster

> cover image: Photo by <a href="https://unsplash.com/@jbcreate_?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash">Joseph Barrientos</a> on <a href="https://unsplash.com/photos/ship-helm-eUMEWE-7Ewg?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash">Unsplash</a>

In this article,
we will talk about deploying [Kubernetes dashboard](https://github.com/kubernetes/dashboard) on a Kubernetes cluster.
Kubernetes dashboard is the official web user interface for getting overview of a kubernetes cluster.
It can be used for managing, monitoring, and troubleshooting a kubernetes cluster.
Kubernetes dashboard is one of the easiest ways to have a dashboard for kubernetes cluster managing and monitoring.

## Install Kubernetes dashboard

Kubernetes dashboard can be easily installed with [Helm](https://helm.sh).
Execute the following commands in a terminal for installation.

```shell
# Add kubernetes-dashboard repository
helm repo add kubernetes-dashboard https://kubernetes.github.io/dashboard/
# Deploy a Helm Release named "kubernetes-dashboard" using the kubernetes-dashboard chart
helm upgrade --install kubernetes-dashboard kubernetes-dashboard/kubernetes-dashboard --create-namespace --namespace kubernetes-dashboard
```

We'll get a reply similar to the following.

***Reply-A:***
```shell
Release "kubernetes-dashboard" does not exist. Installing it now.
NAME: kubernetes-dashboard
LAST DEPLOYED: Fri Aug  9 20:35:55 2024
NAMESPACE: kubernetes-dashboard
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
*************************************************************************************************
*** PLEASE BE PATIENT: Kubernetes Dashboard may need a few minutes to get up and become ready ***
*************************************************************************************************

Congratulations! You have just installed Kubernetes Dashboard in your cluster.

To access Dashboard run:
  kubectl -n kubernetes-dashboard port-forward svc/kubernetes-dashboard-kong-proxy 8443:443

NOTE: In case port-forward command does not work, make sure that kong service name is correct.
      Check the services in Kubernetes Dashboard namespace using:
        kubectl -n kubernetes-dashboard get svc

Dashboard will be available at:
  https://localhost:8443 # ‼ note down this url for accessing the dashboard
```

## Create a user for accessing kubernetes dashboard

We need a user to access the kubernetes dashboard.
We can regulate access control with Role-based access control (RBAC).
Reference [[3]](https://kubernetes.io/docs/reference/access-authn-authz/rbac/#role-and-clusterrole) provides a detailed explanation on RBAC.

***Service Account:*** 
A service account provides a distinct identity in a Kubernetes cluster.
We can consider a service account is similar to a user.
But Service account and use account are two different things.
Reference [[4]](https://kubernetes.io/docs/concepts/security/service-accounts) provides a detailed explanation on the service account.
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
  - It doesn't give the permission for the following,
    - write access to the resource quota or to the namespace itself
    - write access to EndpointSlices (or Endpoints)
- edit
  - Provides permission to read and write access for most of the resources
  - It doesn't give the permission for the following,
    - viewing or modifying roles or role bindings
    - permissions aren't provided by the admin cluster role
- view
  - Provides read-only access for most of the resources
  - It doesn't give the permission for the following,
    - viewing roles or role bindings
    - viewing Secrets
    - permissions aren't provided by the edit cluster role

Since great power comes with great responsibility, it is good to use minimum permission as possible.

![Uncle Ben advice](https://i.giphy.com/media/v1.Y2lkPTc5MGI3NjExdXByam4wdWpsZzFydnlkaXg1ejNsZHR0b2MzdTd2enViZnVoY2N2OCZlcD12MV9pbnRlcm5hbF9naWZfYnlfaWQmY3Q9Zw/10KIsXhwdoerHW/giphy.gif)

In this example kubernetes configuration, we use the predefined admin cluster role.

***Cluster Role Binding:*** 
Cluster role binding attaches a cluster role to a service account.
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

We can generate an access token using the following.

```shell
kubectl -n kubernetes-dashboard create token admin-user
```

## Accessing kubernetes dashboard

There are two ways of accessing kubernetes dashboard.

1. kubectl port-forward **(I prefer this way)**
   - execute the following command in the terminal,
       ```shell
       kubectl -n kubernetes-dashboard port-forward svc/kubernetes-dashboard-kong-proxy 8443:443
       ```
   - Now we can access kubernetes dashboard from `https://localhost:8443` **(url in Reply-A)**
   - We use https for accessing, because the url in Reply-A is a https request
   - If the url in Reply-A is a http request, we have to access the kubernetes dashboard with http
   - Since we'll get a short url, I prefer this method.

2. kubectl proxy
   - execute the following command in the terminal,
     ```shell
      kubectl proxy --port=8001
      ```
   - Now we can access kubernetes dashboard
     from `http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard-kong-proxy:443/proxy/`

## Create long-lived Bearer Token for accessing Kubernetes dashboard

We can create a token and save it as a secret with the following YAML configuration.

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

Then we can get the created token by executing the following command on the terminal.

```shell
kubectl get secret admin-user -n kubernetes-dashboard -o jsonpath={".data.token"} | base64 -d
```

#### _That's it! We’ve successfully deployed kubernetes dashboard on our cluster._</h4>

![minions hurray](https://i.giphy.com/media/v1.Y2lkPTc5MGI3NjExbTliNHo3YWc4NHZxMTV0bjk0cHN0emZraWM0ODBmcWxtb3VxNGs3MCZlcD12MV9pbnRlcm5hbF9naWZfYnlfaWQmY3Q9Zw/11sBLVxNs7v6WA/giphy.gif)

## Pulumi Infrastructure as Code for Kubernetes dashboard deployment (Optional)

> [<svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" fill="currentColor" class="bi bi-github" viewBox="0 0 16 16">
<path d="M8 0C3.58 0 0 3.58 0 8c0 3.54 2.29 6.53 5.47 7.59.4.07.55-.17.55-.38 0-.19-.01-.82-.01-1.49-2.01.37-2.53-.49-2.69-.94-.09-.23-.48-.94-.82-1.13-.28-.15-.68-.52-.01-.53.63-.01 1.08.58 1.23.82.72 1.21 1.87.87 2.33.66.07-.52.28-.87.51-1.07-1.78-.2-3.64-.89-3.64-3.95 0-.87.31-1.59.82-2.15-.08-.2-.36-1.02.08-2.12 0 0 .67-.21 2.2.82.64-.18 1.32-.27 2-.27s1.36.09 2 .27c1.53-1.04 2.2-.82 2.2-.82.44 1.1.16 1.92.08 2.12.51.56.82 1.27.82 2.15 0 3.07-1.87 3.75-3.65 3.95.29.25.54.73.54 1.48 0 1.07-.01 1.93-.01 2.2 0 .21.15.46.55.38A8.01 8.01 0 0 0 16 8c0-4.42-3.58-8-8-8"/>
</svg>&nbsp;&nbsp;simple-k8s](https://github.com/dilanka-rathnasiri/simple-k8s)

This GitHub project is an example of Infrastructure as Code with [Pulumi](https://www.pulumi.com) for Kubernetes dashboard deployment.
Since this article isn't focusing on Infrastructure as Code, we'll not discuss the code here.
But I'll think this will be helpful for getting a rough idea.

## Summary
In this article, we’ve discussed deploying Kubernetes dashboard on a Kubernetes cluster.
Kubernetes dashboard is one of the easiest ways to get an overview of a kubernetes cluster.
Hands-on experience of the Kubernetes dashboard is valuable on the learning Kubernetes.
Also, it'll be a good jumpstart.
Even though the Kubernetes dashboard provides an overview of the cluster to some extent,
I think it'll not be enough for an advanced production Kubernetes cluster.

## References
1. https://github.com/kubernetes/dashboard/blob/master/docs/user/accessing-dashboard/README.md
2. https://github.com/kubernetes/dashboard/blob/master/docs/user/access-control/creating-sample-user.md
3. https://kubernetes.io/docs/reference/access-authn-authz/rbac/#role-and-clusterrole
4. https://kubernetes.io/docs/concepts/security/service-accounts
