# Monitoring and Logging K8s Cluster and Apps

## 01. K8s Metrics Server

- In K8s, **Metrics Server** collects CPU/memory metrics and to some extent adjusts the resources needed by containers automatically.
- Metrics Server collects those metrics every 15 seconds from the kubelet agent and then exposes them in the API server of the Kubernetes master via the Metrics API.

```
alias k=kubectl
k get nodes

# To access metrics collected by Metrics Server
k top

# To check if Metrics Server is installed
k top node minikube

[If you get an error "error: Metrics API not available", it means Metrics Server is not available in your current Kubernetes cluster]

# (alternatively)  To check if Metrics Server is installed
kubectl get pods -n kube-system | grep metrics-server

```

### Installing Metrics Server on K8s cluster

- **Using a YAML manifest file**

```
kubectl apply -f https://github.com/kubernetes-sigs/metricsserver/
releases/latest/download/components.yaml
```

- **Using Helm Chart**

  - To install Metrics Server using Helm charts, you can go to Artifact Hub and then find the Metrics Server Helm charts at https://artifacthub.io/packages/helm/metrics-server/metrics-server

    ```
    # To Install
    helm repo add metrics-server https://kubernetes-sigs.github.io/metrics-server/

    # To upgrade
    helm upgrade --install metrics-server metrics-server/metricsserver
    ```

- Using minikube add-on

```
minikube addons list

minikube addons list | grep metrics-server

minikube addons enable metrics-server

kubectl get pod,svc -n kube-system

kubectl get pods -n kube-system | grep metrics-server
```

### Checking out CPU/memory metrics

```
k top node minikube
```

## 02. Monitoring applications on a Kubernetes cluster

- In K8s, Metrics Server is not only used to monitor the Kubernetes worker nodes but also Kubernetes Pods and containers.

```
# Create a new Pod to demonstrate monitoring
kubectl run nginx --image=nginx

kubectl get pod nginx
```

- **Monitoring the resource usage of an application**

```
k top pod nginx

# If there are multiple containers, it will list the name of the containers in that pod
k top pod nginx --containers

# show all the metrics of all the Pods across different namespaces
k top pod -A
OR
k top pod --all-namespaces

kubectl top pod --sort-by=cpu
kubectl top pod –-sort-by=memory

kubectl top pod -A --sort-by=memory
```

- **Checking application details**

```
kubectl describe pod nginx

[Observe the Events section at the bottom of the preceding command result that shows a logs of recent events related to the pod]
```

## 03. Monitoring cluster events

```
kubectl get events

kubectl get events --sort-by=.metadata.creationTimestamp

# To collect the events during a deployment
kubectl get events --watch
```

## 04. Managing logs at the cluster node and Pod levels

```
# Checking out the node details
kubectl describe node minikube

# Checking the node status
kubectl get node minikube -o wide
```

## 05. Managing container stdout and stderr logs

- In the Unix and Linux OSs, there are three I/O streams, called STDIN, STDOUT, and STDERR.

- STDOUT is usually a command’s normal output, and STDERR is typically used to output error messages.

- Kubernetes uses the kubectl logs <podname> command to log STDOUT and STDERR.

```
kubectl logs <podname>

kubectl logs nginx
```

### Example

- Now, we'll use a container to write text to the standard output stream with a frequency of once per second. We can do this by deploying a new pod.

```
apiVersion: v1
kind: Pod
metadata:
  name: logger
spec:
  containers:
  - name: packs
    image: busybox:1.28
    args: [/bin/sh, -c,'i=0; while true; do echo "$i: $(date)";i=$((i+1)); sleep 1; done']
```

- You can use the kubectl logs command to retrieve the logs from the logger Pod as follows

```
k logs logger

# to retrieve the specific container log by using the -c flag
k logs logger -c packs

# to stream the logs
kubectl logs -f logger

# to return logs newer than a certain duration
kubectl logs --since=1h
```
