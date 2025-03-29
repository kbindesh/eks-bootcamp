## 01. `Lab` - Deploying an application on EKS cluster and make it accessible within the cluster using `clusterIP` service

## 02. `Lab` - Deploying nginx application on EKS cluster and publish it using `nodeport` service

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

**NOTE**: As per k8s best practices, nodeport service is not a recommended solution for making your application accessible to outside k8s cluster users.

## 03. `Lab` - Deploying an application on EKS cluster and publish it using k8s `loadbalancer` service & AWS Network Load Balancer (NLB)

### 3.1 Create a NLB service and Application manifests

- You may refer to this manifest: [](./manifests/nlb-nodeport-svc-for-app.yml)

```
kubectl apply -f nlb-nodeport-svc-for-app.yml
```

- Now, switch to AWS Management console >> EC2 >> Load Balancer, and you should see an elastic load balancer of type _network_ getting created.

### 3.2 Access the application over the Internet

- Get the NLB URL and hit it through your browser. You should see your application default landing page.
