# Helm Chart

### This Helm chart deploys two-tire-application on a Kubernetes cluster.

### Prerequisites
- Kubernetes cluster running (local, cloud, or KIND).
- kubectl installed and configured.
- Helm 3 installed. Install Helm with:
```bash

curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
chmod 700 get_helm.sh
./get_helm.sh
```

## Chart Structure
```bash

apache/
├── Chart.yaml         # Chart metadata
├── values.yaml        # Default values for customization
└── templates/         # Kubernetes resource templates
    ├── deployment.yaml
    └── service.yaml

# Two-tire applicaiton
- Make a new directoy for project helm charts
```
mkdir two-tire-app
helm create mysql-chart
```

- Go to  templates/deployment.yaml to change Image and port and add the enviornment variables there
- Edit Values.yaml to save the enviornment variables.

Package chart with

```
helm package mysql-chart

```
then Install heml-chart

```
 helm install mysql-chart ./mysql-chart

```
To check if its helm chart deplyed or not 

```
kubectl get all

```

> If something goes wrong then `helm uninstall mysql-chart` then make changes in mysql-chart/ files and again package and install



