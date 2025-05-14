# Deploying Stateless Application using K8s Deployments

## Step-xx: Create a Deployment

## Step-xx: Perform scaling operations

## Step-xx: Perform a Rolling updates

## Step-xx: Perform a Rollback

## Step-xx: Verify the results using kubectl

## Step-xx: View the EKS resources from the AWS Management Console

### Necessary IAM Permissions to view the EKS resources

- To view EKS resources in the AWS Management Console, the IAM user or role needs the following IAM permissions (policies):

  - eks:DescribeCluster
  - eks:ListClusters
  - eks:ListTagsForResource
  - eks:DescribeNodegroup
  - eks:ListNodegroups
  - eks:DescribeFargateProfile
  - eks:ListFargateProfiles
  - eks:DescribeAddon
  - eks:ListAddons

- **Root User Permissions**:

  - While the root user has broad permissions, it's generally a bad practice to use it directly for EKS operations.
  - Instead, use IAM users or roles with specific permissions.

- Kubernetes API Access

- Example IAM policy

- Kubernetes RBAC Configuration

  - **aws-auth ConfigMap**
    - This ConfigMap maps IAM principals (users or roles) to kubernetes users and groups.
  - **Mapping IAM Principals**

    - You need to add an entry to the aws-auth ConfigMap that maps the IAM principal (the root user or the IAM user/role) to a k8s group with the necessary permissions.

    - Example aws-auth ConfigMap Entry:

    ```
      mapUsers:
      - groups:
        - system:masters
        userarn: arn:aws:iam::<your-account-id>:user/<your-iam-user-name>
        username: <your-kubernetes-username>
    ```

  - **Kubernetes Role and RoleBindings**
    - Create a Kubernetes Role or ClusterRole with the necessary permissions to view Kubernetes resources and then create a RoleBinding or clusterRoleBinding to bind that Role/ClusterRole to the group specified in the aws-auth ConfigMap.
