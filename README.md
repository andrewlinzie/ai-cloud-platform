# AI Cloud Platform - EKS + GitOps Delivery System

This project is a production-style cloud platform that demonstrates how to build, deploy, and manage a modern microservices system using:
- AWS EKS (Kubernetes)
- GitOps with ArgoCD
- CI/CD with GitHub Actions
- Infrastructure as Code with Terraform
- Hybrid architecture (EKS + EC2 + Jenkins)

The system is designed to reflect real-world platform engineering practices, not just isolated components.

# 🧠 System Overview

This platform consists of:

## 🔹 Microservices (EKS)
- api-service -> public-facing API (entry point)
- ai-inference-service -> internal AI processing service

## 🔹 Internal System (EC2)
- cms-monolith -> internal content management system (Docker + Jenkins)

## 🔹 Platform Infrastructure
- terraform-platform -> AWS infrastructure (VPC, EKS, IAM, ECR, EC2)
- gitops-infra -> Kubernetes deployment state (source of truth for workloads)

## 🏗️ Architecture

At a high level:

```
Developer → GitHub (api-service repo)
        ↓
GitHub Actions (CI)
        ↓
Build Docker Image → Push to ECR
        ↓
Update GitOps Repo (image tag)
        ↓
ArgoCD detects change
        ↓
Deploy to EKS
```

# ⚙️ Core Design Principles

## 1. Separation of Concerns

Each system has clear ownership boundaries:
- Terraform
  - Owns AWS infrastructure (EKS, VPC, IAM, EC2)
- GitOps (ArgoCD)
  - Owns Kubernetes deployment state
- Service Repos
  - Own build + artifact creation only
- No overlapping responsibility

## 2. GitOps Deployment Model

This platform follows a true GitOps workflow:
- No direct kubectl apply
- No deployments from CI pipelines
- Git is the single source of truth

CI updates:

`
gitops-infra/dev/api-service/values.yaml
`

ArgoCD:
- Watches repo
- Detects changes
- Applies them to the cluster

## 3. Build Once, Promote Everywhere

The system uses immutable artifacts:
- Docker images tagged with commit SHA
- Same image promoted across environments

`
dev -> staging -> prod
`

No rebuilding per environment.

## 4. Environment Promotion Strategy

| Environment | Deployment Type |
|-------------|-----------------|
| dev	      | automatic       |
| staging     | manual approval |
| prod	      | manual approval |

This reduces blast radius and enforces controlled releases.

# 🔄 Example Deployment Flow (API Service)

When code is pushed to api-service:

## Step 1 - CI Pipeline Starts

GitHub Actions triggers the workflow.

## Step 2 — Validation (Quality Gate)
- Linting
- Tests

If this fails → pipeline stops

## Step 3 — Build Artifact
- Docker image is built
- Tagged with commit SHA

Example:

`
api-service:abc123
`

## Step 4 — Secure AWS Access (OIDC)
- No stored AWS credentials
- GitHub assumes IAM role dynamically

## Step 5 — Push to ECR

Image is stored in:

`
Amazon ECR
`

## Step 6 — Update GitOps Repo

Pipeline updates:

`
gitops-infra/dev/api-service/values.yaml
`

Only change:

`
image.tag = <commit SHA>
`

## Step 7 — ArgoCD Deploys

ArgoCD:
- Detects Git change
- Syncs cluster
- Rolls out new version

## 🔁 Result

`
push -> test -> build -> push -> update Git -> Argo deploys
`

# 🧱 Infrastructure (Terraform)

Terraform is used to provision:
- VPC (public/private subnets)
- EKS cluster + node groups
- IAM roles and policies
- ECR repositories
- EC2 instance for CMS
- Remote state (S3 + DynamoDB)

## Terraform Execution Model
- Runs via GitHub Actions
- Uses OIDC (no static credentials)
- Environment-based execution:
  - dev -> auto apply
  - staging -> approval required
  - prod -> approval required
 
# 🔧 Hybrid Architecture (Intentional)

This system is intentionally not fully Kubernetes-based:

| Component  | Runtime |
|------------|---------|
|API Service | EKS     |
|AI Inference| EKS     |
|CMS Monolith| EC2     |

## Why?

The CMS is:
- Internal-facing
- Low concurrency
- Operationally simpler

So it uses:
- Docker
- EC2
- Jenkins

Instead of unnecessary Kubernetes complexity.

# 📦 CI/CD Systems

## GitHub Actions (Microservices + Infra)
- API & AI services
- Terraform pipelines

## Jenkins (CMS)
- Builds CMS image
- Deploys to EC2
- Handles container lifecycle

# 👀 Observability Readiness

The platform includes foundational observability signals:
- `/health` endpoints for all services
- Kubernetes logs (kubectl logs)
- ArgoCD deployment visibility
- CI/CD pipeline logs (GitHub + Jenkins)

This enables:
- debugging deployments
- validating runtime behavior
- monitoring system health

The system is designed to support future integration with:
- Prometheus
- Grafana
- centralized logging

# 🔁 Rollback Strategy

Rollback is Git-driven:
- Revert GitOps commit
- ArgoCD re-syncs cluster
- Previous version is restored

This ensures:
- fast recovery
- no manual cluster changes
- full traceability

# 💡 Key Highlights

- Fully automated IaC via CI/CD
- Secure AWS access using OIDC (no secrets)
- True GitOps deployment model
- Immutable artifact promotion
- Clear system boundaries
- Hybrid architecture (EKS + EC2)
- End-to-end working platform

# 🔗 Repositories

- API Service: https://github.com/andrewlinzie/api-service
- AI Inference Service: https://github.com/andrewlinzie/ai-inference-service
- CMS Monolith: https://github.com/andrewlinzie/cms-monolith
- Terraform Platform: https://github.com/andrewlinzie/terraform-platform
- GitOps Infrastructure: https://github.com/andrewlinzie/gitops-infra

# 🎯 What This Demonstrates

This project demonstrates the ability to:
- Design production-style cloud architectures
- Implement CI/CD pipelines correctly
- Apply GitOps principles in practice
- Build systems with clear ownership boundaries
- Think in terms of platform engineering, not just services
