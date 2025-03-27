# Setting-up an `Amazon Elastic Kubernetes Service (EKS) Cluster` on Win 11 system

- In this section, we will learn how to setup an Amazon EKS cluster, a Production-grade Kubernetes distribution.
- In order to setup the EKS cluster, we will perform following steps:
  - **Install AWS CLI**
  - **Install kubectl CLI**
  - **Install eksctl CLI**
  - **Create and configure EKS Cluster**
  - **Access the EKS cluster**

## Step-01: Set up the `AWS CLI`

- To provision resources in AWS from the command line, you need to obtain an AWS access key ID and secret key to use in the command line.
- Then you need to configure these credentials in the AWS CLI.

### 1.1 Download & Install AWS CLI

- For **AWS CLI install or update**, refer to this link: https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html

### 1.2 Create an access key

- Sign into the AWS Management Console.
- Navigate to **IAM** service >> **Create new User with just programmatic access (CLI)** >> Assign **AdministratorAccess** Policy.
- Choose the name of the user whose access keys you want to manage, and then choose the **Security credentials** tab.
- In the **Access keys** section, say **Create access key**
- Choose **Download .csv file** with keys.

### 1.3 Configure the AWS CLI

- After installing the AWS CLI, do the following steps to configure it.
- On command prompt/terminal, enter the following command:

```
aws configure
```

- Enter the requested details:
  - **AWS Access Key ID** [None]: <iam_user_access_key>
  - **AWS Secret Access Key** [None]: <iam_user_secret_access_key>
  - **Default region name** [None]: us-east-1
  - **Default output format** [None]: json

## Step-02: Install `kubectl` CLI

- For kubectl installation process, refer to this link:
  https://docs.aws.amazon.com/eks/latest/userguide/install-kubectl.html

- [Direct download link (Kubernetes 1.29) for Windows](https://s3.us-west-2.amazonaws.com/amazon-eks/1.29.0/2024-01-04/bin/windows/amd64/kubectl.exe)

- After you install kubectl, you can verify its version:

  ```
  kubectl version --client
  ```

- When first installing kubectl, it isn't yet configured to communicate with any server.

```
# If you ever need to update the eks cluster config
aws eks update-kubeconfig --region region-code --name my-cluster

```

## Step-03: Install `eksctl` CLI

- For eksctl installation process, refer to this official documentation link: https://eksctl.io/installation/

## Step-04: Create an Amazon EKS cluster & Node groups

### 4.1 Create EKS cluster using eksctl

```
eksctl create cluster --name=labekscluster --region=us-east-1 --zones=us-east-1a,us-east-1b --without-nodegroup

# Get List of clusters
eksctl get cluster
```

:warning: **The cluster creation process can take around 10 to 15 mins to complete.**

### 4.2 Create EC2 Keypair for Nodegroup EC2 instances

- Create a new EC2 Keypair with name as `eks-nodes-keypair`
- To create a new kaypair, you can use EC2 service
  - AWS Management console >> EC2 >> Key Pairs >> New Key
    - Name: eks-nodes-keypair

### 4.3 Create an `EKS Node Group` with additional add-ons in public subnets

```
# Create public Node Group

eksctl create nodegroup --cluster=labekscluster --region=us-east-1 --name=eksdemo1-ng-public1 --node-type=t3.small --nodes=2 --nodes-min=2 --nodes-max=4 --node-volume-size=20 --ssh-access --ssh-public-key=eks-nodes-keypair --managed --asg-access --external-dns-access --full-ecr-access --appmesh-access --alb-ingress-access
```

## Step-05: Verify EKS Cluster & Worker nodes

### 5.1 Verify NodeGroup subnets | Make sure EC2 instances are in public subnet

- Go to **Services** -> **EKS** -> **eksdemo** -> **eksdemo1-ng1-public**
- Click on **Associated subnet** in Details tab
- Click on **Route Table** Tab.
- We should see that internet route via Internet Gateway (0.0.0.0/0 -> igw-xxxxxxxx)

### 5.2 Verify EKS Cluster and NodeGroup from AWS management console

- Navigate to **Services** -> **Elastic Kubernetes Service** -> **<eks_cluster_name>**

### 5.3 List Worker nodes

```
# List EKS clusters
eksctl get cluster

# List NodeGroups in a cluster
eksctl get nodegroup --cluster=<clusterName>

# List Nodes in current kubernetes cluster
kubectl get nodes -o wide

# Our kubectl context should be automatically changed to new cluster
kubectl config view --minify
```
