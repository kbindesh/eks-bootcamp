# Securing and Accessing Amazon EKS cluster

## 01. Key Concepts

- **K8s Authentication process**
- **AWS IAM service** (identity provider) - User, Group, Role
- **RBAC** (role based access control)
- **Subjects** (users, groups, service account)
- **Operations** (list, get, create, update, delete, watch, patch, post, put)
- **Resources** (pod, deployment, secrets, service, configmap etc)
- **kubeconfig**
- **aws-auth configMap**
- **OIDC** (OpenID Connect) identity provider

## 02. EKS Authentication method (default)

- EKS uses **AWS IAM** service for managing users, groups, and certificates for one or more K8s clusters.
- IAM is a global service
- IAM allows you to create users and groups, as well as assign AWS permissions including the AWS EKS API.
- By default, whoever creates an EKS cluster, is automatically granted
  _system:masters_ role (permissions), effectively making it the system administrator role for the EKS cluster

## 03. Lab - Managing Users & RBAC in EKS

In this lab, you will use AWS IAM users to enable them accessing EKS cluster as _cluster admin_ and _cluster viewer_ respectively.

### Step-3.1: Create a `cluster Admin` user for EKS

The _cluster admin_ user will have all administrative privileges across your EKS cluster.

- **Create an IAM User**

  - **AWS Management console** >> **IAM** >> **Users** >> **Create** >>
    - **Name**: k8s-cluster-admin
    - **Permissions**: Don't assign any policy for now >> Create

- **Generate the access keys for the above create user and download it**
  [We will need these access keys in the later steps, so have it handy]

- **Connect to the EKS cluster**

  - Connect to your EKS cluster and run the following commands:

  ```
  kubectl get configmap -n kube-system

  kubectl get configmap -n kube-system aws-auth -o yaml > aws-auth-configmap.yml
  [preceding command will have info about role & mapUsers info]

  vi aws-auth-configmap.yml

  [Scroll to mapUsers section and add the following]
  - userarn: arn:aws:iam::<account-id>:user:k8s-cluster-admin
    username: k8s-cluster-admin
    groups:
      - system:masters

  [Save & exit from the file]

  # Apply the updated aws-auth-configmap file changes
  kubectl apply -f aws-auth-configmap.yml
  ```

**INFO**: You may refer to the sample aws-auth file here [aws-auth-configmap.yml](./aws-auth-configmap.yml)

- **Add the IAM user credentials to the .aws credentials file**

  - %HOMEPATH%/.aws/credentials >> Open in notepad
  - Add credentials for k8s-cluster-admin user in a new profile

  ```
  [clusteradmin]
  aws_access_key_id=
  aws_secret_access_key=
  region=us-east-1
  output=json
  ```

- **Check the current IAM user credentials being used by kubectl**

  ```
  aws sts get-caller-identity

  # Set the AWS_PROFILE environment variable to prod-reader user
  $env:AWS_PROFILE="clusteradmin"

  # After updating, again check the current IAM user creds being used
  aws sts get-caller-identity
  ```

- **Test the setup**

```
kubectl get pods -n kube-system

[Try to access anything in the EKS cluster and it should allow you to do that]
```

### Step-3.2: Create a `production viewer` (read-only) user for EKS

The **production viewer** user will have a read-only access to only _production_ namespace and NO access to any other resource in your EKS cluster.

- **Create a Namespace**

```
kubectl create namespace production
```

- **Create an IAM User**

  - **AWS Management console** >> **IAM** >> **Users** >> **Create** >>
    - **Name**: prod-reader
    - **Permissions**: Don't assign any policy for now >> Create

- **Generate the access keys for the above create user and download it**
  [We will need these access keys in the later steps, so have it handy]

- **Create a `Role` (k8s) & `Role Binding` for prod-reader user**

  - For Role, refer to the [role-node-reader.yml](./manifests/role-prod-reader.yml)
  - For RoleBinding, refer to the [rolebinding-prod-reader.yml](./manifests/rolebinding-prod-reader.yml)

```

# Execute both the manifests to create a role & rolebinding for prod-user

kubectl apply -f role-node-reader.yml

kubectl apply -f rolebinding-prod-reader.yml
```

- **Update the `aws-auth configmap`**

```
kubectl -n kube-system get configmap aws-auth -o yaml > aws-auth-configmap.yml

vi aws-auth-configmap.yml


[You may refer to the attached sample aws-auth file - aws-auth-configmap.yml]

[Scroll all the way to the "mapUsers" section and following snippet. If not present, create it]

mapUsers: |
  - userarn: arn:aws:iam::<account-id>:user:prod-reader
    username: prod-reader
    groups:
      - prod-reader-role

[Save & exit from the file]

# Apply the updated aws-auth-configmap file changes
kubectl apply -f aws-auth-configmap.yml
```

- **Add the IAM user credentials to the .aws credentials file**

  - %HOMEPATH%/.aws/credentials >> Open in notepad
  - Add credentials for prod-reader user in a new profile

  ```
  [productionreader]
  aws_access_key_id=<access_key_here>
  aws_secret_access_key=<secret_key_here>
  region=us-east-1
  output=json
  ```

  - **Check the current IAM user credentials being used**

  ```
  aws sts get-caller-identity

  [The preceding command would display the details of your IAM user being used for accessing EKS cluster; if it is not prod-reader, follow next command]

  # Set the AWS_PROFILE environment variable to prod-reader user
  $env:AWS_PROFILE="productionreader"

  # After updating, again check the current IAM user creds being used
  aws sts get-caller-identity

  [This time you will see prod-reader user in the command output]
  ```

- **Test the setup**

```
kubectl get pods

kubectl get pods -n kube-system

[You should get an error as a prod-reader you can only read resources of production namespace]

kubectl get pods -n production
[This command will succeed]
```

## 04. Container Image Security

## 05. EKS Worker Node security

## 06. Container Runtime security
