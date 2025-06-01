# Load Balancing in EKS using `AWS Load Balancer Controller` (earlier name - AWS ALB Ingress Controller)

## What is AWS Load Balancer Controller (ALBIC)?

- We will use Kubernetes Ingress resource to create ALB
- We will use Kubernetes Service resource to create NLB

## Random steps

```
eksctl utils associate-iam-oidc-provider --region us-east-1 --cluster labekscluster --approve
```

### Create IAM Policies

```
# Download the IAM policy doc
curl -o .\manifests\alb-manifests\iam_policy_latest.json https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/main/docs/install/iam_policy.json

# Verify the downloaded file

# Create IAM Policy using policy downloaded
aws iam create-policy --policy-name AWSLoadBalancerControllerIAMPolicy --policy-document file://manifests\alb-manifests\iam_policy_latest.json


[Keep the above created iam policy arn handy. We'll need it in later steps]
arn:aws:iam::154511248558:policy/AWSLoadBalancerControllerIAMPolicy
```

### Create an IAM role for AWS LoadBalancer Controller and attach the role to the K8s service account

- Understand _IAM Roles for Service Accounts_ - https://eksctl.io/usage/iamserviceaccounts/

- **Create IAM Role using eksctl**

```
# Verify if any existing service account
kubectl get sa -n kube-system
kubectl get sa aws-load-balancer-controller -n kube-system

[If no serviceAccount exists with name aws-load-balancer-controller, continue with the next step]

eksctl create iamserviceaccount --cluster=labekscluster --namespace=kube-system --name=aws-load-balancer-controller  --attach-policy-arn=arn:aws:iam::154511248558:policy/AWSLoadBalancerControllerIAMPolicy --override-existing-serviceaccounts --approve

# Get IAM Service Account
eksctl  get iamserviceaccount --cluster labekscluster

[From the preceding command's output, keep the ROLE ARN value handy, as we'll need it in the next step ]
```

- If you want to see the created IAM Role, navigate to AWS Management console >> IAM >> Roles >> Search for IAM Role name created in the preceding step.

- Again verify and describe the created serviceAccount

```
kubectl get sa -n kube-system

kubectl get sa aws-load-balancer-controller -n kube-system
[This time you'll see new serviceAccount listed]

kubectl describe sa aws-load-balancer-controller -n kube-system
```

### (Optional) Install Helm (v3)

- Reference:
  - https://helm.sh/docs/intro/install/
  - https://chocolatey.org/install
  - https://docs.aws.amazon.com/eks/latest/userguide/helm.html

```
choco install kubernetes-helm
```

### Install the `AWS Load Balancer Controller` using Helm (v3)

```
# Add the eks-charts repository
helm repo add eks https://aws.github.io/eks-charts

# Update your local repo
helm repo update

# Install the AWS Load Balancer Controller | REPLACE VPC ID WITH YOURS
helm install aws-load-balancer-controller eks/aws-load-balancer-controller -n kube-system --set clusterName=labekscluster --set serviceAccount.create=false --set serviceAccount.name=aws-load-balancer-controller  --set region=us-east-1 --set vpcId=vpc-01778c7e810337ed9 --set image.repository=public.ecr.aws/eks/aws-load-balancer-controller

# Verify that the controller is installed and Webhook Service created
kubectl -n kube-system get deployment
kubectl -n kube-system get deployment aws-load-balancer-controller
kubectl -n kube-system describe deployment aws-load-balancer-controller

# Verify AWS Load Balancer Controller Webhook service created | ClusterIP Service
kubectl -n kube-system get svc
kubectl -n kube-system get svc aws-load-balancer-webhook-service
kubectl -n kube-system describe svc aws-load-balancer-webhook-service

# Verify Labels in Service and Selector Labels in Deployment
kubectl -n kube-system get svc aws-load-balancer-webhook-service -o yaml
kubectl -n kube-system get deployment aws-load-balancer-controller -o yaml
Observation:
1. Verify "spec.selector" label in "aws-load-balancer-webhook-service"
2. Compare it with "aws-load-balancer-controller" Deployment "spec.selector.matchLabels"
3. Both values should be same which traffic coming to "aws-load-balancer-webhook-service" on port 443 will be sent to port 9443 on "aws-load-balancer-controller" deployment related pods.
```

### Verify AWS Load Balancer Controller Logs

```
# List Pods
kubectl get pods -n kube-system

# Review logs for AWS LB Controller Pod-01
kubectl -n kube-system logs -f <POD-NAME>
kubectl -n kube-system logs -f  aws-load-balancer-controller-86bd6-5pjfk

# Review logs for AWS LB Controller Pod-02
kubectl -n kube-system logs -f <POD-NAME>
kubectl -n kube-system logs -f aws-load-balancer-controller-88cbd6-vqqsk
```

