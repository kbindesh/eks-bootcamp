# Kubernetes Cheatsheet

This is a quick reference to most often used key kubernetes concepts and the associated kubectl commands.

## Required utilities

- Kubernetes cluster
- kubectl - k8s client

## 01. Common `kubectl` options

### View API document of any Kubernetes API object

```
kubectl explain <api_object>

# Example
kubectl explain pods
kubectl explain pods --recursive
kubectl explain deployments
```

### Getting help around any kubectl command

```
kubectl -h
OR
kubectl --help

# Examples
kubectl --help
kubectl create --help
kubectl apply --help
```

### Set an alias for `kubectl`

```
# Set an alias
alias k=kubectl
echo 'alias k=kubectl' >>~/.bashrc
```

### Render the `kubectl` command output in various formats

```
# render in json format
kubectl get <object> -o json

# render in yaml format
kubectl get <object> -o yaml

# render in results in plain-text format with any additional information
kubectl get <object> -o wide

# Examples
kubectl get pods -o json
kubectl get pods -o yaml
kubectl get pods -o wide
```

### alias for `namespace` (-n)

```
kubectl get <object> -n <namespace>

# Example
kubectl get pods -n production
```

### get the filtered list of k8s resources based on labels (-l)

```
# syntax
kubectl get <object> -l <label_key>=<label_value>

# Examples
kubectl get pods -l app=webapp
kubectl logs -l apptype=frontend
```

### Stream resource changes in real time (--watch)

```
# Example
kubectl get deployment my-deploy --watch
```

### Force any actions on k8s resources (--force)

```
# Example
kubectl delete pod bin-pod --force
```

## 02. K8s Cluster Management Commands

## XX. K8s `Nodes` Commands

## XX. K8s manifests (configuration files) Commands

```
# Create K8s resource
kubectl create -f <k8s_manifest_filename>.yaml
OR
kubectl create -f <k8s_manifest_filename>.yaml

# Create k8s resources in all the manifest files present in a directory
kubectl create -f <k8s_manifests_directory>

# Create k8s resources from a remote URL
kubectl create -f <remote_URL>

# Retrieve the status of the resources associated with a k8s manifest
kubectl get -f <k8s_manifest_filename>.yaml

# Show differences between the live cluster state and the config file
kubectl diff -f <k8s_manifest_filename>.yaml

# Delete the resources associated with a k8s manifest
kubectl delete -f <k8s_manifest_filename>.yaml
```

## XX. K8s `Namespaces` Commands

- A K8s namespace is a logical partition within a cluster that isolates groups of resources for organizational or access control purposes.

<table style="width:100%">
  <tr>
    <th>Resource</th>
    <th>API Kind</th>
    <th>Namespace Scoped</th>
    <th>Command</th>
    <th>Description</th>
  </tr>
  <tr>
    <td rowspan="10">Namespace</td>
  </tr>
   <tr>
    <td rowspan="10">Namespace</td>
  </tr>
 <tr>
    <td rowspan="10">True</td>
  </tr>
 <tr>
    <td>kubectl create namespace NAMESPACE_NAME</td>
    <td>Create a Namespace</td>
  </tr>
 <tr>
    <td>kubectl get namespaces</td>
    <td>List all the existing Namespaces</td>
  </tr>
 <tr>
    <td>kubectl describe namespace NAMESPACE_NAME</td>
    <td>Display detailed info about one or more namespaces</td>
  </tr>
 <tr>
    <td>kubectl delete namespace NAMESPACE_NAME</td>
    <td>Delete a Namespace</td>
  </tr>
 <tr>
    <td>kubectl edit namespace NAMESPACE_NAME</td>
    <td>Edit and live-update the definition of a namespace</td>
  </tr>    
 <tr>
    <td>kubectl top namespace NAMESPACE_NAME</td>
    <td>Display resource (cpu,memory,storage) usage for a namespace</td>
  </tr>    
 <tr>
    <td>kubectl api-resources --namespaced=true</td>
    <td>List all the K8s APIs which are namespace scoped</td>
  </tr>          
</table>

## XX. `LimitRange` and `ResourceQuota` Commands

## XX. K8s `Pods` Commands

## XX. K8s `replicaSets` Commands

## XX. K8s `Deployment` Commands

## XX. K8s `DaemonSets` Commands

## XX. K8s Storage services Commands (StorageClass, PVC, PV)

## XX. HorizontalPodAutoscaling Commands

## XX. K8s `Job` and `CronJob` Commands

## XX. K8s Security services Commands (role,roleBinding,clusterRole, clusterRoleBinding)

## XX. K8s `Events` Commands

## XX. K8s `Logs` Commands

## All the K8s APIs with namespace scope
