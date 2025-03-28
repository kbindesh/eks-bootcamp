# Configuring Applications in K8s

## 01. ConfigMaps

- ConfigMap is stores configuration data in key-value pairs.
- ConfigMaps can be used to configure the app running in a container by configuring a pod to consume ConfigMaps using:
  - Environment variables
  - Command-line arguments
  - Mounting a volume with configuration files

### 1.1 Create a configMap

- Kindly refer [manifests/configMap.yml](./manifests/configMap.yml) file for a sample configMap

- Now create a configmap by executing the above configMap manifest file:

```
kubectl apply -f configMap.yml
```

### 1.2 List all the configMaps

```
kubectl get configmap
or
kubectl get cm
```

### 1.3 Describe a configMap

```
kubectl describe configMap <configmap_name>
```

### 1.4 `Consume configMap data` into a `Pod` using `Environment Variable`

- **Create a Pod**

  - Kindly refer [manifests/pod-consuming-configmap.yml](./manifests/pod-consuming-configmap.yml) file for a sample Pod consuming configMap created in the preceding steps via environment variable.

- You can use the following command to check the configmap value:

```
kubectl logs bin-configmap
```

### 1.5 `Consume configMap data` into a `Pod` using `Container Volumes`

- **Create a Pod**

  - Kindly refer [manifests/pod-consuming-configmap.yml](./manifests/pod-consuming-configmap-vol.yml) file for a sample Pod consuming configMap created in the preceding step using container volume.

- You can use the _kubectl logs_ command to check the pod for the mounted data value, or use the following command to check the configmap:

```
kubectl exec bin-volume-pod -- ls /etc/config
```

### 1.6 Delete a configMap

```
kubectl delete cm bin-configmap
```
