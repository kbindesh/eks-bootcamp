# Kubernetes RABC with Service Account

## 01. What is a Service Account?

- A Service Account provides an identity for a process that runs in a pod.
- Service Accounts are used by apps and processes to authenticate as they communicate with the kube-apiServer.
- These Service Accounts can be restricted by K8s RBAC.
- This allows you to authenticate and restrict access and what that process or apps can do.

## XX. `Namespace-wide RBAC Policies` for `ServiceAccounts`

### Check if RBAC is installed

```
kubectl api-versions | grep rbac

[If you see rbac.authorization.k8s.io/v1 in the results, it is installed]
```

### Create a new Namespace

- **Create a new k8s manifest _dev-namespace.yml_ for creating a _development_ namespace**

```
apiVersion: v1
kind: Namespace
metadata:
   name: development
```

- **Execute the above k8s manifest**

```
kubectl apply -f dev-namespace.yml

# List all the namespaces
kubectl get ns
```

### Create a Service Account (in custom namespace)

- **Create a new k8s manifest _webapp-serviceaccount.yml_ for creating a service account _webapp-serviceaccount_ in _development_ namespace**

```
apiVersion: v1
kind: ServiceAccount
metadata:
  name: webapp-serviceaccount
  namespace: development
```

- **Execute the above k8s manifest**

```
kubectl apply -f webapp-serviceaccount.yml

# List all the service accounts
kubectl get serviceaccounts -n development
OR
kubectl get sa -n development
```

- **Describe above created service account _webapp-serviceaccount_ and observe the _Tokens_ attribute**

```
# Describe serviceaccount
# Token is stored as a secret, so it can be viewed with token name
kubectl describe serviceaccount webapp-serviceaccount -n development

[Get the Tokens value (k8s secret name) from the preceding command's output]


# Describe the secret which has above serviceaccount's token
kubectl describe secret <tokens_value_from_last_cmd> -n development
```

### Create Role (in custom namespace)

- **Create a new k8s manifest _pod-reader-role.yml_ for creating a role in _development_ namespace**

```
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: development
  name: pod-reader
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "watch", "list"]
```

- **Execute the above k8s manifest _pod-reader-role.yml_**

```
kubectl apply -f pod-reader-role.yml

# List all the roles present in 'development' namespace
kubectl get roles -n development

[You should see a new role pod-reader]

# Describe the above role to check assigned permissions
kubectl describe role pod-reader -n development
```

### Create RoleBinding (in custom namespace)

- This RoleBinding will assign **serviceAccount** (webapp-serviceaccount) and **Role** (pod-reader).
- So, the _ServiceAccount_ will ne able to list, get, watch pods in the development namespace.

```
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: pod-reader-rolebinding
  namespace: development
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: pod-reader
subjects:
- kind: ServiceAccount
  name: webapp-serviceaccount
  namespace: development
```

- **Execute the above k8s manifest _pod-reader-rolebinding.yml_**

```
kubectl apply -f pod-reader-rolebinding.yml

# List all the roles present in 'development' namespace
kubectl get rolebindings -n development

[You should see a new rolebinding 'pod-reader-rolebinding' in the list]

# Describe the above role to check assigned permissions
kubectl describe rolebinding pod-reader-rolebinding -n development
```

### Test permissions of Service Account using Pod

- To test the permissions that assigned to the Service Account, I'm creating a new custom pod with kubectl utility inside it.
- Now, we will create a Pod and assign a serviceAccount to it. Then, get inside the Pod and try to access Pod API and other APIs.

```
apiVersion: v1
kind: Pod
metadata:
  name: pod-with-kubectl
  namespace: development
spec:
  containers:
  - image: bibinwilson/docker-kubectl:latest
    name: kubectl
  serviceAccountName: webapp-serviceaccount
```

- **Execute the above k8s manifest _pod-with-serviceaccount.yml_**

```
kubectl apply -f pod-with-serviceaccount.yml

# List all the Pods present in Development namespace
kubectl get pods -n development
```

- **Connect to the above created Pod and run kubectl commands to make a callout to diff APIs**:

```
# Connect to the pod
kubectl exec -it --namespace=development pod-with-kubectl -- /bin/bash

[The preceding command will end you up inside the pod container shell]

# Test-01: Check if you're able to list Pods from 'development' namespace
kubectl get pods -n development

[The preceding command will work | will list all the pods present in development namespace]

# Test-02: Check if you're able to list Pods from the 'default' namespace
kubectl get namespaces

[The last command will result in error: Error from server (Forbidden) because pod is not allowed to access default ns]

# Test-03: Check if you're able to list services from the 'default' namespace
kubectl get svc

[The last command will also result in error: Error from server (Forbidden) because pod is not allowed to access service API]

```

## XX. Cluster-wide RBAC Policies
