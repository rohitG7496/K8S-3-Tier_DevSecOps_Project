# 3-Tier DevSecOps Project Setup Guide

This guide provides step-by-step instructions to set up the entire stack on a self-managed Kubernetes cluster (setup via `kubeadm`).

## Prerequisites

- **Kubernetes Cluster**: A running cluster set up using `kubeadm` (v1.23+ recommended).
- **AWS Environment**: If using EBS CSI Driver, your worker nodes must be running on AWS EC2 and have the appropriate IAM permissions attached to their instance profiles.
- **Tools**: `kubectl`, `helm` (optional), `git` installed on your management machine.

---

## 1. Cluster & Infrastructure Components

### 1.1 Kubeadm Cluster Setup
Ensure your cluster is up and running. If you haven't set it up yet, follow the official [Kubernetes implementation with kubeadm](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/) guide.
*   **Master Node**: Control plane components.
*   **Worker Nodes**: Where your application and tools will run.

### 1.2 Install Add-ons

#### EBS CSI Driver
1.  **IAM Permissions**: Ensure your EC2 Worker Nodes have the `AmazonEBSCSIDriverPolicy` attached.
2.  **Install Driver** (using local folder):
    ```bash
    kubectl apply -k ebs-csi-driver
    ```

#### NGINX Ingress Controller
Deploy the NGINX Ingress Controller to manage external access:
```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/baremetal/deploy.yaml
```
*Note: Check the service type. On AWS, you might want to patch it to `LoadBalancer` or use `NodePort` with an external Load Balancer.*

#### Cert-Manager
Install cert-manager for certificate management:
```bash
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.12.0/cert-manager.yaml
```

---

## 2. CI/CD Tools Setup for DevSecOps

### 2.1 SonarQube Setup
Deploy SonarQube using the provided manifest:
```bash
kubectl apply -f sonarqube.yml
```
*   **Storage**: Uses `storageClassName: ebs-sc` (ensure this SC exists or update manifest to match your EBS CSI setup).
*   **Access**: `https://sonar.jnrpro.solutions` (Ensure DNS points to your Ingress Controller).

### 2.2 Jenkins Setup
Deploy Jenkins:
```bash
kubectl apply -f jenkins.yml
```
*   **Access**: `https://jenkins.jnrpro.solutions`

To retrieve the initial admin password:
```bash
kubectl logs jenkins-0 -n base | grep -A 5 "Jenkins initial setup is required"
```

### 2.3 ArgoCD Setup
Deploy ArgoCD for GitOps:
```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/warm/manifests/install.yaml

# Apply project specific ArgoCD configs
kubectl apply -f k8s-manifest-files/argocd.yml
```

---

## 3. Application Deployment (3-Tier)

### 3.1 Local Development
1.  **Backend**: `cd api && npm install && npm start`
2.  **Frontend**: `cd client && npm install && npm start`
3.  **Database**: Ensure a local MySQL instance is running or use Docker.

### 3.2 Kubernetes Deployment
Deploy the application components to your cluster:

```bash
# 1. Secrets (Database Credentials)
# Ensure you have updated secrets.yaml with your base64 encoded values
kubectl apply -f k8s-manifest-files/secrets.yaml

# 2. Database (MySQL)
kubectl apply -f k8s-manifest-files/db.yml

# 3. Backend API
kubectl apply -f k8s-manifest-files/backend.yaml

# 4. Frontend Client
kubectl apply -f k8s-manifest-files/frontend.yaml
```

---

## 4. Verification

1.  **Check Pods**: `kubectl get pods -A` - Ensure all pods are `Running`.
2.  **Check Storage**: `kubectl get pvc -A` - Ensure all claims are `Bound`.
3.  **Check Ingress**: `kubectl get ingress -A` - Verify strict addresses.
