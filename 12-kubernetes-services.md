# Kubernetes Services

- Pods are mortal and immutable. In other terms, pods are unreliable and you should never connect to them directly.
- You should always connect to them through service.

## 01. `Service` Overview

- Kubernetes treats Pods as ephemeral objects and deletes them when any of the following
  events occur:

  - Rolling updates
  - Scale-down operations
  - Rollbacks
  - Failures

- K8s has a solution to it above issue as _Service_.
- _Service_ objects sits in front of one or more identical Pods and expose them via a reliable DNS name, IP address, and port.

## 02. Service Types

- Kubernetes has several types of services, major ones are as follows:
  1. ClusterIP Service
  2. NodePort Service
  3. LoadBalancer Service

### 2.1. `ClusterIP` Service

- ClusterIP is the default service type.
- It gets a name and IP that is programmed into the internal network fabric.
- ClusterIP service is accessible only from inside the cluster.

- Let's consider an example.

  - You're deploying an application called _nginx_ and you want other apps on the cluster to access it by it's name of IP address.
  - To satisfy these requirements, you create a new ClusterIP service called _nginx-cip-svc_.

- The following k8s manifest _nginx-cip-svc-demo.yml_ will deploy nginx application with 3 replicas (pods) and a ClusterIP service, which will sit in front of nginx app pods:

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
        - name: nginx
          image: nginx:latest
          ports:
            - containerPort: 80
          resources:
            requests:
              memory: "64Mi"
              cpu: "250m"
            limits:
              memory: "128Mi"
              cpu: "500m"
---
apiVersion: v1
kind: Service
metadata:
  name: bin-service
spec:
  type: ClusterIP
  selector:
    app: nginx
  ports:
  - protocol: TCP
    port: 8080
    targetPort: 80
```

- Apply the above manifest:

```
kubectl apply -f nginx-cip-svc-demo.yml

# List services
kubectl get svc

# List Deployments
kubectl get deploy
```

### 2.2. NodePort Service

### 2.3. LoadBalancer Service

- Create a k8s manifest for creating a loadbalancer service on EKS cluster followed by AWS NLB:

```
apiVersion: v1
kind: Service
metadata:
  name: voteapp-lb-svc
  labels:
    app: vote
  annotations:
    service.beta.kubernetes.io/aws-load-balancer-type: nlb    # It will create AWS Network Load Balancer
spec:
  type: LoadBalancer
  selector:
    app: vote
  ports:
  - port: 80
    targetPort: 8080
```

- Deploy all the voting app manifests:

```
# Deploy all manifests
kubectl apply -f k8s-manifests/

# List Services | Check for above created LoadBalancer service
kubectl get svc

# Verify Pods
kubectl get pods
```

- As a result of above deployment, an AWS NLB will be created which you can check from the AWS management console >> EC2 >> Load Balancers.
