# Building scalable applications in Kubernetes

In this section we will learn the following concepts:

- Pod resource requests & limits
- Horizontal vs Vertical scaling
- Horizontal Pod Autoscaler (HPA)
- Vertical Pod Autoscaler (VPA)
- Cluster Autoscaler in EKS

## 01. Pods with resource requests & limits - Qn

- Kubernetes allows us to specify the resource requirements of a container in the pod specification, which basically refers to how much resource (cpu, memory) a container needs.
- It usually gives us the following values:
- **resources.limits.cpu** is the resource limit set on CPU usage.
- **resources.limits.memory** is the resource limit set on memory usage.
- **resources.requests.cpu** is the minimum CPU usage requested to allow your application to be up and running.
- **resources.requests.memory** is the minimum memory usage requested to allow your application to be up and running.
- **resources.limits.ephemeral-storage** is the limit on ephemeral storage resources.

### Create a Pod with resource requests & limits specs

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

- Execute the above manifests and check the usuage:

```
kubectl apply -f pod-with-resource-specs.yml

# Get the list of Pods
kubectl get pods

# To check the allocation resources of that worker node
kubectl describe node

# To check the actual resource usage of the node or pod | metrics server must be installed
kubectl top
```

## 02. Autoscaling your apps using `HorizontalPodAutoScaler (HPA)`

- Horizontal scaling is basically about automatically increasing and decreasing the no. of pods (replicas) based on the incoming load.

- Kubernetes achieves horizontal autoscaling using _HorizontalPodAutoScaler (HPA)_.

- HPA automatically scales the number of pods in a deployment, replicationController or replicaSet, statefulSet based on resource's CPU utilization.

- HPA needs K8s metrics server to monitor and get the CPU metrics of a Pod.

### Step-2.1: Install Metrics Server

- By default Amazon EKS comes with pre-installed metrics server and you need not install it manually.

```
# Check if metrics server is already installed
kubectl get deployment metrics-server -n kube-system

[If the preceding command list 'metrics-server' deployment, that mean metrics server is already installed and you can skip this entire step]

# (optional) Install Metrics Server
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/download/v0.3.6/components.yaml

# Verify the metrics-server by listing deployments from kube-system namespace
kubectl get deployment metrics-server -n kube-system
```

### Step-2.2: Deploy Application

- The following k8s manifest _php-apache-deployment.yml_ deploys an app deployment (nginx) and a clusterIP service.

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: php-apache
spec:
  selector:
    matchLabels:
      run: php-apache
  template:
    metadata:
      labels:
        run: php-apache
    spec:
      containers:
      - name: php-apache
        image: registry.k8s.io/hpa-example
        ports:
        - containerPort: 80
        resources:
          limits:
            cpu: 500m
          requests:
            cpu: 200m
---
apiVersion: v1
kind: Service
metadata:
  name: php-apache
  labels:
    run: php-apache
spec:
  type: ClusterIP
  ports:
  - port: 80
    targetPort: 80
  selector:
    run: php-apache
```

- **Apply the above manifest to create a deployment and a clusterIP service**:

```
kubectl apply -f php-apache-deployment.yml

# List deployed Pods, Service and Deployment
kubectl get po,svc,deploy
```

### Step-2.3: Create a HorizontalPodAutoscaler for the above deployed Application

- Create a Horizontal Pod Autoscaler resource for the php-apache deployment using following manifest _php-apache-hpa.yml_:

```
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: php-apache-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: php-apache
  minReplicas: 1
  maxReplicas: 5
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 50
```

- Execute the above manifest to create a Horizontal Pod Autoscaler resource for the php-apache deployment:

```
kubectl apply -f php-apache-hpa.yml

kubectl get hpa
```

### Step-2.4: Add load to the Application and Verify the scaling behaviour

- Create a load for the web server by running a another Pod (container):

```
kubectl run -i --tty load-generator --rm --image=busybox --restart=Never -- /bin/sh -c "while sleep 0.01; do wget -q -O- http://php-apache; done"
```

- To see the deployment scale out happening, periodically run the following command in a separate terminal from the terminal that you ran the previous step in:

```
kubectl get hpa php-apache
```

- It may take a minute or two for the replica count to increase. As long as actual CPU percentage is higher than the target percentage, the replica count keep increasing, up to 5.

- Now, stop the load by exiting from the load-generator pod by pressing Ctrl+C keys. You will now see the number of replicas scaled down to 1.

### Step-2.5: Clean-up the Resources

```
kubectl delete deployment.apps/php-apache service/php-apache horizontalpodautoscaler.autoscaling/php-apache
```

## 03. Autoscaling your apps using `VerticalPodAutoScaler (VPA)`

## 04. Scaling your EKS cluster using `Cluster Autoscaler`

- The EKS cluster autoscaler automatically adjusts the no. of nodes in your cluster when pods fail to launch due to lack of resources or when nodes in the cluster are underutilized.

```
# To verify if your NodeGroup has autoscaler enabled (--asg-access), check eksctl create nodegroup command

eksctl create nodegroup --asg-access
```

# References

- https://docs.aws.amazon.com/eks/latest/userguide/horizontal-pod-autoscaler.html
- https://docs.aws.amazon.com/eks/latest/userguide/vertical-pod-autoscaler.html
