# Helm - K8s Package Manager

- Official website: https://helm.sh/

## 01. Preparing a K8s and Helm environment

- In this section, we will outline the tools and concepts that are required in order to begin working with Helm.
- The following topics will be covered:
  - **Preparing a local Kubernetes environment with `Minikube`**
  - **Setting up `kubectl`**
  - **Setting up `Helm`**
  - **Configuring `Helm`**

## 02. Install Helm

- https://helm.sh >> Docs >> Introduction >> Installing Helm
- [As a pre-requisite, you can download and install chocolatey package manager] from https://chocolatey.org/install#individual
- https://docs.aws.amazon.com/eks/latest/userguide/helm.html

- **Install Helm on Windows**

```
# Install Helm on Windows with Chocolatey package manager
choco install kubernetes-helm
```

- **Install Helm on Linux**

```
# Install Helm using the binaries
curl https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3 > get_helm.sh

chmod 700 get_helm.sh

./get_helm.sh
```

- _NOTE_: If you get a message that openssl must first be installed, you can install it with the following command.

```
sudo yum install openssl
```

- **Check the version of Helm that you install**

```
helm version | cut -d + -f 1
```

- **Get Help**

```
helm --help
OR
helm -h
```

- **How does Helm know about the K8s cluster?**
  - Using the _.kube config_ file.
  - Helm uses the same _kube-config_ file that kubectl uses.
  - In case you want to use another config file, update the KUBECONFIG environment variable: _KUBECONFIG .kube/config_.

## 03. Helm Repo

### 3.1 Finding Charts

- Helm 3 comes with no default repository, but you can search the **Helm Hub** (https://artifacthub.io) or specific repositories.

```
# Search the Helm Hub for a specific repository
helm search hub | wc -l

[The preceding command would tell you the total no. of charts present in the Hub]

# Search the hub for a specific keyword like mariadb
helm search hub mariadb --max-col-width 60 | head -n 10
```

### 3.1 List Helm Repo

```
helm repo list
```

### 3.2 Add a Helm Repo

- You can either install charts directly from the _hub_ or do some research and add individual repositories:

-**Example-01**

```
# Syntax - Add helm repo
helm repo add <repo_name> <repo_url>

# For example, for Prometheus, there is the "prometheus-community" Helm repository
helm repo add prometheus-community https://prometheus-community.github.io/helmcharts

# Now, we can search the prometheus repo
helm search repo prometheus

# To get more info about a specific chart, we can use the "show" command or "inspectalias" command too
helm show chart prometheus-community/prometheus

# To get more info about a specific chart values (default)
helm show values prometheus-community/prometheus | wc -l
```

- **Example-02**

```
# Syntax - Add helm repo
helm repo add <repo_name> <repo_url>

# Example - Add helm repo
helm repo add bitnami https://charts.bitnami.com/bitnami
```

### 3.3 Search Chart in a Repo

```
# Search a apache helm chart in bitnami Repo
helm search repo apache

# Search a mysql helm chart in bitnami Repo
helm search repo mysql

# Search all the versions of a particular helm chart (e.g. mysql)
helm search repo mysql --versions
```

### 3.4 Remove a Helm Repo

```
# Syntax - Delete helm repo
helm repo remove <repo_name>

# Example - Delete helm repo
helm repo remove bitnami
```

## 04. Helm Charts

### 4.1 Install a helm chart

```
# Syntax
helm install <installation_alias> <helm_chart_name>

# Example-01
helm install mysqldb bitnami/mysql

# Example-02
helm install prometheus prometheus-community/prometheus -n monitoring --createnamespace

```

### 4.2 Install a helm chart in a particular namespace

```
# Syntax
helm install --namespace <ns_name> <installation_name> <helm_chart_name>
OR
helm install -n <ns_name> <installation_name> <helm_chart_name>

# Example
helm install --namespace dev_ns mysqldb bitnami/mysql
```

- **Note**: _installation_name_ must be unique within a k8s namespace

### 4.3 Check the status of Helm installation

```
# Syntax
helm status <installation_name>

# Example-01
helm status mysqldb

# Example-02
helm status -n monitoring prometheus | grep STATUS
```

### 4.4 List all the helm installations/releases

```
# List all the helm installations made in "default namespace"
helm list
OR
helm ls

# List all the helm installations made in a particular namespace
helm list --namespace <ns_name>

# List all the helm installations across all the namespaces
helm list -A

# you can list all the secrets that have the "owner=helm" label
kubectl get secret -A -l owner=helm
```

### 4.5 Helm install in dryrun mode (--dryrun)

```
helm install mysqldb bitnami/mysql --dry-run

helm upgrade mysqldb bitnami/mysql --dry-run
```

### 4.6 Uninstall a helm installation

```
# Uninstall an installation from a default namespace
helm uninstall mysqldb

# Uninstall an installation from a particular namespace
helm uninstall mysqldb -n <ns_name>

# Uninstall an installation, keeping revision history
helm uninstall mysqldb --keep-history
```

## 05. Passing custom configuration to K8s apps using Helm

1. using **--set** option
2. using **--values** option

#### 4.1 Pass custom config using `--set option`

```
helm install mysqldb bitnami/mysql --set auth.rootPassword = dbpassword123
```

#### 4.2 Pass custom config using `--values option`

- **Create values.yml file**

**values.yml**

```
auth:
  rootPassword: "dbpassword1234"
```

- **Deploy mysql app with custom config using Helm & values.yml**

- Pass the above created values.yml file as a parameter while installing mysql database chart

```
helm install mysqldb bitnami/mysql --values values.yml
```
