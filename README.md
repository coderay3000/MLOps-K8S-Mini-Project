# Kubernetes Learning Notes & Practical Workflow

A comprehensive guide covering Kubernetes fundamentals, CI/CD integration, managed Kubernetes services (EKS, AKS, GKE), Minikube setup, deployment workflow, troubleshooting, and production best practices.

---

# Table of Contents

- Introduction
- Kubernetes in CI/CD Workflow
- Where Kubernetes Clusters are Created in Real Life
- Managed Kubernetes Services
  - AWS EKS
  - Azure AKS
  - Google GKE
- EKS vs AKS vs GKE Comparison
- Kubernetes Project Workflow
- Minikube Installation
- Deployment Steps
- Troubleshooting
- Debugging
- Load Testing
- ImagePullBackOff Fix
- Key Takeaways

---

# Introduction

Kubernetes is an open-source container orchestration platform that automates:

- Deployment
- Scaling
- Load balancing
- Self-healing
- Rolling updates

Instead of manually running Docker containers on servers, Kubernetes manages containers automatically.

---

# Kubernetes in a CI/CD Workflow

## Traditional Workflow

```
GitHub
    ↓
GitHub Actions
    ↓
Build Docker Image
    ↓
Push to Amazon ECR
    ↓
Deploy to EC2
```

---

## Kubernetes Workflow

```
GitHub
    ↓
GitHub Actions
    ↓
Build Docker Image
    ↓
Push Image to ECR
    ↓
kubectl apply
    ↓
Kubernetes Cluster
```

Instead of deploying directly on an EC2 instance, GitHub Actions deploys the latest Docker image into a Kubernetes cluster.

Example deployment step:

```yaml
- name: Deploy to Kubernetes
  run: |
    kubectl apply -f deployment.yaml
```

---

## Example GitHub Actions Workflow

```yaml
name: CI/CD Pipeline with Kubernetes

on:
  push:
    branches:
      - main

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:

      - uses: actions/checkout@v3

      - name: Login to ECR
        run: aws ecr get-login-password --region <region> |
             docker login --username AWS --password-stdin \
             <account-id>.dkr.ecr.<region>.amazonaws.com

      - name: Build Docker Image
        run: |
          docker build -t app:latest .
          docker push <ecr-url>/app:latest

      - name: Deploy to Kubernetes
        uses: azure/k8s-deploy@v4
        with:
          manifests: |
            deployment.yaml
            service.yaml
```

---

# Benefits of Kubernetes

- Automatic scaling
- Rolling updates
- Zero downtime deployment
- Self healing
- Load balancing
- High availability

---

# Where Are Kubernetes Clusters Created?

## Local Development

Used only for learning and testing.

Tools:

- Minikube
- Kind
- Docker Desktop Kubernetes

---

## Production

Companies generally use managed Kubernetes services instead of self-hosting.

Popular options:

- Amazon EKS
- Azure AKS
- Google GKE

Large enterprises with strict compliance requirements may also create on-premise Kubernetes clusters using tools like:

- kubeadm
- Rancher

---

# Why Minikube Isn't Used in Production

Minikube is intended only for local development.

Limitations:

- Single-machine setup
- Limited scalability
- No production-grade high availability
- Not suitable for enterprise workloads

---

# Managed Kubernetes Services

Cloud providers manage the Kubernetes control plane while developers manage their applications.

---

## AWS Elastic Kubernetes Service (EKS)

### Features

- Managed Control Plane
- IAM integration
- CloudWatch integration
- Elastic Load Balancer support
- VPC networking
- Auto Scaling Groups

### Best For

- Existing AWS users
- Enterprise applications
- Large-scale workloads

---

## Azure Kubernetes Service (AKS)

### Features

- Beginner-friendly
- Azure AD integration
- Azure Monitor
- Blob Storage support
- Automated upgrades
- Virtual Nodes

### Best For

- Microsoft ecosystem
- Azure ML
- Hybrid cloud deployments

---

## Google Kubernetes Engine (GKE)

### Features

- Created by Google
- GKE Autopilot
- Automatic scaling
- Cloud Monitoring
- Vertex AI integration
- GPU and TPU support

### Best For

- AI/ML workloads
- Startups
- Modern cloud-native applications

---

# EKS vs AKS vs GKE

