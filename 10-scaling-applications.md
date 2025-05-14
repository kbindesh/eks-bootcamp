# Building scalable applications in Kubernetes

In this section we will learn the following concepts:

- Pod resource requests & limits
- Horizontal vs Vertical scaling
- Horizontal Pod Autoscaler (HPA)
- Vertical Pod Autoscaler (VPA)
- Cluster Autoscaler in EKS

## xx. Pods with resource requests & limits - Qn

- Kubernetes allows us to specify the resource requirements of a container in the pod specification, which basically refers to how much resource (cpu, memory) a container needs.
- It usually gives us the following values:
- **resources.limits.cpu** is the resource limit set on CPU usage.
- **resources.limits.memory** is the resource limit set on memory usage.
- **resources.requests.cpu** is the minimum CPU usage requested to allow your application to be up and running.
- **resources.requests.memory** is the minimum memory usage requested to allow your application to be up and running.
- **resources.limits.ephemeral-storage** is the limit on ephemeral storage resources.

### Hands-on: Pod with resource requests & limits specs

- The following k8s manifest is an example of defining a pod with resource request and limits:

```
apiVersion: v1
kind: Pod
metadata:
  name: binapp-pod
spec:
containers:
- name: binapp-container
  image: busybox
  command: ['sh', '-c', 'echo stay there! && sleep 3600']
  resources:
    requests:
      memory: "64Mi"      # Minimum 64 MB of memory
      cpu: "250m"
    limits:
      memory: "128Mi"     # Maximum 128 MB of memory
      cpu: "500m"
```

- Execute the above manifests:

```
kubectl apply -f pod-with-resource-specs.yml

# Get the list of Pods
kubectl get pods

# To check the allocation resources of that worker node
kubectl describe node

# To check the actual resource usage of the node or pod | metrics server must be installed
kubectl top
```

## xx. Autoscaling your apps using HorizontalPodAutoScaler (HPA)

## xx. Autoscaling your apps VerticalPodAutoScaler (VPA)

## xx. Scaling your EKS cluster using Cluster Autoscaler
