# Helm Basic Tutorial & Commands

## About

A Helm example project with resources from the LinkedIn Learning course [Kubernetes: Package Management with Helm](https://www.linkedin.com/learning/kubernetes-package-management-with-helm/), to get to know the Helm commands, the template language and how to publish a Kubernetes app with Helm.

## Install Helm

Prerequisites:

- An available Kubernetes cluster locally or on the cloud.
  - Most Common local Kubernetes cluster managers: [Kind](https://kind.sigs.k8s.io/docs/), [Minikube](https://minikube.sigs.k8s.io/docs/), ...
- The Kubernetes command-line `kubectl` to run commands against Kubernetes clusters.

### Install Helm on Linux

For most Debian-like distributions:

```bash
curl https://baltocdn.com/helm/signing.asc | sudo apt-key add -
sudo apt-get install apt-transport-https --yes
echo "deb https://baltocdn.com/helm/stable/debian/ all main" | sudo tee /etc/apt/sources.list.d/helm-stable-debian.list
sudo apt-get update
sudo apt-get install helm
```

> See [Links](#links) bellow for official helm install guide on all supported systems.

### Helm Completion

Add the Helm autocompletion for your shell for an easier use of the helm cli:

For Bash:

```bash
helm completion bash > /etc/bash_completion.d/helm
```

For Zsh:

```bash
helm completion zsh > "${fpath[1]}/_helm"
```

> See [Links](#links) bellow for official helm completion guide on all supported shells.

## Install a Helm chart from a repository

Install the `bitnami/kube-state-metrics` chart in our local Kubernetes cluster:

```bash
# Add a helm repo
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
helm repo list

# Creata a namespace
kubectl create ns metrics
kubectl get ns

# Install chart from repo in namespace
helm install kube-state-metrics bitnami/kube-state-metrics -n metrics
helm ls -n metrics

# Inspect the chart
kubectl get all -n metrics
kubectl logs -n metrics kube-state-metrics-6dc949474-gbknx

# Port Forward to expose service
kubectl port-forward -n metrics svc/kube-state-metrics --address 0.0.0.0 8090:8080
```

### The Helm Show command

Inspect a helm chart from a repo:

```bash
helm show chart bitnami/kube-state-metrics
```

Inspect the values of the helm chart:

```bash
# Save into a yaml file
helm show values bitnami/kube-state-metrics > values.yaml
```

### Update the Helm chart

Upgrade the chart version:

```bash
# View current chart and app versions
helm ls -n metrics

# Update chart to a specific version
helm upgrade -n metrics kube-state-metrics bitnami/kube-state-metrics --version 0.4.0
kubectl get all -n metrics
```

## Create a Helm chart

Create a boilerplate helm chart:

```bash
helm create first-chart
```

### Create a ConfigMap

A boilerplate ConfigMap in `templates/configmap.yaml`:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: first-chart-configmap
data:
  port: "8080"
```

#### Install a Helm chart

In the `first-chart` directory run:

```bash
helm install first-chart .
```

Monitor the changes:

```bash
kubectl get cm
kubectl describe cm first-chart-configmap
```

#### Update the Helm chart

If we make changes to the `first-chart`, we can update our cluster with:

```bash
# First check the chart files for the change
helm template first-chart .
# Apply the change to the cluster
helm upgrade first-chart .
```

Monitor the changes:

```bash
kubectl get cm
kubectl describe cm first-chart-configmap
```

### Create a Secret

A boilerplate Secret in `templates/secret.yaml`:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: first-secret
type: Opaque
data:
  username: YWRtaW4=
  password: NHc1NzIkOXNuczEmIQ==
```

#### Update the Helm chart

If we make changes to the `first-chart`, we can update our cluster with:

```bash
# First check the chart files for the change
helm template first-chart .
# Apply the change to the cluster
helm upgrade first-chart .
```

Monitor the changes:

```bash
kubectl get secrets
kubectl describe secrets first-secret
```

### Rollback a Helm Release

View revision history of the `first-chart` chart:

```bash
helm history first-chart
```

To rollback to the previous (most recent) version, run:

```bash
helm rollback first-chart
```

To verify the change:

```bash
helm history first-chart
```

To rollback to a specific revision (e.g. Revision 2), run:

```bash
# Last argument is the revision number displayed, on the helm history command
helm rollback first-chart 2
```

### Render a ConfigMap value dynamically

Helm allows us to use the double curly braces syntax (`{{ }}`) to add variable values in our charts. Helm will pull the values from the `Chart.yaml` and `values.yaml` files.

To add a version value in our ConfigMap we can have in `templates/configmap.yaml`:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: first-chart-configmap-{{.Chart.Version}}
data:
  port: "8080"
  allowTesting: "true"
```

In the `Chart.yaml` file the `version` value is already present, and we can update the minor version:

```yaml
apiVersion: v2
name: first-chart
description: A Helm chart for Kubernetes

type: application

version: 0.1.1

appVersion: "1.16.0"
```

#### Update the Helm chart

If we make changes to the `first-chart`, we can update our cluster with:

```bash
# Apply the change to the cluster
helm upgrade first-chart .
```

Monitor the changes:

```bash
kubectl get cm
```

### Uninstall a helm chart

To uninstall the `first-chart`:

```bash
helm uninstall first-chart
```

## Links

- [LinkedIn Learning - Kubernetes: Package Management with Helm](https://www.linkedin.com/learning/kubernetes-package-management-with-helm/)
- [Kubernetes Docs - Install Tools](https://kubernetes.io/docs/tasks/tools/)
- [Kind - Local Kubernetes Cluster](https://kind.sigs.k8s.io/docs/)
- [Minikube - Local Kubernetes Cluster](https://minikube.sigs.k8s.io/docs/)
- [Helm Docs - Install Helm](https://helm.sh/docs/intro/install/)
- [Helm Docs - Helm Completion](https://helm.sh/docs/helm/helm_completion/)
