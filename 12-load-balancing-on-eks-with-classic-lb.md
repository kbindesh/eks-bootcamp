# Setting Load Balancing for Applications on EKS

## 01. Which load balancers are available in AWS?

- In Amazon EKS, you can use the following three types of AWS load balancer services:

  1. Network Load Balancer (NLB) - Layer-04
  2. Application Load Balancer (ALB) - Layer-07
  3. Classic Load Balancer (will be deprecated soon)

- Ref: https://aws.amazon.com/compare/the-difference-between-the-difference-between-application-network-and-gateway-load-balancing/

## 02. `Proxy` and `Direct Server Return (DSR)` Modes

## 03. Choosing the right AWS ELB

- Imagine that you want to expose a simple microservice running on EKS to the outside world in a scalable and resilient manner. Which ELB would you choose?
- Here are some questions you might want to ask to help you make your decision:

  - _What type of interface are you exposing?_ If it's not based on HTTP/HTTPS or HTTPv2, you will need to use an NLB.
  - _Do you want to offload encryption?_ If so, in most cases, you will want to use SSL/TLS, and in a lot of cases, you will offload the encryption/decryption process to the ELB.
  - _Do you need advanced web security?_ Only an ALB integrates with AWS Web Application Firewall (WAF) and AWS Shield, which acts as a distributed-denial-of-service (DDoS) protection service.
  - _Do you need low latency, or can you cope with large traffic bursts?_ As the NLB is an SDN construct, it is very low-latency and scales much faster than an ALB.

## 04. `Lab` - Deploying an application on EKS cluster and publish it using k8s `loadbalancer` service & AWS Classic Load Balancer

- **IMP**: K8s _loadbalancer service_ is available only with the managed kubernetes services (like EKS, AKS, GKE) as it uses external load balancer services to distribute the load.

- For this lab manifest, you may refer [manifests/loadbalancer-svc-for-nginx-app.yml](./manifests/loadbalancer-svc-for-nginx-app.yml)

```
# To execute the above manifest >> This will deploy nginx app and a nodeport service to publish it
kubectl apply -f loadbalancer-svc-for-nginx-app.yml

# Get the list of all the services
kubectl get svc

[The preceding command will list services in which you will find nginx-lb-service loadBalancer service]
```

- You can also check the created resources and associated load balancer details from the AWS management console.

[Make sure you have necessary permissions to view the EKS resources from the AWS console]

- Executing above manifest will also create a new AWS Classic load balancer; to see, navigate to AWS console >> EC2 >> Load Balancers.

- Now, to access the EKS hosted nginx application over a load balancer, copy the
  _DNS Name_ of the AWS load balancer and visit it through your browser.

```
# Sample classic load balancer DNS name
http://khk234j23j4j23j4kkjk2k-1151857589.eu-west-3.elb.amazonaws.com
```

[You should see the nginx sample page]

- Enable AWS classic load balancer to listen on port 443 (HTTPS)

- Allow AWS classic loadbalancer ingress traffic on port 443 (HTTPS)
