# Setting Load Balancing for Applications on EKS

- In the earlier chapters, we've learnt how you can use a NodePort, Ingress, and/or AWS Load Balancer (ELB) to expose a simple application.
- In this section, you will demonstrate on how to scale and provide greater resilience in your application using AWS Elastic Load Balancers (ELBs).

## 01. Which load balancers are available in AWS?

- In Amazon EKS, you can use the following two types of AWS load balancer services:
  1. Network Load Balancer (NLB) - Layer-04
  2. Application Load Balancer (ALB) - Layer-07

## 02. `Proxy` and `Direct Server Return (DSR)` Modes

## 03. Choosing the right ELB

- Imagine that you want to expose a simple microservice running on EKS to the outside world in a scalable and resilient manner. Which ELB would you choose?
- Here are some questions you might want to ask to help you make your decision:

  - _What type of interface are you exposing?_ If it's not based on HTTP/HTTPS or HTTPv2, you will need to use an NLB.
  - _Do you want to offload encryption?_ If so, in most cases, you will want to use SSL/TLS, and in a lot of cases, you will offload the encryption/decryption process to the ELB.
  - _Do you need advanced web security?_ Only an ALB integrates with AWS Web Application Firewall (WAF) and AWS Shield, which acts as a distributed-denial-of-service (DDoS) protection service.
  - _Do you need low latency, or can you cope with large traffic bursts?_ As the NLB is an SDN construct, it is very low-latency and scales much faster than an ALB.

## 04. `Lab` - Deploying an application on EKS cluster and make it accessible within the cluster using `clusterIP` service

## 05. `Lab` - Deploying nginx application on EKS cluster and publish it using `nodeport` service

- For this lab manifest, you may refer [manifests/nodeport-svc-for-nginx-app.yml](./manifests/nodeport-svc-for-nginx-app.yml)

```
# To execute the above manifest >> This will deploy nginx app and a nodeport service to publish it
kubectl apply -f nodeport-svc-for-nginx-app.yml

# Get the list of all the services
kubectl get svc

[The preceding command will list services in which you will also find nginx-service nodeport svc]
```

- **Open the worker node's port 30080 to allow ingress traffic on this port**.

  - EC2 >> Instances >> Select any of your worker node >> select Security tab >> open attached security group on a new tab >> Edit Ingress Rules
  - Rule Specs
    - Type: Custom TCP
    - Port Range: 30080
    - Destination: Anywhere-IPv4
    - Description: Allowing k8s nodeport svc traffic

- **Access the application**
  - Launch a browser and hit this URL: <your_worker_node_public_ip>:30080
  - You should see a sample _Welcome to nginx_ page popping-up.

## 06. `Lab` - Deploying an application on EKS cluster and publish it using k8s `loadbalancer` service & AWS Application Load Balancer (ALB)
