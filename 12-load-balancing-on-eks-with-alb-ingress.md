# Load Balancing in EKS using `AWS Load Balancer Controller` (earlier name - AWS ALB Ingress Controller)

## Key Concepts

- Kubernetes Services - clusterIP, nodePort, loadBalancer
- Kubernetes Deployments and Pods
- IngressClass
- Ingress (resource)
- AWS IAM Roles and Policies
- IRSA and OIDC

## 01. Hands-on: Publishing K8s Application using Ingress (AWS Load Balancer Controller) with `Default backend`

- Reference Doc - https://docs.aws.amazon.com/eks/latest/userguide/aws-load-balancer-controller.html

### Step-1.1: Prerequisites

- AWS CLI
- eksctl and kubectl
- [AWS EKS Cluster with atleast one worker node](./02-setting-up-eks-cluster.md)
- OIDC Provider
  ```
  eksctl utils associate-iam-oidc-provider --region us-east-1 --cluster labekscluster --approve
  ```

### Step-1.2: Create an AWS IAM Policy

- Create an IAM policy using AWS CLI for the AWS Load Balancer Controller that allows it to make calls to AWS APIs on your behalf.

- Official GitHub repo: https://github.com/kubernetes-sigs/aws-load-balancer-controller

```
# Download the IAM policy doc
curl -o .\manifests\alb-manifests\iam_policy_latest.json https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/main/docs/install/iam_policy.json

# Verify the downloaded file
ls -l

# Create an AWS IAM Policy using the downloaded policy (.json) file
aws iam create-policy --policy-name AWSLoadBalancerControllerIAMPolicy --policy-document file://manifests\alb-manifests\iam_policy_latest.json


[Keep the above created iam policy name & ARN handy. We'll need it in later steps]
e.g. arn:aws:iam::15451128558:policy/AWSLoadBalancerControllerIAMPolicy
```

### Step-1.3: Create an `AWS IAM Role` & `K8s serviceAccount` for AWS Load Balancer Controller

- **Create an AWS IAM Role and K8s serviceAccounr using eksctl**

```
# Verify if any existing service account
kubectl get sa -n kube-system
kubectl get sa aws-load-balancer-controller -n kube-system

[If no serviceAccount exists with name aws-load-balancer-controller, continue with the next step]

# Create iamserviceaccount | replace below policy arn with your IAM Policy arn created in prev step | IAM >> Policy
eksctl create iamserviceaccount --cluster=labekscluster --namespace=kube-system --name=aws-load-balancer-controller  --attach-policy-arn=arn:aws:iam::154511248558:policy/AWSLoadBalancerControllerIAMPolicy --override-existing-serviceaccounts --approve

# Get IAM Service Account
eksctl get iamserviceaccount --cluster labekscluster
```

- If you want to see the created IAM Role from the AWS Management console, navigate to AWS Management console >> IAM >> Roles >> Search for IAM Role name created in the preceding step.

- Now, let's describe the above created serviceAccount

```
# List all the serviceaccounts present in kube-system namespace
kubectl get sa -n kube-system

# Describe aws-load-balancer-controller serviceaccount
kubectl describe sa aws-load-balancer-controller -n kube-system

[You should see the IAM role reference]
```

### Step-1.4: Install AWS Load Balancer Controller (ingress) using HELM

#### Step-1.4.1: Install Helm (v3)

- Reference:
  - https://helm.sh/docs/intro/install/
  - https://chocolatey.org/install
  - https://docs.aws.amazon.com/eks/latest/userguide/helm.html

```
choco install kubernetes-helm
```

### Step-1.5: Install the `AWS Load Balancer Controller` using Helm (v3)

```
# Add the eks-charts repository
helm repo add eks https://aws.github.io/eks-charts

# Update your local repo
helm repo update

# Install the AWS Load Balancer Controller | REPLACE VPC ID WITH YOURS
helm install aws-load-balancer-controller eks/aws-load-balancer-controller -n kube-system --set clusterName=labekscluster --set serviceAccount.create=false --set serviceAccount.name=aws-load-balancer-controller

# Verify that the AWS Load Balancer controller is installed
kubectl get deployment -n kube-system
kubectl get deployment aws-load-balancer-controller -n kube-system
kubectl describe deployment aws-load-balancer-controller -n kube-system

# Verify AWS Load Balancer controller Webhook clusterIP service is created
kubectl get svc -n kube-system
kubectl get svc aws-load-balancer-webhook-service -n kube-system
kubectl describe svc aws-load-balancer-webhook-service -n kube-system

# Verify Labels in Service and Selector Labels in Deployment
kubectl get svc aws-load-balancer-webhook-service -n kube-system -o yaml
kubectl get deployment aws-load-balancer-controller -n kube-system -o yaml

Observation:
1. Verify "spec.selector" label in "aws-load-balancer-webhook-service"
2. Compare it with "aws-load-balancer-controller" Deployment "spec.selector.matchLabels"
3. Both values should be same which traffic coming to "aws-load-balancer-webhook-service" on port 443 will be sent to port 9443 on "aws-load-balancer-controller" deployment related pods.
```

