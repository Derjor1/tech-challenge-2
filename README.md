# AWS DevOps Tech Challenge – EKS Deployment with CI/CD

## Overview

This project demonstrates a complete end-to-end DevOps workflow on AWS, including:

* Containerizing a Python Flask application
* Storing images in Amazon ECR
* Provisioning infrastructure using Terraform
* Deploying to Amazon EKS using Helm
* Exposing the application via an AWS Application Load Balancer (ALB)
* Implementing Horizontal Pod Autoscaling (HPA)
* Automating the entire process using Jenkins CI/CD

The goal of this project is to simulate a production-style deployment pipeline with infrastructure, application, and automation layers working together.

---

## Architecture

```
User (Browser)
    ↓
AWS Application Load Balancer (ALB)
    ↓
Kubernetes Ingress
    ↓
Service (ClusterIP)
    ↓
Pod (Flask App running in container)
    ↓
Amazon EKS (Node Group - EC2 t3.small)
```

### CI/CD Flow

```
GitHub
  ↓
Jenkins Pipeline
  ↓
Docker Build (AMD64 via buildx)
  ↓
Push to Amazon ECR
  ↓
Helm Deploy to EKS
```

---

## Live Application

Public URL:

```
http://k8s-default-techchal-8d5fca6f00-280804531.us-east-1.elb.amazonaws.com
```

---

## Technologies Used

* AWS EKS
* AWS ECR
* AWS IAM
* Terraform
* Docker
* Kubernetes
* Helm
* Jenkins
* Python (Flask)

---

## Project Structure

```
tech-challenge-2/
│
├── app.py
├── requirements.txt
├── Dockerfile
├── .dockerignore
├── Jenkinsfile
├── .gitignore
│
├── terraform/
│   └── main.tf
│
├── helm/
│   └── tech-challenge-app/
│       ├── Chart.yaml
│       ├── values.yaml
│       └── templates/
│           ├── deployment.yaml
│           ├── service.yaml
│           ├── ingress.yaml
│           ├── hpa.yaml
│
└── README.md
```

---

## Setup Instructions

### 1. Clone the Repository

```bash
git clone https://github.com/YOUR_USERNAME/tech-challenge-2.git
cd tech-challenge-2
```

---

### 2. Provision Infrastructure (Terraform)

```bash
cd terraform

terraform init
terraform apply
```

This creates:

* VPC and networking components
* EKS cluster
* Managed node group (t3.small)
* IAM roles and policies

---

### 3. Configure kubectl Access

```bash
aws eks update-kubeconfig \
  --region us-east-1 \
  --name tech-challenge-cluster
```

Verify:

```bash
kubectl get nodes
```

---

### 4. Build and Push Docker Image (Manual - Initial Setup)

```bash
docker buildx build \
  --platform linux/amd64 \
  -t <ACCOUNT_ID>.dkr.ecr.us-east-1.amazonaws.com/tech-challenge-app:latest \
  --push .
```

---

### 5. Deploy Application with Helm

```bash
cd helm/tech-challenge-app

helm upgrade --install tech-challenge-app .
```

---

### 6. Access Application

```bash
kubectl get ingress
```

Open the ALB DNS URL in your browser.

---

## Jenkins CI/CD Pipeline

The Jenkins pipeline automates the entire deployment process.

### Pipeline Stages

1. **Checkout Code**
2. **Verify Tools (AWS CLI, kubectl, Helm, Docker)**
3. **Login to Amazon ECR**
4. **Build Docker Image (AMD64 using buildx)**
5. **Push Image to ECR**
6. **Deploy to EKS using Helm**
7. **Verify Deployment**

### Example Flow

```
Code Push → Jenkins → Build Image → Push to ECR → Deploy → Live Update
```

---

## Horizontal Pod Autoscaling (HPA)

The application is configured to scale based on:

* CPU utilization (target: 50%)
* Memory utilization (target: 50%)

### Configuration

* Minimum pods: 1
* Maximum pods: 12

### Verify HPA

```bash
kubectl get hpa
```

---

## Terraform Explanation

Terraform is used to provision all AWS infrastructure.

### Key Components

* EKS Cluster
* Node Group (t3.small instances)
* VPC and subnets
* IAM roles and policies

### Why Terraform?

* Infrastructure as Code
* Repeatable deployments
* Version-controlled infrastructure

---

## Key Design Decisions

### 1. Multi-Architecture Docker Builds

Because the development machine uses ARM (Mac), the pipeline uses:

```bash
--platform linux/amd64
```

to ensure compatibility with EKS nodes.

---

### 2. Helm for Deployment

Helm was used to:

* Template Kubernetes resources
* Manage versioned deployments
* Simplify upgrades

---

### 3. AWS Load Balancer Controller

Used to:

* Automatically create ALB from Kubernetes Ingress
* Route external traffic to the application

---

### 4. Jenkins for CI/CD

Jenkins automates:

* Build
* Push
* Deploy

This eliminates manual deployment steps.

---

## Challenges & Fixes

### 1. Docker Architecture Mismatch

* Issue: ARM vs AMD64
* Fix: Used `docker buildx` with `--platform linux/amd64`

---

### 2. IAM Permission Errors

* Issue: ALB controller could not create resources
* Fix: Updated IAM policy with required permissions

---

### 3. GitHub Push Failure (Large Files)

* Issue: `.terraform` files exceeded 100MB limit
* Fix: Added `.gitignore` and removed cached files

---

### 4. Jenkins Docker Permissions

* Issue: Docker socket permission denied
* Fix: Ran Jenkins container as root

---

### 5. AWS Credentials in Jenkins

* Issue: Jenkins could not authenticate to AWS
* Fix: Mounted `~/.aws` to `/root/.aws`

---

### 6. kubeconfig Read-Only Error

* Issue: Pipeline attempted to modify kubeconfig
* Fix: Removed unnecessary `kubectl config use-context`

---

## Improvements (Production Considerations)

For a production environment, the following improvements would be made:

* Use IAM Roles for Service Accounts (IRSA) instead of mounted credentials
* Run Jenkins agents instead of using controller as executor
* Use GitOps (ArgoCD) for deployments
* Add HTTPS (TLS) to ALB
* Implement monitoring (CloudWatch / Prometheus)
* Add readiness and liveness probes

---

## Conclusion

This project demonstrates a complete cloud-native deployment pipeline using AWS and Kubernetes. It covers:

* Infrastructure provisioning
* Containerization
* Orchestration
* Scaling
* CI/CD automation

The system is fully functional and capable of deploying updates automatically through Jenkins.

---

## Bonus: GitOps Implementation (GitHub Actions + Argo CD)

This project includes a fully working GitOps-based deployment model implemented on the `gitops` branch.

### Overview

In this model, Git becomes the single source of truth for the system state. Instead of deploying directly from a CI tool (like Jenkins), all changes flow through Git, and Argo CD ensures the Kubernetes cluster matches what is defined in the repository.
