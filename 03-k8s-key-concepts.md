# Kubernetes key concepts

- Kubernetes init containers
- Liveness and Readiness probes
- Resource management for pods and containers (requests and limits)
- Namespaces
- Labels and Annotations

## 01. Kubernetes `init containers`

## 02. `Liveness` and `Readiness probes`

### 2.1 Create Liveness probe

### 2.2 Create Readiness probe (HTTP GET)

## 03. `Resource management` for Pods & Containers

- In kubernetes, we can control how much resource (cpu, memory) to allocate for each container of a pods.
- When you provide this info in the pod spec, the scheduler uses this info to find a worker node to deploy a pod which has a requested amount of resources.
- When you request resources for your pod, the _kubelet_ enforces those limits so that the running container is not allowed to use more of that resource than the limit you set.

### 3.1 Pod Definition with resource `Requestes & Limits`

```
spec:
  containers:
    - name: log-aggregator
      image: kbindesh/log-aggregator:v5
      resources:
        requests:
          memory: "128Mi"    # 128 MebiByte is equal to 135 Megabyte (MB)
          cpu: "500m"        # Here, "m" means milliCPU or millicore
        limits:
          memory: "256Mi"
          cpu: "1000m"       # 1000m is equal to 1 VCPU core
```

### 3.2 Lab: Deploy a Pod with resource requests & limits

- Kindly refer to this sample manifest [manifests/app-deploy-with-resource-limits.yml](./manifests/app-deploy-with-resource-limits.yml) which demonstrate how to deploy an application in k8s imposing resource requests & limits

- Now, execute the above manifest:

```
kubectl apply -f app-deploy-with-resource-limits.yml

# List Deployments
kubectl get deployments

# List pods
kubctl get pods

# Describe any pod to check the requested resource details
kubectl describe pod <pod_name>

# Delete all the resources associated with above app deployment
kubectl delete -f app-deploy-with-resource-limits.yml
OR
kubectl delete <deployment_name>
```

## References

- https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/
