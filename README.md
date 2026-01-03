# Deploying WordPress on Amazon EKS Using Helm (MacOS)

## Watch me perform this project here: 

## Overview

This project demonstrates how to deploy a production-ready **WordPress application** on **Amazon Elastic Kubernetes Service (EKS)** using **Helm** as the package manager.  
The implementation covers cluster provisioning, application deployment, service exposure, and horizontal pod autoscaling.

This project is designed to reflect **real-world cloud engineering practices**, emphasizing Infrastructure as Code, Kubernetes-native tooling, scalability, and clean teardown.

---

## Architecture Overview

**Core Components**
- **AWS EKS** – Managed Kubernetes control plane
- **Managed Node Group** – EC2 worker nodes with autoscaling
- **Helm** – Application packaging and lifecycle management
- **WordPress (Containerized)** – Deployed as a Kubernetes workload
- **AWS LoadBalancer Service** – Public ingress
- **Horizontal Pod Autoscaler (HPA)** – CPU-based scaling

**Traffic Flow**
```
User Browser
   |
AWS Load Balancer
   |
Kubernetes Service
   |
WordPress Pods (Auto-Scaled)
```

---

## Prerequisites

### Local Environment (MacOS)
- macOS with admin privileges
- Homebrew installed

### AWS Requirements
- AWS account
- IAM user with EKS, EC2, VPC, IAM permissions
- Programmatic access (Access Key + Secret Key)

---

## Tooling Installation

All tools are installed using **Homebrew**.

### Verify Homebrew
```bash
brew --version
```

### Install AWS CLI
```bash
brew install awscli
aws --version
```

### Install kubectl
```bash
brew install kubectl
kubectl version --client
```

### Install eksctl
```bash
brew tap weaveworks/tap
brew install weaveworks/tap/eksctl
eksctl version
```

### Install Helm
```bash
brew install helm
helm version
```

---

## AWS Configuration

Configure AWS credentials locally:
```bash
aws configure
```

**Required Inputs**
- AWS Access Key ID
- AWS Secret Access Key
- Default region: `us-east-1`
- Output format: `json`

---

## EKS Cluster Provisioning

Create an EKS cluster with a managed node group:
```bash
eksctl create cluster   --name enterprise-cluster   --region us-east-1   --nodegroup-name standard-workers   --node-type t3.micro   --nodes 3   --nodes-min 3   --nodes-max 6   --managed
```

**Provisioning Time:** ~15–25 minutes

---

## Cluster Validation

### Update kubeconfig
```bash
aws eks update-kubeconfig --name enterprise-cluster --region us-east-1
```

### Verify Node Status
```bash
kubectl get nodes
```

Expected:
```
STATUS: Ready
```

---

## Helm Chart Creation

Create a Helm chart scaffold:
```bash
helm create my-microservice
```

This generates:
```
my-microservice/
├── Chart.yaml
├── values.yaml
└── templates/
```

---

## WordPress Configuration

Customize `values.yaml` to define:
- WordPress container image
- Kubernetes Service type (`LoadBalancer`)
- CPU and memory requests/limits
- Persistent storage configuration

Helm dynamically renders Kubernetes manifests using these values.

---

## Application Deployment

Deploy WordPress using Helm:
```bash
helm install my-microservice ./my-microservice
```

Verify:
```bash
helm status my-microservice
kubectl get pods
```

---

## Service Exposure

Retrieve the public endpoint:
```bash
kubectl get svc
```

Locate the **EXTERNAL-IP** of the LoadBalancer and access it via browser to complete WordPress setup.

---

## Horizontal Pod Autoscaling (HPA)

### Create HPA Definition
```bash
touch hpa.yaml
```

Apply:
```bash
kubectl apply -f hpa.yaml
```

Verify:
```bash
kubectl get hpa
```

---

## Load Testing and Autoscaling Validation

Generate traffic using Siege:
```bash
kubectl run siege   --image=jstarcher/siege   --restart=Never   -- /bin/sh -c "siege -c 100 -t 5m http://<EXTERNAL-IP>"
```

Observe scaling:
```bash
kubectl get hpa -w
kubectl get pods -w
```

Expected behavior:
- CPU utilization increases
- Additional pods are created automatically

---

## Cleanup (Critical)

To avoid unnecessary AWS costs:
```bash
eksctl delete cluster --name enterprise-cluster --region us-east-1
```

---

## Key Takeaways

- Demonstrates Kubernetes-native application deployment on AWS
- Uses managed services for reliability and scalability
- Implements autoscaling based on real load
- Mirrors production cloud engineering workflows

---