### Step-1.6: Deploy a sample stateless app (nginx) using k8s Deployment

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app1-nginx-deployment
  labels:
    app: app1-nginx
spec:
  replicas: 1
  selector:
    matchLabels:
      app: app1-nginx
  template:
    metadata:
      labels:
        app: app1-nginx
    spec:
      containers:
        - name: app1-nginx
          image: nginx:latest
          ports:
            - containerPort: 80
```

### Step-1.7: Create a `NodePort service` for the nginx application deployment

```
apiVersion: v1
kind: Service
metadata:
  name: app1-nginx-nodeport-service
  labels:
    app: app1-nginx
spec:
  type: NodePort
  selector:
    app: app1-nginx
  ports:
    - port: 80
      targetPort: 80
```

### Step-1.8: Create an `IngressClass` for AWS Load Balancer Controller

```
apiVersion: networking.k8s.io/v1
kind: IngressClass
metadata:
  name: my-aws-ingress-class
  annotations:
    ingressclass.kubernetes.io/is-default-class: "true"
    # (optional) This annotation is required if you've multiple ingress controller
spec:
  controller: ingress.k8s.aws/alb
  # (optional) You may pass other parameters for the controller (e.g., subnets, security groups)
```

### Step-1.9: Deploy all the K8s manifests

```
# Create IngressClass Resource
kubectl apply -f kube-manifests\

# Verify the created resources
kubectl get ingressclass,pods,svc,deploy
```

### Step-1.10: Create an `Ingress resource` with a `default backend`

AWS Load Balancer Controller - [Annotations](https://kubernetes-sigs.github.io/aws-load-balancer-controller/latest/guide/ingress/annotations/)

```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-nginxapp1
  labels:
    app: app1-nginx
  annotations:
    # Load Balancer Name
    alb.ingress.kubernetes.io/load-balancer-name: app1ingressrules
    alb.ingress.kubernetes.io/scheme: internet-facing
    alb.ingress.kubernetes.io/healthcheck-protocol: HTTP
    alb.ingress.kubernetes.io/healthcheck-port: traffic-port
    alb.ingress.kubernetes.io/healthcheck-path: /
    alb.ingress.kubernetes.io/healthcheck-interval-seconds: "15"
    alb.ingress.kubernetes.io/healthcheck-timeout-seconds: "5"
    alb.ingress.kubernetes.io/success-codes: "200"
    alb.ingress.kubernetes.io/healthy-threshold-count: "2"
    alb.ingress.kubernetes.io/unhealthy-threshold-count: "2"
spec:
  ingressClassName: my-aws-ingress-class # Ingress Class
  defaultBackend:
    service:
      name: app1-nginx-nodeport-service
      port:
        number: 80
```

### Step-1.11: Verify the AWS Load Balancer and it's sepcifications

- Navigate to the AWS Management Console >> EC2 >> Load Balancers >> Select the Load Balancer
  1. Verify Listeners and Rules inside the listener
  2. Verify Target Groups

### Step-1.12: Access App the application over the web

- Copy the Load Balancer DNS Endpoint either from the AWS Management console of using the following commands:

```
kubectl get ingress

# Sample from my environment (for reference only)
http://app1ingress-154912460.us-east-1.elb.amazonaws.com

# Verify AWS Load Balancer Controller pod logs
kubectl get po -n kube-system

## Pod-1 Logs:
kubectl -n kube-system logs -f <POD1-NAME>
kubectl -n kube-system logs -f aws-load-balancer-controller-23b4f64d6c-h2vh4

## Pod-2 Logs:
kubectl -n kube-system logs -f <POD2-NAME>
kubectl -n kube-system logs -f aws-load-balancer-controller-12b4f64d6c-t7qqb
```

## 02. Hands-on: Publishing K8s Application using Ingress (AWS Load Balancer Controller) with `ingress rules`
