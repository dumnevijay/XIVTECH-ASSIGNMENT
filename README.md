# Kubernetes Hello World NGINX Application

This repository contains a simple "Hello World" web application using NGINX deployed on a local Kubernetes cluster.

## Prerequisites

- Docker
- Kind (Kubernetes in Docker)
- kubectl

## Project Structure

```
.
├── app
│   └── index.html               # Simple Hello World HTML page served by NGINX
├── docker
│   └── Dockerfile               # Dockerfile to build a custom NGINX image with our HTML
├── k8s
│   ├── deployment.yaml          # Kubernetes Deployment: defines the NGINX pod and its config
│   └── service.yaml             # Kubernetes Service: exposes the deployment using NodePort
├── demo                         # Contains video demonstration of the deployment process
├── argocd-application.yaml      # Argo CD Application: declares the Git-based Kubernetes app for GitOps
├── kind-config.yaml             # Kind Cluster Config: maps container port 30000 to host for NodePort access
└── README.md                    # Project documentation and setup instructions
```

## Step-by-Step Guide

### 1. Install Prerequisites

#### Install Docker

Follow the [official Docker installation guide](https://docs.docker.com/get-docker/) for your operating system.

#### Install Kind

```bash
# For macOS (using Homebrew)
brew install kind

# For Linux
curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.20.0/kind-linux-amd64
chmod +x ./kind
sudo mv ./kind /usr/local/bin/kind

# For Windows (using Chocolatey)
choco install kind
```

#### Install kubectl

```bash
# For macOS (using Homebrew)
brew install kubectl

# For Linux
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x kubectl
sudo mv kubectl /usr/local/bin/

# For Windows (using Chocolatey)
choco install kubernetes-cli
```

### 2. Create a Local Kubernetes Cluster

```bash
kind create cluster --name hello-world-cluster --config kind-config.yaml
```

Verify that the cluster is running:

```bash
kubectl get nodes
```

### 3. Build the Docker Image

```bash
# Navigate to the project root directory
docker build -t hello-world-nginx -f docker/Dockerfile .
```

### 4. Load the Docker Image into Kind

```bash
kind load docker-image hello-world-nginx --name hello-world-cluster
```

### 5. Deploy the Application to Kubernetes

```bash
kubectl apply -f k8s/ 
```

Or use

```bash
kubectl apply -f k8s/deployment.yaml
kubectl apply -f k8s/service.yaml
```

### 6. Verify the Deployment

Check if the pods are running:

```bash
kubectl get pods
```

Check if the service is created:

```bash
kubectl get svc
```

### 7. Access the Application

The application is exposed on NodePort 30000. You can access it by:

```bash
# If using Kind on local machine
curl localhost:30000

# Or open in a browser
http://localhost:30000
```

### 8. Clean Up

To delete the Kubernetes resources:

```bash
kubectl delete -f k8s/
```

To delete the Kind cluster:

```bash
kind delete cluster --name hello-world-cluster
```

## Optional: Argo CD Integration

To set up Argo CD for GitOps:

### 1. Install Argo CD on your cluster

```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

### 2. Access the Argo CD API Server

```bash
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

Access the UI at: https://localhost:8080

### 3. Login to Argo CD

The default username is `admin`. Get the initial password with:

```bash
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
```

### 4. Create an Argo CD Application

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: hello-world-app
  namespace: argocd
spec:
  project: default
  source:
    repoURL: 'YOUR_GIT_REPO_URL'
    path: k8s
    targetRevision: HEAD
  destination:
    server: 'https://kubernetes.default.svc'
    namespace: default
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```

Apply this configuration:

```bash
kubectl apply -f argocd-application.yaml
```

## Video Demonstration

See the `demo/` directory for a video walkthrough of the setup and deployment process.