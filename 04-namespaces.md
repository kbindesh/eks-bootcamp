# Kubernetes Namespaces

## Overview

## List existing Namespaces

```
kubectl get namespaces
OR
kubectl get ns
```

## Create Namespaces - imperative way

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

## (Optional) Create Namespaces - Declarative way

- You may refer to the following manisfests to create two namespaces namely development and production respectively:
  - [manifests/dev-namespace.yml](./manifests/dev-namespace.yml)
  - [manifests/prod-namespace.yml](./manifests/prod-namespace.yml)

```
kubectl apply -f manifests/dev-namespace.yml

kubectl apply -f manifests/prod-namespace.yml
```

## Deploy an App in dev and production Namespaces

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

## List `Dev` and `Production` resources

```
# List dev pods
kubectl get pods -n development

# List production pods
kubectl get pods -n production
```
