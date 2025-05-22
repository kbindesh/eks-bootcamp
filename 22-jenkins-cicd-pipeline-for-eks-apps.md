# Project: Jenkins pipeline for Application deployment on Amazon EKS Cluster

## XX. Overview

## XX. Pre-requisites

## XX. Setup `K8s Management Server` on ec2 instance

### Setup `Config Server` on EC2 Instance

- AWS Management Console >> EC2 >> Launch Instances
- Name: `config-server`
- AMI: Amazon Linux 2
- Instance Type: t2.micro
- Network Settings
  - VPC: Default
  - Subnet: Default
  - Public IP: Enabled
  - Security Group: Allow Ingress traffic on port 22
- Storage settings
  - Root Volume size: 15GB

### Install `kubectl` on Config server

- Ref: *https://docs.aws.amazon.com/eks/latest/userguide/install-kubectl.html*

```
# Download the kubectl binary
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"

# Assign execute permission to the kubectl utility
chmod +x kubectl

# Move the kubectl utility to /usr/local/bin
mv kubectl /usr/local/bin
```

### Install `eksctl` on Config server

- *https://eksctl.io/installation/*

```
ARCH=amd64
PLATFORM=$(uname -s)_$ARCH

# Download the eksctl binary
curl -sLO "https://github.com/eksctl-io/eksctl/releases/latest/download/eksctl_$PLATFORM.tar.gz"

# (Optional) Get the eksctl checksum
curl -sL "https://github.com/eksctl-io/eksctl/releases/latest/download/eksctl_checksums.txt" | grep $PLATFORM | sha256sum --check

# Extract the downloaded tarball
tar -xzf eksctl_$PLATFORM.tar.gz -C /tmp && rm eksctl_$PLATFORM.tar.gz

# Move the eksctl utility to /usr/local/bin
sudo mv /tmp/eksctl /usr/local/bin
```

### Install `Git` on Config server

```
sudo yum install -y git
```

### Install `AWS CLI`

- Official Documentation: https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html

```
# Remove any previous version of AWS CLI
sudo yum remove -y awscli

# Download the latest AWS CLI
sudo curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"

# Extract the downloaded .zip file
sudo unzip awscliv2.zip

# Install AWS CLI
sudo ./aws/install
```

## XX. Setup `Jenkins Server`

### Create an EC2 Instance

- AWS Management Console >> EC2 >> Launch Instances
  - Name: config-server
  - AMI: Amazon Linux 2
  - Instance Type: t3.small
  - Network Settings
    - VPC: Default
    - Subnet: Default
    - Public IP: Enabled
    - Security Group: Allow Ingress traffic on port 22, 8080
  - Storage settings
    - Root Volume size: 15GB (min)

### Install and Configure `Jenkins`