## AWS Load Balancer Controller - Ingress Basics

### Introduction

Understand the following Ingress Concepts

- Annotations
- ingressClassName
- defaultBackend
- rules

### Create Deployment kube-manifest

```
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: vote
  name: vote
spec:
  replicas: 1
  selector:
    matchLabels:
      app: vote
  template:
    metadata:
      labels:
        app: vote
    spec:
      containers:
      - image: dockersamples/examplevotingapp_vote
        name: vote
        ports:
        - containerPort: 80
          name: vote
```

### Create a NodePort service

```
apiVersion: v1
kind: Service
metadata:
  name: vote-app-nodeport-svc
  labels:
    app: vote
  annotations:
    #Important Note:  Need to add health check path annotations in service level if we are planning to use multiple targets in a load balancer
    #alb.ingress.kubernetes.io/healthcheck-path: /
spec:
  type: NodePort
  selector:
    app: vote
  ports:
    - port: 80
      targetPort: 80
```

### Create an IngressClass

```
apiVersion: networking.k8s.io/v1
kind: IngressClass
metadata:
  name: my-aws-ingress-class
  annotations:
    ingressclass.kubernetes.io/is-default-class: "true"
spec:
  controller: ingress.k8s.aws/alb
  # Optional:  Define other parameters for the controller (e.g., subnets, security groups)
```

```
# Create IngressClass Resource
kubectl apply -f kube-manifests

# Verify IngressClass Resource
kubectl get ingressclass

# Describe IngressClass Resource
kubectl describe ingressclass my-aws-ingress-class
```

### Create an Ingress manifest with default backend

AWS Load Balancer Controller - [Annotations](https://kubernetes-sigs.github.io/aws-load-balancer-controller/latest/guide/ingress/annotations/)

```
# Reference: https://kubernetes-sigs.github.io/aws-load-balancer-controller/latest/guide/ingress/annotations/
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-nginxapp1
  labels:
    app: vote
  annotations:
    #kubernetes.io/ingress.class: "alb"
    alb.ingress.kubernetes.io/scheme: internet-facing
    # Health Check Settings
    alb.ingress.kubernetes.io/healthcheck-protocol: HTTP
    alb.ingress.kubernetes.io/healthcheck-port: traffic-port
    alb.ingress.kubernetes.io/healthcheck-path: /
    alb.ingress.kubernetes.io/healthcheck-interval-seconds: '15'
    alb.ingress.kubernetes.io/healthcheck-timeout-seconds: '5'
    alb.ingress.kubernetes.io/success-codes: '200'
    alb.ingress.kubernetes.io/healthy-threshold-count: '2'
    alb.ingress.kubernetes.io/unhealthy-threshold-count: '2'
spec:
  ingressClassName: my-aws-ingress-class # Ingress Class
  defaultBackend:
    service:
      name: vote-app-nodeport-svc
      port:
        number: 80
```

### Deploy all the manifests

```
# Deploy kube-manifests
kubectl apply -f kube-manifests-default-backend/

# Verify k8s Deployment and Pods
kubectl get deploy
kubectl get pods

# Verify Ingress (Make a note of Address field)
kubectl get ingress

[Verify the ADDRESS value, we should see something like "app1ingress-1334515506.us-east-1.elb.amazonaws.com"]

# Describe Ingress Controller
kubectl describe ingress ingress-nginxapp1

[Review Default Backend and Rules]

# List Services
kubectl get svc

# Verify Application Load Balancer using
Goto AWS Mgmt Console -> Services -> EC2 -> Load Balancers
1. Verify Listeners and Rules inside a listener
2. Verify Target Groups

# Access App using Browser
kubectl get ingress
http://<ALB-DNS-URL>
http://<ALB-DNS-URL>/app1/index.html
or
http://<INGRESS-ADDRESS-FIELD>
http://<INGRESS-ADDRESS-FIELD>/app1/index.html

# Sample from my environment (for reference only)
http://app1ingress-154912460.us-east-1.elb.amazonaws.com
http://app1ingress-154912460.us-east-1.elb.amazonaws.com/app1/index.html

# Verify AWS Load Balancer Controller logs
kubectl get po -n kube-system
## POD1 Logs:
kubectl -n kube-system logs -f <POD1-NAME>
kubectl -n kube-system logs -f aws-load-balancer-controller-65b4f64d6c-h2vh4
##POD2 Logs:
kubectl -n kube-system logs -f <POD2-NAME>
kubectl -n kube-system logs -f aws-load-balancer-controller-65b4f64d6c-t7qqb
```