| Feature | AWS EKS | Azure AKS | Google GKE |
|----------|----------|-----------|------------|
| Ease of Use | Medium | Beginner Friendly | Beginner Friendly |
| Cloud Integration | AWS | Azure | Google Cloud |
| AI/ML Support | SageMaker | Azure ML | Vertex AI |
| Networking | Advanced | Simple | Simple |
| Autoscaling | Auto Scaling Groups | Built-in | Advanced |
| Best For | Enterprise | Microsoft Users | AI/ML |

---

# Kubernetes Project Workflow

---

## Step 1

Build your application.

Verify it works locally.

---

## Step 2

Create Docker Image

```bash
docker build -t kubernetes-test-app:latest .
```

Verify:

```bash
docker images
```

Run locally:

```bash
docker run -p 5000:5000 kubernetes-test-app:latest
```

---

## Step 3

Create

```
deployment.yaml
```

---

## Step 4

Install Minikube

Start Minikube

```bash
minikube start
```

or

```bash
minikube start --embed-certs
```

---

# Verify Minikube

```bash
minikube status
```

```bash
kubectl get all -A
```

```bash
kubectl get pods -A
```

```bash
kubectl get nodes -A
```

---

# Add Multiple Nodes

```bash
minikube start --nodes=2
```

or

```bash
minikube start --nodes=2 --embed-certs
```

---

# Load Docker Image into Minikube

List Docker images

```bash
docker images
```

List Minikube images

```bash
minikube image list
```

Load image

```bash
minikube image load kubernetes-test-app:latest
```

---

# Deploy Application

```bash
kubectl apply -f deployment.yaml
```

Delete deployment

```bash
kubectl delete deployment kubernetes-test-app
```

---

# Test Deployment

Check Pods

```bash
kubectl get pods
```

Check Nodes

```bash
kubectl get nodes
```

Delete Pod

```bash
kubectl delete pod <pod-name>
```

Access Service

```bash
minikube service kubernetes-test-app
```

Open Dashboard

```bash
minikube dashboard
```

---

# Debugging

View logs

```bash
kubectl logs -f <pod-name>
```

View endpoints

```bash
kubectl get endpoints
```

View services

```bash
kubectl get service
```

---

# Load Testing

Use Postman Performance Runner.

Example:

- 10 users
- 1 minute
- Fixed configuration

Monitor:

- Response time
- Error rate
- Throughput

---

# Stop Minikube

```bash
minikube stop
```

---

# ImagePullBackOff Fix

Tag image

```bash
docker tag kubernetes-test-app:latest \
<dockerhub-username>/kubernetes-test-app:latest
```

Push image

```bash
docker push <dockerhub-username>/kubernetes-test-app:latest
```

---

# Troubleshooting

## Registry Connection Error

```
Failed to connect to registry.k8s.io
```

If using a proxy:

```bash
minikube start \
--docker-env HTTP_PROXY=http://proxy:port \
--docker-env HTTPS_PROXY=https://proxy:port
```

Without a proxy:

```bash
unset HTTP_PROXY
unset HTTPS_PROXY
unset NO_PROXY
```

Check DNS

```bash
nslookup registry.k8s.io
```

Reset Minikube

```bash
minikube stop

minikube delete --all
```

Restart

```bash
minikube start
```

---

# Key Takeaways

- Kubernetes automates deployment and scaling of containerized applications.
- GitHub Actions can deploy directly to Kubernetes clusters using `kubectl`.
- Minikube is ideal for learning but not suitable for production.
- Managed Kubernetes services (EKS, AKS, and GKE) reduce operational complexity.
- GKE is particularly strong for AI/ML workloads due to GPU/TPU and Vertex AI integration.
- Kubernetes provides rolling updates, self-healing, load balancing, and high availability.
- Production environments typically rely on managed Kubernetes services rather than self-hosted clusters.

---

## Learning Roadmap

```
Docker
    ↓
Docker Compose
    ↓
Kubernetes Basics
    ↓
Pods
    ↓
ReplicaSets
    ↓
Deployments
    ↓
Services
    ↓
ConfigMaps & Secrets
    ↓
Ingress
    ↓
Volumes
    ↓
Helm
    ↓
Kubernetes on Cloud (EKS / AKS / GKE)
    ↓
CI/CD with Kubernetes
```

---

## Author

**Ayush**

These notes were created as part of an AI/ML and MLOps learning journey, covering Kubernetes from local development with Minikube to production-ready deployments using managed Kubernetes services.