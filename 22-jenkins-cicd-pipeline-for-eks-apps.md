# Jenkins pipeline for Application deployment on Amazon EKS Cluster

## Step-01: Overview

This article will demonstrate how to create a Jenkins based CI/CD pipeline to develop, package, ship and deploy application on Amazon EKS cluster.

## Step-02: Pre-requisites

- An AWS Account with admin privileges (https://console.aws.amazon.com/)
- A GitHub Account (https://github.com/)
- A DockerHub Account (https://hub.docker.com/)

## Step-03: Setup `K8s Management Server` on ec2 instance

### 3.1 Create an EC2 instance - Management Server

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

### 3.2 Install `kubectl` on Config server

- Ref: *https://docs.aws.amazon.com/eks/latest/userguide/install-kubectl.html*

```
# Download the kubectl binary
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"

# Assign execute permission to the kubectl utility
chmod +x kubectl

# Move the kubectl utility to /usr/local/bin
mv kubectl /usr/local/bin
```

### 3.3 Install `eksctl` on Config server

- Reference: *https://eksctl.io/installation/*

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

### 3.4 Install `Git` on Config server

```
sudo yum install -y git

# Check git version
git --version
```

### Install/Upgrade `AWS CLI`

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

[Recommended: Restart the server to bring new changes]
```

## Step-04: Setup `Jenkins Server`

### 4.1 Create an EC2 Instance - Jenkins Server

- AWS Management Console >> **EC2** >> Launch Instances
  - **Name**: config-server
  - **AMI**: Amazon Linux 2
  - **Instance Type**: t3.small
  - **Network Settings**
    - VPC: Default
    - Subnet: Default
    - Public IP: Enabled
    - Security Group: Allow Ingress traffic on port 22, 8080
  - **Storage settings**
    - Root Volume size: 15GB (min)

### 4.2 Install and Configure `Jenkins`

- [Installing and Configuring Jenkins Server](https://github.com/kbindesh/jenkins-masterclass/tree/main/Module-03_Setting_up_Jenkins/01-jenkins-on-amazon-linux)

### 4.3 Install `kubectl`

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

### 4.4 Install/Upgrade `AWS CLI`

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

### 4.5 Install and Configure Docker

```
# Install Docker
sudo yum install -y docker

# Start docker service
sudo systemctl start docker

# Enable docker service
sudo systemctl enable docker
```

### 4.6 Verify all the installed packages

```
java --version

jenkins --version

git --version

aws --version

docker --version

kubectl version --client

eksctl version
```

### 4.7 Elevate access levels of `jenkins` user to run `docker` and `kubectl` commands

```
# Add jenkins user to the docker group
sudo usermod -aG docker jenkins

# Add the jenkins user to the sudoers file
visudo

[Press "G" to go to the end of the file and press "i" to go in insert mode]

# Allow root to run any command anywhere
root    ALL=(ALL)     ALL
jenkins ALL=(ALL)     NOPASSWD: ALL

# Restart the Jenkins service
sudo systemctl restart jenkins
OR
sudo service jenkins restart

# Create a new directory for kubeconfig file
mkdir /var/lib/jenkins/.kube

[IMP: Restart the Jenkins server and now you as a ec2-user & jenkins user should be able to run docker commands]
```

## Step-05: Install `Jenkins Plugins`

- Install the following Jenkins plugins:
  - _Pipeline: Stage view_
  - _Pipeline_
  - _Docker Pipeline_
  - _Pipeline: AWS Steps_

## Step-06: Generate DockerHub Account Security Token and save it on Jenkins server

- Sign-in to you DockerHub account, https://hub.docker.com/ >> Sign-in
- Click on user dropdown list (top-right corner) >> Account settings >> **Personal access tokens** >> **Generate new token**
  - **Description**: Jenkins integration
  - **Expiration Date**: None
  - **Access Permission**: Read & Write
    (Read & Write tokens allow you to push images to any repository managed by your account)

```
# Sample output on generating PAT
To use the access token from your Docker CLI client:

1. User name will be your account name i.e kbindesh

docker login -u kbindesh

2. At the password prompt, enter the personal access token.

dckr_pat_o1B1VJf_3fBS7cKWsdfdfgm-Lxl0
```

## Step-07: Create an `AWS IAM Policies & Role` for Jenkins and K8s Mgmt Servers

- This reference doc link describes the minimum IAM policies needed to run the main use cases of eksctl: </br> *https://eksctl.io/usage/minimum-iam-policies/*

### 7.1 Create an IAM policy - `EksAllAccess`

- AWS Management console >> IAM >> Policies >> Create Policy
- Open JSON Editor (click on JSON button).
- Copy the **EksAllAccess** IAM Policy from [here](./iam-policies/EksAllAccess.json) and paste it in the policy JSON editor.
  - _IMP_: Replace the existing AccountID with your AWS AccountID
- **Policy Name**: EksAllAccess
- Click on **Create Policy** button

### 7.2 Create an IAM policy - `IamLimitedAccess`

- AWS Management console >> IAM >> Policies >> Create Policy
- Open JSON Editor (click on JSON button).
- Copy the **IamLimitedAccess** IAM Policy from [here](./iam-policies/IamLimitedAccess.json) and paste it in the policy JSON editor.
  - _IMP_: Replace the existing AccountID with your AWS AccountID
- **Policy name**: IamLimitedAccess
- Click on **Create Policy** button

### 7.3 Create an AWS IAM Role for Config server - `SetupEKSClusterRole`

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

## Step-08: Assign IAM role to Jenkins and K8s Management Server

- AWS Management console >> EC2 >> Select the Jenkins Server (EC2 Instance)
- Click on **Actions** menu >> **Security** >> **Modify IAM Role** >> Select _SetupEKSClusterRole_ role we created in the previous step.

## Step-09: Create an `Amazon EKS Cluster` using `eksctl` from K8s Management Server

- SSH to K8s Management Server and Run the following commands:

```
eksctl create cluster --name=labekscluster --region=us-east-1 --zones=us-east-1a,us-east-1b --without-nodegroup --kubeconfig=/var/lib/jenkins/.kube/config

# Get List of clusters
eksctl get cluster

# Change the ownership of "kubeconfig" file to jenkins user
sudo chown -R jenkins:jenkins /var/lib/jenkins/.kube
```

:INFO: The cluster creation process can take around 10 to 15 minutes to complete.

### 9.1 Create an EC2 Keypair for Nodegroup EC2 instances (worker nodes)

- You can use any existing keypair or create a new one. In case you already have a keypair, you can skip this step and continue with the next one.

- To create a new EC2 KeyPair, navigate to EC2 Dashboard >> _Network & Security_ section >> _Key Pairs_ >> **Create Key Pair**
  - **Name**: binWinWebServerKey
  - **Key pair type**: RSA
  - **Private key file format**: .pem

### 9.2 Associate IAM OIDC Provider to EKS Cluster

```
eksctl utils associate-iam-oidc-provider --region us-east-1 --cluster labekscluster --approve
```

### 9.3 Create EKS Node Group with additional add-ons

```
eksctl create nodegroup --cluster=labekscluster --region=us-east-1 --name=eksdemo1-ng-public1 --node-type=t3.small --nodes=1 --nodes-min=1 --nodes-max=4 --node-volume-size=20 --ssh-access --ssh-public-key=binWinWebServerKey --managed --asg-access --external-dns-access --full-ecr-access --appmesh-access --alb-ingress-access
```

### 9.4 Verify the created EKS Resources

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

## Step-10: Develop an `Application`, `Dockerfile` & `Jenkinsfile`

### 10.1 Develop an App (source code)

- You can clone this sample app repo: https://github.com/kbindesh/docker-sample-app

```
git clone https://github.com/kbindesh/docker-sample-app.git
```

### 10.2 Create a 'Dockerfile' (for packaging app in Docker Images)

```
FROM node:18-alpine
RUN mkdir -p /usr/src/app
COPY ./app/* /usr/src/app
WORKDIR /usr/src/app
RUN npm install
CMD node /usr/src/app/index.js
```

### 10.3 Create a `Jenkinsfile` (for Jenkins pipeline)

- You may refer to this [Jenkinsfile](./manifests/Jenkinsfile)

## Step-11: Create `Kubernetes manifests` to deploy

## Step-12: Create a new GitHub Repo and check-in the code

## Step-13: Create and Run a Jenkins Pipeline job

- **Name**: eks-app-deployment
- **Type**: Pipeline
- Description: This pipeline is for building docker image of a sample app and deploying it on EKS cluster.
- Trigger: GitHub hook trigger for GITScm polling
- Pipeline
  - Definition: Pipeline script from SCM
  - SCM: Git
  - Repository URL: <your_github_repo_url>
  - Branches to build: <your_github_repo_branch>
  - Script path: Jenkinsfile

## Step-14: Verify the published Docker Image from DockerHub registry and Deployed app on EKS cluster

## Step-15: (Optional) Connect kubectl to an EKS cluster from Jenkins Server (create a kubeconfig file)

- Reference: Ref: https://docs.aws.amazon.com/eks/latest/userguide/create-kubeconfig.html

```
aws sts get-caller-identity

aws eks update-kubeconfig --name labekscluster --region us-east-1

cat ~/.kube/config
```

## Step-16: (Optional) Create a Credential for AWS CLI (which will be used by eksctl)
