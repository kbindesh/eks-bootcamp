# Kubernetes Namespaces

In this section, following topics are demonstrated:

- Namespace Overview
- Listing Namespaces
- Creating Namespaces using both imperative & declarative ways
- Limit Ranges
- Resource Quotas

## Overview

## 01. List existing Namespaces

```
kubectl get namespaces
OR
kubectl get ns
```

## 02. Create Namespaces (imperative way)

```
# Create a dedicated namespace for development environment
kubectl create namespace dev

# Create a dedicated namespace for test environment
kubectl create namespace test

# Create a dedicated namespace for production environment
kubectl create namespace prod
```

- List Namespaces

```
kubectl get ns
```

## 03. (Optional) Create Namespaces (Declarative way)

- You may refer to the following manisfests to create two namespaces namely development and production respectively:
  - [manifests/dev-namespace.yml](./manifests/dev-namespace.yml)
  - [manifests/prod-namespace.yml](./manifests/prod-namespace.yml)

```
kubectl apply -f manifests/dev-namespace.yml

kubectl apply -f manifests/prod-namespace.yml
```

## 04. Deploy an App in dev and production Namespaces

- You can specify details for an application in two ways:

  1. **using `kubectl apply -n` option**

     ```
     # Deploy nginx app in production namespace using kubectl -n option
     kubectl -n production apply -f nginx-deployment.yml
     ```

  2. **using `namespace` attribute of the `metadata` element in your namespace manifest**

     ```
     # Notice metadata.namespace attribute

     apiVersion: apps/v1
     kind: Deployment
     metadata:
       name: nginx-deployment
       namespace: production
       labels:
         app: nginx
     ```

  - You may refer to the following manifests to deploy nginx app in production and development namespaces:

    ```
    kubectl apply -f [app-deploy-in-dev-ns.yml](./manifests/app-deploy-in-dev-ns.yml)

    kubectl apply -f [app-deploy-in-prod-ns.yml](./manifests/app-deploy-in-prod-ns.yml)
    ```

### List `Dev` and `Production` resources

```
# List dev pods
kubectl get pods -n development

# List production pods
kubectl get pods -n production
```

## 05. Kubernetes `Limit Ranges`

As an Administrator, you might be concerned about making sure that a single object cannot monopolize all available resources within a namespace.

- A **LimitRange** is a policy to constrain the resource allocations (limits and requests) that you can specify for each applicable object kind (such as _Pod_ or _PersistentVolumeClaim_) in a namespace.
- A **LimitRange** provides constraints that can:
  - _Enforce minimum and maximum compute resources usage per Pod or Container in a namespace_.
  - _Enforce minimum and maximum storage request per PersistentVolumeClaim in a namespace_.
  - _Enforce a ratio between request and limit for a resource in a namespace_.
  - _Set default request/limit for compute resources in a namespace and automatically inject them to Containers at runtime_.

### 5.1 Create a Namespace manifest

```
apiVersion: v1
kind: Namespace
metadata:
  name: development
```

### 5.2 Create a Limit Range for Namespaces

- You may refer to this manifest: [manifests/dev-namespace-with-limit-range.yml](./manifests/dev-namespace-with-limit-range.yml)

### 5.2 Update App K8s manifests with dev namespace

- Update the app deployment manifest with namespace details:
  - **namespace**: development
- Remove the resource specs from the deployment manifest (as we are imposing these specs from the limit range object).

- Final app deployment manifest would look something like this: [manifests/app-deployment-without-res-specs.yml](./manifests/app-deployment-without-res-specs.yml)

### 5.3 Deploy Application

```
kubectl apply -f app-deployment-without-res-specs.yml
```

### 5.4 Describe deployed application resource details

```
kubectl get pods -n development -w

kubectl get pod <pod_name> -o yaml -n development

# Get and describe limits of development namespace
kubectl get limits -n development

kubectl get limits dev-ns-resource-limit-range -n development
```

## 06. Kubernetes `Resource Quota`

### 6.1 Create a new `namespace`, `limit range` and `resource quota`

- You may refer to this manifest [manifests/dev-namespace-limit-and-res-quota.yml](./manifests/dev-namespace-limit-and-res-quota.yml) for creating the following resources:
  - Namespace: development
  - LimitRange: <capacity_for_indivisual_container>
  - ResourceQuota: <capacity_allocated_for_namespace>

```
kubectl apply -f
```

### 6.2 Test and describe deployed resources

```
# List pods of development namespace
kubectl get pods -n development -w

# View Pod Specification (CPU & Memory)
kubectl get pod <pod-name> -o yaml -n development

# Get & Describe Limits set
kubectl get limits -n development
kubectl describe limits dev-ns-resource-limit-range -n development

# Get Resource Quota
kubectl get quota -n development
kubectl describe quota dev-ns-resource-quota -n development
```

## References

- https://kubernetes.io/docs/concepts/policy/limit-range/
- https://kubernetes.io/docs/concepts/policy/resource-quotas/
