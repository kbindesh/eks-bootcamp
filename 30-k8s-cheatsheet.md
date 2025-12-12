# `kubectl` Cheatsheet

This is a quick reference to most often used key kubectl commands with it's description and examples.

## Prerequisites

- Kubernetes cluster
- kubectl - k8s client

## 01. Common `kubectl` options

<table style="width:100%">
  <tr>
    <th>Command</th>
    <th>Examples</th>
    <th>Description</th>
  </tr>
  <tr>
    <td>alias k=kubectl</td>
    <td>
        alias k=kubectl</br>
        echo 'alias k=kubectl' >>~/.bashrc
    </td>
    <td>Set an alias for kubectl</td>
  </tr>  
  <tr>
    <td>kubectl explain</br></td>
    <td>
        kubectl explain pods</br>
        kubectl explain pods --recursive</br>
        kubectl explain deployments
    </td>
    <td>View API document of any Kubernetes API object</td>
  </tr>
 <tr>
    <td>kubectl --help</br>OR</br>kubectl -h </td>
    <td>
      kubectl --help</br>
      kubectl create --help</br>
      kubectl apply --help
    </td>
    <td>Getting help around any kubectl command</td>
  </tr>
 <tr>
    <td>kubectl api-resources</td>
    <td>
      kubectl api-resources</br>
      kubectl api-resources -o wide</br>
      kubectl api-resources --namespaced=true</br>
      kubectl api-resources --namespaced=false</br>
      kubectl api-resources --namespaced=true --sort-by=name
      kubectl api-resources --namespaced=true --sort-by=kind
    </td>
    <td>List all the supporting kubernetes API-RESOURCES</td>
  </tr>  
  <tr>
    <td>kubectl get API_RESOURCE -o json</td>
    <td>
      kubectl get pods -o json</br>
      kubectl get namespaces -o json
    </td>
    <td>Render the kubectl command output in json format</td>
  </tr>
  <tr>
    <td>kubectl get API_RESOURCE -o yaml</td>
    <td>
      kubectl get pods -o yaml</br>
      kubectl get deployments -o yaml
    </td>
    <td>Render the kubectl command output in yaml format</td>
  </tr>  
  <tr>
    <td>kubectl get API_RESOURCE -o wide</td>
    <td>
      kubectl get nodes -o wide</br>
      kubectl get pods -o wide
    </td>
    <td>List k8s objects with additional information</td>
  </tr>
  <tr>
    <td>kubectl get API_RESOURCE -n <namespace></td>
    <td>
      kubectl get pods -n production</br>
      kubectl get roles -n production
    </td>
    <td>alias for namespace</td>
  </tr>  
  <tr>
    <td>kubectl get API_RESOURCE -l LABEL_KEY=LABEL_VALUE</td>
    <td>
      kubectl get pods -l app=webapp</br>
      kubectl logs -l apptype=frontend
    </td>
    <td>Get the filtered list of k8s resources based on labels (-l)</td>
  </tr>
  <tr>
    <td>kubectl get API_RESOURCE OBJECT_NAME --watch</td>
    <td>
      kubectl get deployment my-deployment --watch
    </td>
    <td>Stream resource changes in real time (--watch)</td>
  </tr>    
</table>

## 02. K8s Cluster & Context management Commands

<table style="width:100%">
  <tr>
    <th>Command</th>
    <th>Description</th>
    <th>Examples</th>
  </tr>
  <tr>
    <td>kubectl cluster-info</td>
    <td>
      Displays the addresses and status of the control plane components & running core services
    </td>
    <td>
      kubectl cluster-info</br>
      kubectl cluster-info dump</br>
      kubectl cluster-info --context=bin-cluster</br>
     </td>
  </tr>  
</table>

## XX. K8s manifests (configuration files) Commands

<table style="width:100%">
  <tr>
    <th>Command</th>
    <th>Description</th>
  </tr>
 <tr>
    <td>kubectl apply -f K8s_MANIFEST.yaml</br>
  </td>
    <td> Creates or updates the objects defined in the YAML file to match the desired state</td>
  </tr>  
 <tr>
    <td>kubectl create -f K8s_MANIFEST.yaml</br>
      OR</br>
      kubectl create -f K8s_MANIFEST.yaml</td>
    <td>Create a K8s object</td>
  </tr>
 <tr>
    <td>kubectl create -f MANIFEST_DIR_NAME</td>
    <td>Create k8s object associated with all the manifest files present in a directory</td>
  </tr>
 <tr>
    <td>kubectl create -f REMOTE_URL</td>
    <td>Create k8s object from a remote URL</td>
  </tr>
 <tr>
    <td>kubectl get -f K8s_MANIFEST.yaml</td>
    <td>Retrieve the status objects associated with a k8s manifest</td>
  </tr>
 <tr>
    <td>kubectl diff -f K8s_MANIFEST.yaml</td>
    <td>Show differences between the live cluster state and the config file</td>
  </tr>    
 <tr>
    <td>kubectl delete -f K8s_MANIFEST.yaml</td>
    <td>Delete objects associated with a k8s manifest</td>
  </tr>         
</table>

## XX. K8s `Nodes` Commands

- A K8s node can be a physical or virtual machine that runs K8s containerized workloads.
- Each node will have the following:
  1. **kubelet**: for communication with the control plane
  2. **Container runtime**: Like containerd or CRI-O to run containers
  3. **kube-proxy**: For managing k8s objects networking.

<table style="width:100%">
  <tr>
    <th>Command</th>
    <th>Description</th>
    <th>Examples</th>
  </tr>
  <tr>
    <td>kubectl get nodes</td>
    <td>
      List k8s cluster nodes
    </td>
    <td>
      kubectl get nodes</br>
      kubectl get nodes -o wide</br>
     </td>
  </tr>
  <tr>
    <td>kubectl delete nodes</td>
    <td>
      Delete a node or multiple nodes.
    </td>
    <td>
      kubectl delete node NODE_NAME
     </td>
  </tr>  
    
</table>

### `Expert's TIP`: Recommended practice for safely evict pods & removing a node from the k8s cluster is:

- Step-01: kubectl drain NODE_NAME
- Step-02: kubectl delete NODE_NAME

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
