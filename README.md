# 🐺 DevOps Implementation — Docker, Kubernetes, AWS EKS, Terraform, CI/CD, GitHub Actions, ArgoCD, Helm, OTEL

**Production-grade DevOps implementation notes with 55+ hands-on demos — from containers to full observability.**

Documenting my hands-on journey building and deploying a complete retail store microservices application using modern DevOps tools and practices on AWS Cloud.

[![AWS](https://img.shields.io/badge/AWS-FF9900?style=for-the-badge&logo=amazonaws&logoColor=white)](https://aws.amazon.com/)
[![Docker](https://img.shields.io/badge/Docker-2496ED?style=for-the-badge&logo=docker&logoColor=white)](https://www.docker.com/)
[![Kubernetes](https://img.shields.io/badge/Kubernetes-326CE5?style=for-the-badge&logo=kubernetes&logoColor=white)](https://kubernetes.io/)
[![Terraform](https://img.shields.io/badge/Terraform-7B42BC?style=for-the-badge&logo=terraform&logoColor=white)](https://www.terraform.io/)
[![Helm](https://img.shields.io/badge/Helm-0F1689?style=for-the-badge&logo=helm&logoColor=white)](https://helm.sh/)
[![ArgoCD](https://img.shields.io/badge/ArgoCD-EF7B4D?style=for-the-badge&logo=argo&logoColor=white)](https://argoproj.github.io/cd/)
[![GitHub Actions](https://img.shields.io/badge/GitHub_Actions-2088FF?style=for-the-badge&logo=githubactions&logoColor=white)](https://github.com/features/actions)
[![Prometheus](https://img.shields.io/badge/Prometheus-E6522C?style=for-the-badge&logo=prometheus&logoColor=white)](https://prometheus.io/)
[![Grafana](https://img.shields.io/badge/Grafana-F46800?style=for-the-badge&logo=grafana&logoColor=white)](https://grafana.com/)

---

## 🏗️ Project Overview

This repository documents the complete implementation of a **5-microservice retail store application** deployed on AWS using production-grade DevOps practices. Every section includes detailed notes, architecture diagrams, commands, troubleshooting, and key takeaways from hands-on labs.

### The Retail Store Architecture

```
┌─────────────────────────────────────────────────────────────────────┐
│                        AWS EKS Cluster                              │
│                                                                     │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────┐ │
│  │    UI    │  │  Catalog  │  │   Carts  │  │  Orders  │  │Check-│ │
│  │ Service  │  │  Service  │  │  Service │  │  Service │  │ out  │ │
│  │ (Spring  │  │   (Go +   │  │ (Spring  │  │ (Spring  │  │(Node │ │
│  │  Boot)   │  │   MySQL)  │  │  Boot +  │  │  Boot +  │  │ .js +│ │
│  │          │  │          │  │ DynamoDB) │  │ Postgres) │  │Redis)│ │
│  └──────────┘  └──────────┘  └──────────┘  └──────────┘  └──────┘ │
│                                                                     │
│  ┌──────────────────────────────────────────────────────────────┐   │
│  │                    Monitoring Stack                           │   │
│  │  Prometheus │ Grafana │ AlertManager │ OpenTelemetry │ X-Ray │   │
│  └──────────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────────┘
                              │
          ┌───────────────────┼───────────────────┐
          ▼                   ▼                   ▼
    ┌──────────┐       ┌──────────┐        ┌──────────┐
    │ RDS MySQL│       │DynamoDB  │        │ElastiCache│
    │ RDS Pg   │       │          │        │  (Redis)  │
    └──────────┘       └──────────┘        └──────────┘
                              │
                              ▼
                       ┌──────────┐
                       │   SQS    │
                       │ (Queue)  │
                       └──────────┘
```

### AWS Data Plane Services Used

| Service | Used By | Purpose |
|---------|---------|---------|
| Amazon RDS (MySQL) | Catalog Service | Product catalog relational database |
| Amazon RDS (PostgreSQL) | Orders Service | Order management database |
| Amazon DynamoDB | Carts Service | Shopping cart NoSQL database |
| Amazon ElastiCache (Redis) | Checkout Service | Session cache and temp storage |
| Amazon SQS | Orders Service | Message broker for order processing |

---

## 📚 Course Sections & Notes

### Section 1: Docker
> Containerization fundamentals to advanced multi-platform builds

| # | Topic | Status | Notes |
|---|-------|--------|-------|
| 1 | [Docker Fundamentals & CLI](./docker/01-fundamentals.md) | ⬜ | pull, run, exec, stop, start, rm, rmi, logs, inspect |
| 2 | [Pull & Run Containers](./docker/02-pull-and-run.md) | ⬜ | Docker Hub, container lifecycle |
| 3 | [Build Images & Push to DockerHub](./docker/03-build-and-push.md) | ⬜ | docker build, tag, push |
| 4 | [Dockerfile Mastery](./docker/04-dockerfile.md) | ⬜ | FROM, COPY, ADD, ARG, ENV, RUN, EXPOSE, CMD, ENTRYPOINT, WORKDIR, HEALTHCHECK, USER |
| 5 | [Docker Compose](./docker/05-compose.md) | ⬜ | Multi-container apps, volumes, networks, health checks, profiles |
| 6 | [Docker BuildKit & Multi-Platform](./docker/06-buildkit.md) | ⬜ | buildx, AMD64/ARM64 builds, multi-stage optimization |

### Section 2: Terraform
> Infrastructure as Code from basics to production EKS clusters

| # | Topic | Status | Notes |
|---|-------|--------|-------|
| 1 | [Tools Installation](./terraform/01-tools-setup.md) | ⬜ | AWS CLI, Terraform, kubectl |
| 2 | [Terraform Foundation](./terraform/02-foundation.md) | ⬜ | Providers, resources, variables, outputs |
| 3 | [Production VPC](./terraform/03-vpc.md) | ⬜ | Public/private subnets, NAT, IGW |
| 4 | [VPC with tfvars](./terraform/04-tfvars.md) | ⬜ | Variable management, precedence |
| 5 | [Remote Backend](./terraform/05-remote-backend.md) | ⬜ | S3 + DynamoDB state locking |
| 6 | [VPC with Remote Backend](./terraform/06-vpc-remote-backend.md) | ⬜ | Production state setup |
| 7 | [Terraform Modules](./terraform/07-modules.md) | ⬜ | Reusable infrastructure modules |
| 8 | [EKS Cluster with Terraform](./terraform/08-eks-cluster.md) | ⬜ | Cluster provisioning, node groups, IAM, kubeconfig |

### Section 3: Kubernetes
> Core concepts to production workload management

| # | Topic | Status | Notes |
|---|-------|--------|-------|
| 1 | [Pods](./kubernetes/01-pods.md) | ⬜ | Creating and managing pods |
| 2 | [Deployments](./kubernetes/02-deployments.md) | ⬜ | Declarative updates, rollouts |
| 3 | [Services](./kubernetes/03-services.md) | ⬜ | ClusterIP, NodePort, LoadBalancer, ExternalName, Headless |
| 4 | [ConfigMaps](./kubernetes/04-configmaps.md) | ⬜ | Environment variables, configuration |
| 5 | [StatefulSets](./kubernetes/05-statefulsets.md) | ⬜ | Stateful applications |
| 6 | [Secrets Management](./kubernetes/06-secrets.md) | ⬜ | K8s Secrets, AWS Secrets Manager, CSI Driver |
| 7 | [Persistent Storage](./kubernetes/07-storage.md) | ⬜ | PV, PVC, StorageClasses, EBS CSI Driver |
| 8 | [Ingress & Load Balancing](./kubernetes/08-ingress.md) | ⬜ | ALB Controller, HTTP/HTTPS, SSL/TLS |
| 9 | [Autoscaling — HPA](./kubernetes/09-hpa.md) | ⬜ | CPU/memory based pod autoscaling |
| 10 | [Autoscaling — Karpenter](./kubernetes/10-karpenter.md) | ⬜ | Node autoscaling, Spot instances, interruption handling |

### Section 4: Helm
> Kubernetes package management for complex deployments

| # | Topic | Status | Notes |
|---|-------|--------|-------|
| 1 | [Helm Basics](./helm/01-basics.md) | ⬜ | Installation, fundamentals |
| 2 | [Custom Values](./helm/02-custom-values.md) | ⬜ | Overrides, environment-specific configs |
| 3 | [Chart Structure](./helm/03-chart-structure.md) | ⬜ | Templates, helpers, dependencies |
| 4 | [Package & Publish](./helm/04-package-publish.md) | ⬜ | Creating and publishing charts |
| 5 | [Retail Store Deployment](./helm/05-retail-store.md) | ⬜ | Full application deployment with Helm |

### Section 5: CI/CD Pipeline
> End-to-end GitOps with GitHub Actions and ArgoCD

| # | Topic | Status | Notes |
|---|-------|--------|-------|
| 1 | [GitHub Actions + ECR](./cicd/01-github-actions.md) | ⬜ | Workflows, Docker builds, OIDC auth, semantic versioning |
| 2 | [ArgoCD Installation](./cicd/02-argocd-setup.md) | ⬜ | Architecture, components, GitOps principles |
| 3 | [CD with ArgoCD + Helm](./cicd/03-argocd-helm.md) | ⬜ | Applications, auto-sync, self-heal |
| 4 | [Complete CI/CD Flow](./cicd/04-full-pipeline.md) | ⬜ | Code → Build → Push → Deploy → Rollback |

### Section 6: Observability (OpenTelemetry)
> Production monitoring, tracing, logging, and metrics

| # | Topic | Status | Notes |
|---|-------|--------|-------|
| 1 | [ADOT Setup](./monitoring/01-adot-setup.md) | ⬜ | AWS Distro for OpenTelemetry, OTEL Collector |
| 2 | [Traces — AWS X-Ray](./monitoring/02-xray-traces.md) | ⬜ | Auto-instrumentation for Java & Node.js, sampling, cost optimization |
| 3 | [Logs — CloudWatch](./monitoring/03-cloudwatch-logs.md) | ⬜ | Log aggregation, Insights queries |
| 4 | [Metrics — Prometheus & Grafana](./monitoring/04-prometheus-grafana.md) | ⬜ | AMP, AMG, dashboards, application metrics |

### Section 7: Production Deployment
> Real-world microservices with AWS data plane integration

| # | Topic | Status | Notes |
|---|-------|--------|-------|
| 1 | [AWS Data Plane Setup](./production/01-data-plane.md) | ⬜ | RDS, ElastiCache, SQS, DynamoDB with Terraform |
| 2 | [Microservices Integration](./production/02-microservices.md) | ⬜ | All 5 services connected to AWS data plane |
| 3 | [External DNS](./production/03-external-dns.md) | ⬜ | Route53, custom domains, SSL automation |
| 4 | [EKS Add-Ons](./production/04-eks-addons.md) | ⬜ | LB Controller, EBS CSI, Pod Identity, Secret Store CSI |

---

## 🗂️ Repository Structure

```
DevOps-Implementation/
├── README.md
├── docker/
│   ├── images/
│   ├── 01-fundamentals.md
│   ├── 02-pull-and-run.md
│   ├── 03-build-and-push.md
│   ├── 04-dockerfile.md
│   ├── 05-compose.md
│   └── 06-buildkit.md
├── terraform/
│   ├── images/
│   ├── 01-tools-setup.md
│   ├── 02-foundation.md
│   ├── 03-vpc.md
│   ├── 04-tfvars.md
│   ├── 05-remote-backend.md
│   ├── 06-vpc-remote-backend.md
│   ├── 07-modules.md
│   └── 08-eks-cluster.md
├── kubernetes/
│   ├── images/
│   ├── 01-pods.md
│   ├── 02-deployments.md
│   ├── 03-services.md
│   ├── 04-configmaps.md
│   ├── 05-statefulsets.md
│   ├── 06-secrets.md
│   ├── 07-storage.md
│   ├── 08-ingress.md
│   ├── 09-hpa.md
│   └── 10-karpenter.md
├── helm/
│   ├── images/
│   ├── 01-basics.md
│   ├── 02-custom-values.md
│   ├── 03-chart-structure.md
│   ├── 04-package-publish.md
│   └── 05-retail-store.md
├── cicd/
│   ├── images/
│   ├── 01-github-actions.md
│   ├── 02-argocd-setup.md
│   ├── 03-argocd-helm.md
│   └── 04-full-pipeline.md
├── monitoring/
│   ├── images/
│   ├── 01-adot-setup.md
│   ├── 02-xray-traces.md
│   ├── 03-cloudwatch-logs.md
│   └── 04-prometheus-grafana.md
└── production/
    ├── images/
    ├── 01-data-plane.md
    ├── 02-microservices.md
    ├── 03-external-dns.md
    └── 04-eks-addons.md
```

---

## 📝 Note Format

Every note follows a consistent structure for easy reference:

```markdown
# Topic Title

## Overview
What this is and why it matters in production.

## Architecture
Diagrams showing how components connect.
(Created using Excalidraw)

## Key Concepts
Core ideas in my own words — not copy-paste.

## Hands-On Implementation
Step-by-step of what I actually built.

## Commands Reference
Important commands with explanations.

## Configuration Files
Key YAML/HCL/Dockerfile configs with annotations.

## Mistakes & Troubleshooting
Things that broke, error messages, and how I fixed them.

## Key Takeaways
Quick summary for interview prep and future reference.
```

---

## 🎯 Learning Path

```
Docker Fundamentals
        │
        ▼
Dockerfile & Compose ──→ Build & Push Images
        │
        ▼
Terraform Basics ──→ VPC ──→ Remote State ──→ Modules
        │
        ▼
EKS Cluster (Terraform) ──→ kubectl Setup
        │
        ▼
K8s Core (Pods, Deployments, Services)
        │
        ▼
K8s Advanced (Secrets, Storage, Ingress, Autoscaling)
        │
        ▼
Helm Charts ──→ Package & Deploy Microservices
        │
        ▼
CI/CD (GitHub Actions ──→ ECR ──→ ArgoCD)
        │
        ▼
Observability (OpenTelemetry ──→ X-Ray ──→ Prometheus ──→ Grafana)
        │
        ▼
Production Deployment (5 Microservices + AWS Data Plane)
```

---

## 🏆 Certifications In Progress

| Certification | Target | Status |
|--------------|--------|--------|
| AWS Solutions Architect Associate (SAA-C03) | March 2026 | 📖 Studying |
| HashiCorp Terraform Associate (003) | March 2026 | ⬜ Up Next |

---

## 🔗 Related Projects

| Project | Description | Stack |
|---------|-------------|-------|
| [CampusQuick](https://github.com/YOUR-USERNAME/CampasQuick-Cloud_Delivery) | Serverless campus delivery platform | AWS Lambda, DynamoDB, S3, CloudFront, Cognito, API Gateway |
| Project Wolverine (Coming Soon) | Self-healing K8s infrastructure with AI incident response | EKS, Terraform, Prometheus, Grafana, Claude API, GitHub Actions |

---

## 👨‍💻 Author

**Sumukh Pitre**
MS Informatics (Cloud Concentration) | Northeastern University, Boston


---

## 📄 License

This repository is for educational purposes. Notes are written in my own words based on hands-on learning.

MIT License — see [LICENSE](./LICENSE) for details.