- [Installing and Configuring Jenkins Server](https://github.com/kbindesh/jenkins-masterclass/tree/main/Module-03_Setting_up_Jenkins/01-jenkins-on-amazon-linux)

### Install `kubectl`

- Official Documentation: https://docs.aws.amazon.com/eks/latest/userguide/install-kubectl.html

```
# Download the kubectl binary
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"

# Assign execute permission to the kubectl utility
chmod +x kubectl

# Move the kubectl utility to /usr/local/bin
mv kubectl /usr/local/bin

# To verify, simply get the kubectl version
kubectl version
```

### Install `AWS CLI`

- Official Documentation: https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html

```
# Remove any previous version of AWS CLI
sudo yum remove -y awscli

# Download the latest AWS CLI
sudo curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"

# Extract the downloaded .zip file
sudo unzip awscliv2.zip

# Install AWS CLI
sudo ./aws/install
```

## XX. Create an `AWS IAM Policies & Role` for Jenkins and K8s Mgmt Servers

- This reference doc link describes the minimum IAM policies needed to run the main use cases of eksctl: </br> *https://eksctl.io/usage/minimum-iam-policies/*

### Create an IAM policy - `EksAllAccess`

- AWS Management console >> IAM >> Policies >> Create Policy
- Open JSON Editor (click on JSON button).
- Copy the **EksAllAccess** IAM Policy from [here](./iam-policies/EksAllAccess.json) and paste it in the policy JSON editor.
  - _IMP_: Replace the existing AccountID with your AWS AccountID
- **Policy Name**: EksAllAccess
- Click on **Create Policy** button

### Create an IAM policy - `IamLimitedAccess`

- AWS Management console >> IAM >> Policies >> Create Policy
- Open JSON Editor (click on JSON button).
- Copy the **IamLimitedAccess** IAM Policy from [here](./iam-policies/IamLimitedAccess.json) and paste it in the policy JSON editor.
  - _IMP_: Replace the existing AccountID with your AWS AccountID
- **Policy name**: IamLimitedAccess
- Click on **Create Policy** button

### Create an AWS IAM Role for Config server - `SetupEKSClusterRole`

- Create an IAM Role for Config server with access to following services:

  - IAM
  - EC2
  - CloudFormation
  - EKS
  - ECR
  - VPC

- AWS Management console >> IAM >> Roles >> Create Role.
- **Trusted entity type**: AWS service
  - Service or Use case: EC2
- **Add Permissions**
  - Select following `IAM policies` from the list:
    - **AmazonEC2FullAccess**
    - **AWSCloudFormationFullAccess**
    - **EksAllAccess**
    - **IamLimitedAccess**
- **Name, Review & Create**
  - **Role Name**: `SetupEKSClusterRole`
  - **Description**: This IAM role is responsible for setting-up EKS cluster from config server.

## XX. Assign IAM role to Jenkins and K8s Management Server

- AWS Management console >> EC2 >> Select the Jenkins Server (EC2 Instance)
- Click on **Actions** menu >> **Security** >> **Modify IAM Role** >> Select _SetupEKSClusterRole_ role we created in the previous step.

## XX. Create an `Amazon EKS Cluster` using `eksctl` from K8s Management Server

- SSH to K8s Management Server and Run the following commands:

```
eksctl create cluster --name=labekscluster --region=us-east-1 --zones=us-east-1a,us-east-1b --without-nodegroup

# Get List of clusters
eksctl get cluster
```

:warning: The cluster creation process can take around 10 to 15 mins to complete.

### Create an EC2 Keypair for Nodegroup EC2 instances (worker nodes)

- You can use any existing keypair or create a new one. In case you already have a keypair, you can skip this step and continue with the next one.

- To create a new EC2 KeyPair, navigate to EC2 Dashboard >> _Network & Security_ section >> _Key Pairs_ >> **Create Key Pair**
  - **Name**: binWinWebServerKey
  - **Key pair type**: RSA
  - **Private key file format**: .pem

### Associate IAM OIDC Provider to EKS Cluster

```
eksctl utils associate-iam-oidc-provider --region us-east-1 --cluster labekscluster --approve
```

### Create EKS Node Group with additional add-ons

```
eksctl create nodegroup --cluster=labekscluster --region=us-east-1 --name=eksdemo1-ng-public1 --node-type=t3.small --nodes=1 --nodes-min=1 --nodes-max=4 --node-volume-size=20 --ssh-access --ssh-public-key=binWinWebServerKey --managed --asg-access --external-dns-access --full-ecr-access --appmesh-access --alb-ingress-access
```

### Verify the created EKS Resources

- **Verify NodeGroup subnets | Make sure EC2 instances are in public subnet**

  - Go to AWS Management Console >> **Services** -> **EKS** -> **eksdemo** -> **eksdemo1-ng1-public**
  - Click on **Associated subnet** in Details tab
  - Click on **Route Table** Tab.
  - We should see that internet route via Internet Gateway (0.0.0.0/0 -> igw-xxxxxxxx)

- **Verify EKS Cluster and NodeGroup from AWS management console**

  - Navigate to **Services** -> **Elastic Kubernetes Service** -> **<eks_cluster_name>**

- **List Worker nodes**

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

## XX. Connect kubectl to an EKS cluster from Jenkins Server (create a kubeconfig file)

- Reference: Ref: https://docs.aws.amazon.com/eks/latest/userguide/create-kubeconfig.html

```
aws sts get-caller-identity

aws eks update-kubeconfig --name labekscluster --region us-east-1

cat ~/.kube/config
```

## XX. Install `Jenkins Plugins`

- Install the following Jenkins plugins:
  - Pipeline: Stage view
  - Pipeline
  - Docker Pipeline

## XX. Develop Application (source code)

## XX. Generate DockerHub Account Security Token and save it on Jenkins server

## XX. Create a 'Dockerfile' (for packaging app in Docker Images)

## XX. Create a `Jenkinsfile` (for Jenkins pipeline)

## XX. Create `Kubernetes manifests`

## XX. Create a new GitHub Repo and check-in the App code

## XX. Create a Credential for AWS CLI (which will be used by eksctl)
