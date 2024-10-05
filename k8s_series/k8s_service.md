# Kubernetes Service

In this article, we will talk about the Kubernetes service.

### What is a Kubernetes Service?

Kubernetes service is an abstraction to expose a group of pods over a network.

### Why Kubernetes service

- Since Kubernetes pods can be created and destroyed dynamically 
  - Pods won't have a stable address to access
  - So, the service provides a stable address to access
- Access to multiple pods using a single endpoint
- Separate application logic from the underlying pods
  - So, no need of worrying about the pod count or pod health status
- Service provides load balancing
  - So, client requests are balanced across the pods

### How to define a Kubernetes service

The following is an example manifest for a Kubernetes service
```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    app: challenger-app
    project: dodge
  ports:
    - protocol: TCP
      port: 80
      targetPort: 5000
```

The 

### Service types

### Discovering services

### What is a headless service?

### Multi-port services

### Summary
In this article, we’ve discussed the kubernetes service.
I hope this article will help you to get an overview idea of the Kubernetes service.

### Reference
1. https://kubernetes.io/docs/concepts/services-networking/service/
2. https://youtu.be/T4Z7visMM4E?si=Ls93-PvscZYC2nfH
3. https://cloud.google.com/kubernetes-engine/docs/concepts/service