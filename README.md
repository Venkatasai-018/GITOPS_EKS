# GitOps on Amazon EKS with Terraform and ArgoCD

[![Terraform](https://img.shields.io/badge/Terraform-%235835CC.svg?logo=terraform&logoColor=white)](https://www.terraform.io/)
[![AWS EKS](https://img.shields.io/badge/AWS%20EKS-FF9900?logo=amazonaws&logoColor=white)](https://aws.amazon.com/eks/)
[![ArgoCD](https://img.shields.io/badge/ArgoCD-EF7B4D?logo=argo&logoColor=white)](https://argoproj.github.io/cd/)
[![Kubernetes](https://img.shields.io/badge/Kubernetes-326CE5?logo=kubernetes&logoColor=white)](https://kubernetes.io/)

A production-ready GitOps implementation for deploying and managing Kubernetes workloads on Amazon EKS using Terraform for infrastructure provisioning and ArgoCD for continuous delivery.

---

## Table of Contents

- [Overview](#overview)
- [Architecture](#architecture)
- [Prerequisites](#prerequisites)
- [Project Structure](#project-structure)
- [Getting Started](#getting-started)
- [Configuration](#configuration)
- [Accessing ArgoCD](#accessing-argocd)
- [Application Stack](#application-stack)
- [Operations](#operations)
- [Troubleshooting](#troubleshooting)
- [Contributing](#contributing)
- [License](#license)

---

## Overview

This repository provides Infrastructure as Code (IaC) to deploy a complete GitOps environment on AWS, featuring:

| Component | Technology | Purpose |
|-----------|------------|---------|
| Infrastructure | Terraform | Automated provisioning of AWS resources |
| Container Orchestration | Amazon EKS (v1.30) | Managed Kubernetes cluster |
| GitOps Engine | ArgoCD | Declarative continuous delivery |
| Storage | EBS CSI Driver | Persistent volume support |
| Networking | AWS VPC | Isolated network with public/private subnets |

### Key Features

- **Fully Automated**: Single `terraform apply` deploys the entire stack
- **GitOps-Native**: Application deployments managed declaratively via Git
- **Production-Ready**: Multi-AZ deployment with private node groups
- **Scalable**: Auto-scaling node groups (2-3 nodes)

---

## Architecture

```
┌──────────────────────────────────────────────────────────────────────────┐
│                              AWS Cloud                                    │
│  ┌────────────────────────────────────────────────────────────────────┐  │
│  │                        VPC (10.0.0.0/16)                           │  │
│  │                                                                    │  │
│  │   ┌──────────────────┐              ┌──────────────────┐          │  │
│  │   │  Public Subnet   │              │  Public Subnet   │          │  │
│  │   │  10.0.101.0/24   │              │  10.0.102.0/24   │          │  │
│  │   │    (AZ-1a)       │              │    (AZ-1b)       │          │  │
│  │   └────────┬─────────┘              └────────┬─────────┘          │  │
│  │            │                                 │                    │  │
│  │            └──────────┬──────────────────────┘                    │  │
│  │                       │                                           │  │
│  │                 ┌─────▼─────┐                                     │  │
│  │                 │  NAT GW   │                                     │  │
│  │                 └─────┬─────┘                                     │  │
│  │                       │                                           │  │
│  │   ┌──────────────────┴───────────────────┐                       │  │
│  │   │                                       │                       │  │
│  │   ▼                                       ▼                       │  │
│  │   ┌──────────────────┐    ┌──────────────────┐                   │  │
│  │   │  Private Subnet  │    │  Private Subnet  │                   │  │
│  │   │  10.0.1.0/24     │    │  10.0.2.0/24     │                   │  │
│  │   │                  │    │                  │                   │  │
│  │   │  ┌────────────────────────────────────┐ │                   │  │
│  │   │  │          EKS Cluster               │ │                   │  │
│  │   │  │  ┌──────────┐   ┌───────────────┐  │ │                   │  │
│  │   │  │  │  ArgoCD  │──▶│  3-Tier App   │  │ │                   │  │
│  │   │  │  │          │   │ ┌───┐ ┌───┐ ┌───┐│ │                   │  │
│  │   │  │  └──────────┘   │ │FE │→│BE │→│DB ││ │                   │  │
│  │   │  │                 │ └───┘ └───┘ └───┘│ │                   │  │
│  │   │  │                 └───────────────┘  │ │                   │  │
│  │   │  └────────────────────────────────────┘ │                   │  │
│  │   └──────────────────┘    └──────────────────┘                   │  │
│  └────────────────────────────────────────────────────────────────────┘  │
└──────────────────────────────────────────────────────────────────────────┘
```

---

## Prerequisites

Ensure the following tools are installed and configured:

| Tool | Version | Purpose |
|------|---------|---------|
| [AWS CLI](https://aws.amazon.com/cli/) | v2.x | AWS authentication and resource management |
| [Terraform](https://www.terraform.io/) | >= 1.0 | Infrastructure provisioning |
| [kubectl](https://kubernetes.io/docs/tasks/tools/) | v1.28+ | Kubernetes cluster management |
| [Git](https://git-scm.com/) | Latest | Version control |

### AWS Configuration

```bash
aws configure
# Ensure your IAM user/role has permissions for EKS, EC2, IAM, and VPC
```

---

## Project Structure

```
.
├── terraform/
│   ├── main.tf              # Core infrastructure resources
│   ├── provider.tf          # Provider configurations (AWS, Kubernetes, kubectl)
│   ├── variables.tf         # Input variable definitions
│   ├── terraform.tf         # Terraform version constraints
│   └── output.tf            # Output value definitions
│
├── manifests/
│   ├── argocd-app.yaml      # ArgoCD Application manifest
│   ├── namespace.yaml       # Kubernetes namespace definition
│   ├── frontend.yaml        # Frontend deployment and service
│   ├── backend.yaml         # Backend deployment and service
│   ├── postgres.yaml        # PostgreSQL StatefulSet with PVC
│   ├── secret.yaml          # Database credentials (base64 encoded)
│   └── kustomization.yaml   # Kustomize configuration
│
├── .gitignore               # Git ignore patterns
└── README.md                # Project documentation
```

---

## Getting Started

### 1. Clone the Repository

```bash
git clone <repository-url>
cd GITOPS_EKS
```

### 2. Initialize Terraform

```bash
cd terraform
terraform init
```

### 3. Review Infrastructure Plan

```bash
terraform plan
```

### 4. Deploy Infrastructure

```bash
terraform apply --auto-approve
```

> **Note**: Deployment takes approximately 15-20 minutes to complete.

### 5. Configure kubectl

```bash
aws eks update-kubeconfig --region us-east-1 --name gitops-eks-cluster
```

### 6. Verify Deployment

```bash
# Check cluster nodes
kubectl get nodes

# Verify ArgoCD installation
kubectl get pods -n argocd

# Check application status
kubectl get applications -n argocd
```

---

## Configuration

### Terraform Variables

| Variable | Description | Default | Required |
|----------|-------------|---------|----------|
| `region` | AWS region for deployment | `us-east-1` | No |
| `cluster_name` | Name of the EKS cluster | `gitops-eks-cluster` | No |
| `vpc_cidr` | CIDR block for the VPC | `10.0.0.0/16` | No |

### Override Defaults

Create a `terraform.tfvars` file:

```hcl
region       = "us-west-2"
cluster_name = "my-gitops-cluster"
vpc_cidr     = "172.16.0.0/16"
```

---

## Accessing ArgoCD

### Retrieve ArgoCD URL

```bash
kubectl get svc argocd-server -n argocd -o jsonpath='{.status.loadBalancer.ingress[0].hostname}'
```

### Retrieve Admin Password

**PowerShell (Windows):**
```powershell
$secret = kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath='{.data.password}'
[System.Text.Encoding]::UTF8.GetString([System.Convert]::FromBase64String($secret))
```

**Bash (Linux/macOS):**
```bash
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath='{.data.password}' | base64 -d && echo
```

### Login Credentials

| Field | Value |
|-------|-------|
| Username | `admin` |
| Password | *(Retrieved from command above)* |

---

## Application Stack

The deployed 3-tier application consists of:

| Component | Image | Port | Description |
|-----------|-------|------|-------------|
| **Frontend** | `itsbaivab/frontend` | 3000 | React-based web interface |
| **Backend** | `itsbaivab/backend` | 8080 | REST API service |
| **Database** | `postgres:latest` | 5432 | PostgreSQL with persistent storage |

### Data Flow

```
User → Frontend (NodePort/LB) → Backend (ClusterIP) → PostgreSQL (ClusterIP)
```

---

## Operations

### Common Commands

```bash
# View all ArgoCD applications
kubectl get applications -n argocd

# Check application sync status
kubectl describe application 3tier-app -n argocd

# View application pods
kubectl get pods -n 3tirewebapp-dev

# Stream application logs
kubectl logs -f deployment/frontend -n 3tirewebapp-dev
kubectl logs -f deployment/backend -n 3tirewebapp-dev

# Monitor ArgoCD server logs
kubectl logs -n argocd -l app.kubernetes.io/name=argocd-server -f
```

### Force Application Sync

```bash
kubectl patch application 3tier-app -n argocd --type merge -p '{"operation": {"initiatedBy": {"username": "admin"}, "sync": {}}}'
```

### Cleanup

```bash
cd terraform
terraform destroy --auto-approve
```

> **Warning**: This will permanently delete all AWS resources including the EKS cluster and data.

---

## Troubleshooting

### Issue: ArgoCD CRD Annotation Too Long

**Error:** `metadata.annotations: Too long: must have at most 262144 bytes`

**Solution:** Enable server-side apply in the kubectl_manifest resource:

```hcl
resource "kubectl_manifest" "argocd" {
  # ...
  server_side_apply = true
  force_conflicts   = true
}
```

### Issue: PowerShell Local-Exec Errors

**Error:** `'#' is not recognized as an internal or external command`

**Solution:** Use PowerShell interpreter:

```hcl
provisioner "local-exec" {
  interpreter = ["PowerShell", "-Command"]
  command     = "..."
}
```

### Issue: Application Not Syncing

**Diagnosis:**
```bash
kubectl describe application 3tier-app -n argocd
kubectl get events -n 3tirewebapp-dev
```

**Common Causes:**
- Git repository not accessible
- Invalid manifests in the Git repository
- Resource quota exceeded

### Issue: Terraform Destroy Fails

**Error:** `DependencyViolation: The subnet has dependencies`

**Solution:** Manually remove stale resources from state:
```bash
terraform state rm 'module.vpc.aws_subnet.public[0]'
terraform destroy --auto-approve
```

---

## Contributing

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/enhancement`)
3. Commit changes (`git commit -m 'Add new feature'`)
4. Push to branch (`git push origin feature/enhancement`)
5. Open a Pull Request

---

## License

This project is licensed under the MIT License. See [LICENSE](LICENSE) for details.

---

<p align="center">
  <sub>Built with Terraform, ArgoCD, and Amazon EKS</sub>
</p>
