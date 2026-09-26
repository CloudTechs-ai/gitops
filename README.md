# CloudTechs GitOps

**Production-style GitOps deployment platform for a cloud-native healthcare/pharma application running on AWS EKS.**

This repository manages the **Kubernetes deployment lifecycle** for the CloudTechs platform using **ArgoCD, Helm, GitHub Actions, Amazon ECR, and AWS EKS**.

The platform follows a **GitOps / Infrastructure-as-Code architecture** where application code, infrastructure, and deployment configuration are separated into independently managed repositories.

> **Core stack:** AWS EKS · Kubernetes · Helm · ArgoCD · GitHub Actions · Amazon ECR · Terraform · Docker · PostgreSQL

---

## ☁️ Cloud-Native Architecture

```text
                                  USERS
                                    │
                                    ▼
                         ┌────────────────────┐
                         │    AWS Load        │
                         │    Balancer        │
                         └─────────┬──────────┘
                                   │
                                   ▼
                         ┌────────────────────┐
                         │    AWS EKS         │
                         │                    │
                         │  API Gateway       │
                         │       │            │
                         │  ┌────┼────────┐   │
                         │  ▼    ▼        ▼   │
                         │ Auth  Catalog  Inventory
                         │                    │
                         │ Manufacturing      │
                         │ Supplier           │
                         │ Notifications      │
                         │                    │
                         │ Pharma Frontend    │
                         └─────────┬──────────┘
                                   │
                                   ▼
                            Amazon RDS
                            PostgreSQL


              ┌──────────────────────────────────────┐
              │            DELIVERY PIPELINE          │
              │                                      │
              │ Developer                            │
              │    │                                 │
              │    ▼                                 │
              │ GitHub                               │
              │    │                                 │
              │    ▼                                 │
              │ GitHub Actions                       │
              │    │                                 │
              │    ├── Tests                          │
              │    ├── SAST                           │
              │    ├── Dependency Scanning            │
              │    ├── Docker Build                   │
              │    └── Container Security             │
              │    │                                 │
              │    ▼                                 │
              │ Amazon ECR                            │
              │    │                                 │
              │    ▼                                 │
              │ GitOps Repository                    │
              │    │                                 │
              │    ▼                                 │
              │ ArgoCD                               │
              │    │                                 │
              │    ▼                                 │
              │ AWS EKS                              │
              └──────────────────────────────────────┘
```

---

# 🚀 What This Repository Demonstrates

This project was designed to demonstrate practical **Cloud Engineering, DevOps, Kubernetes, and SRE practices** rather than simply deploying containers to a cluster.

### Kubernetes

* AWS EKS application deployment
* Kubernetes namespaces and workloads
* Health checks and container lifecycle management
* Service discovery and networking
* Declarative Kubernetes configuration
* Environment-specific configuration

### GitOps

* ArgoCD continuous delivery
* Git as the source of truth
* Declarative desired state
* Automated synchronization
* Configuration drift detection
* Auditable deployment history
* Git-based environment promotion

### Cloud / AWS

* Amazon EKS
* Amazon ECR
* Amazon RDS PostgreSQL
* AWS IAM
* AWS Load Balancing
* GitHub OIDC → AWS IAM authentication

### CI/CD

* GitHub Actions
* Automated testing
* Docker image builds
* Security scanning
* Immutable image tagging
* Automated GitOps configuration updates
* Development environment deployment

### Security

* Gitleaks secret scanning
* CodeQL SAST
* Semgrep
* OWASP dependency scanning
* Trivy container scanning
* Cosign image signing
* GitHub OIDC authentication
* No long-lived AWS access keys in CI

---

# 🏗️ GitOps Workflow

The deployment model follows a **build once, deploy through GitOps** approach.

```text
Developer
    │
    │ git push
    ▼
Application Repository
    │
    ▼
GitHub Actions
    │
    ├── Unit Tests
    ├── Integration Tests
    ├── SAST
    ├── Dependency Scan
    ├── Docker Build
    └── Trivy Scan
    │
    ▼
Amazon ECR
    │
    │ image: sha-28fb144
    ▼
GitOps Repository
    │
    │ update image tag
    ▼
ArgoCD
    │
    │ reconcile
    ▼
AWS EKS
    │
    ▼
Running Application
```

Every deployed version can be traced through:

```text
Source Commit
     ↓
CI Build
     ↓
Container Image
     ↓
GitOps Commit
     ↓
ArgoCD Sync
     ↓
Kubernetes Deployment
```

This creates a clear deployment audit trail from **source code to production workload**.

---

# ⛵ ArgoCD

ArgoCD is the continuous delivery engine for the Kubernetes platform.

Instead of manually deploying workloads with `kubectl`, ArgoCD continuously compares the Kubernetes cluster against the desired state stored in Git.

```text
             Git
              │
              ▼
          Desired State
              │
              ▼
           ArgoCD
              │
              ▼
          AWS EKS
              │
              ▼
         Actual State
              │
              │
              └──── Reconciliation ────► Git State
```

### ArgoCD manages

* Application deployments
* Helm releases
* Environment configuration
* Kubernetes manifests
* Application health
* Deployment synchronization
* Configuration drift

This allows the Kubernetes environment to be managed declaratively instead of relying on manual deployment commands.

---

# ⛵ Helm-Based Application Deployment

The platform uses a **shared Helm chart** to standardize Kubernetes deployments across services.

```text
                     Shared Helm Chart
                            │
          ┌─────────────────┼─────────────────┐
          │                 │                 │
          ▼                 ▼                 ▼
     Auth Service      Inventory         Supplier
          │                 │                 │
          ▼                 ▼                 ▼
     values-auth      values-inventory   values-supplier
          │                 │                 │
          └─────────────────┼─────────────────┘
                            ▼
                       AWS EKS
```

This avoids maintaining duplicated Kubernetes manifests for every application.

Service-specific configuration can define:

* Container image
* Image tag
* Replicas
* Ports
* Environment variables
* Resource requests/limits
* Liveness probes
* Readiness probes
* Service configuration
* Ingress configuration
* Secret references

---

# 📁 Repository Structure

```text
gitops/
│
├── argocd/
│   └── applications/
│       ├── api-gateway.yaml
│       ├── auth-service.yaml
│       ├── drug-catalog-service.yaml
│       ├── inventory-service.yaml
│       ├── manufacturing-service.yaml
│       ├── supplier-service.yaml
│       ├── notification-service.yaml
│       └── pharma-ui.yaml
│
├── envs/
│   └── dev/
│       ├── values-api-gateway.yaml
│       ├── values-auth-service.yaml
│       ├── values-drug-catalog-service.yaml
│       ├── values-inventory-service.yaml
│       ├── values-manufacturing-service.yaml
│       ├── values-supplier-service.yaml
│       ├── values-notification-service.yaml
│       └── values-pharma-ui.yaml
│
├── helm-charts/
│   └── microservice/
│       ├── templates/
│       ├── Chart.yaml
│       └── values.yaml
│
├── k8s/
│   └── ...
│
├── db-init/
│   └── ...
│
└── README.md
```

---

# 🌎 Environment Management

Environment configuration is separated from the reusable Helm templates.

Example:

```text
envs/
└── dev/
    ├── values-api-gateway.yaml
    ├── values-auth-service.yaml
    ├── values-inventory-service.yaml
    └── values-pharma-ui.yaml
```

A CI pipeline can update an individual service without modifying the Helm chart itself.

Example:

```yaml
image:
  repository: <aws-account>.dkr.ecr.<region>.amazonaws.com/auth-service
  tag: sha-28fb144
```

The resulting Git commit becomes the declarative deployment record for that version.

---

# 🔐 Security & Supply Chain

Security is integrated into the deployment lifecycle rather than treated as a separate process.

## CI Security

Application repositories run:

* **Gitleaks** — secret detection
* **CodeQL** — static application security testing
* **Semgrep** — security and OWASP rules
* **OWASP Dependency-Check** — dependency vulnerability analysis
* **Trivy** — container vulnerability scanning

## Container Security

Images are:

1. Built using Docker
2. Scanned for vulnerabilities
3. Tagged using immutable commit identifiers
4. Signed using Cosign
5. Published to Amazon ECR
6. Referenced by the GitOps repository

Example:

```text
auth-service:sha-28fb144
```

instead of relying on mutable tags such as:

```text
auth-service:latest
```

## AWS Authentication

CI authenticates to AWS using:

```text
GitHub Actions
      │
      ▼
GitHub OIDC
      │
      ▼
AWS IAM Role
      │
      ▼
AWS Resources
```

This eliminates the need to store long-lived AWS access keys inside GitHub Actions secrets.

---

# 🗄️ Database Initialization

The `db-init/` directory contains Kubernetes resources required for database initialization and supporting platform setup.

Database-related initialization is separated from application deployment configuration so that foundational resources can be managed independently from application releases.

---

# ☸️ Kubernetes Platform Resources

The `k8s/` directory contains Kubernetes resources that sit outside the reusable application Helm chart.

These can include:

* Namespaces
* Platform-level resources
* Supporting configuration
* Shared Kubernetes resources
* Database-related resources

This separation keeps the shared application chart focused on reusable workloads while allowing platform-specific Kubernetes resources to be managed independently.

---

# 🔄 Environment Promotion

The platform uses Git-based promotion between environments.

```text
                Application Change
                        │
                        ▼
                   GitHub Actions
                        │
                        ▼
                    DEV Deploy
                        │
                        ▼
                  Validation / QA
                        │
                        ▼
                 Promotion Change
                        │
                        ▼
                      PROD
```

Production promotion is intentionally separated from automatic development deployment, providing an explicit control point before production changes.

---

# 🧩 Repository Ecosystem

CloudTechs separates infrastructure, application code, frontend code, and deployment configuration.

```text
┌──────────────────────────────────────────────────────────┐
│                    CloudTechs Platform                   │
└──────────────────────────────────────────────────────────┘

        ┌──────────────┐
        │  med-infra   │
        │              │
        │  Terraform   │
        │  AWS / EKS   │
        │  IAM / ECR   │
        │  RDS         │
        └──────┬───────┘
               │
               │ Infrastructure
               ▼
        ┌──────────────┐
        │     AWS      │
        └──────┬───────┘
               │
               │ Runtime
               ▼
        ┌──────────────┐
        │     EKS      │
        └──────┬───────┘
               ▲
               │
        ┌──────┴───────┐
        │    ArgoCD    │
        └──────▲───────┘
               │
               │ GitOps
               │
        ┌──────┴─────────────┐
        │       gitops       │
        │                    │
        │ Helm + ArgoCD      │
        │ Environment Values │
        │ Kubernetes Config  │
        └──────▲─────────────┘
               │
               │ Image + Deployment Update
               │
        ┌──────┴─────────────┐
        │ med-pharma-backend │
        │                    │
        │ Spring Boot        │
        │ Node.js            │
        │ Docker             │
        └─────────┬──────────┘
                  │
                  ▼
              Amazon ECR
```

### Companion Repositories

| Repository              | Responsibility                                                     |
| ----------------------- | ------------------------------------------------------------------ |
| **med-infra**           | Terraform-managed AWS infrastructure                               |
| **med-pharma-backend**  | Java/Spring Boot and Node.js microservices                         |
| **med-pharma-frontend** | React frontend                                                     |
| **gitops**              | Helm, ArgoCD, Kubernetes, and environment deployment configuration |

---

# 🛠️ Technology Stack

| Category       | Technologies                            |
| -------------- | --------------------------------------- |
| Cloud          | **AWS**                                 |
| Kubernetes     | **Amazon EKS**                          |
| GitOps         | **ArgoCD**                              |
| Packaging      | **Helm**                                |
| CI/CD          | **GitHub Actions**                      |
| Registry       | **Amazon ECR**                          |
| Infrastructure | **Terraform**                           |
| Containers     | **Docker**                              |
| Backend        | **Java 17 / Spring Boot / Node.js**     |
| Database       | **PostgreSQL / Amazon RDS**             |
| Security       | **CodeQL / Semgrep / Gitleaks / Trivy** |
| Supply Chain   | **Cosign / Sigstore**                   |
| Identity       | **GitHub OIDC / AWS IAM**               |

---

# 💡 Engineering Principles

### Declarative Infrastructure

Infrastructure and deployment state are represented as code rather than manual configuration.

### Git as the Source of Truth

Changes to deployment state are committed to Git and become part of the platform's audit trail.

### Immutable Artifacts

Deployments reference immutable commit-based container images.

### Automated Reconciliation

ArgoCD continuously works to make the Kubernetes cluster match the desired Git state.

### Separation of Concerns

Infrastructure, application source code, and deployment configuration are managed independently.

### Reusable Deployment Patterns

Shared Helm templates provide a consistent deployment model across services.

### Security by Default

Security scanning and supply-chain controls are incorporated directly into CI/CD.

---

# 📊 What This Project Demonstrates

This project demonstrates practical experience with:

**AWS Cloud Engineering**

→ EKS
→ ECR
→ RDS
→ IAM
→ Load Balancing

**Kubernetes**

→ Deployments
→ Services
→ Namespaces
→ Health Probes
→ Helm
→ Configuration Management

**DevOps / CI/CD**

→ GitHub Actions
→ Docker
→ Automated Testing
→ Security Scanning
→ Immutable Artifacts

**GitOps / SRE**

→ ArgoCD
→ Declarative Deployments
→ Reconciliation
→ Drift Detection
→ Environment Promotion
→ Deployment Auditability

**Cloud Security**

→ OIDC
→ IAM
→ SAST
→ Dependency Scanning
→ Container Scanning
→ Image Signing

---

# 🚀 Platform Outcome

The resulting architecture provides a repeatable deployment path from **developer commit to running Kubernetes workload**:

```text
Developer
    ↓
GitHub
    ↓
GitHub Actions
    ↓
Security + Testing
    ↓
Docker
    ↓
Amazon ECR
    ↓
GitOps Repository
    ↓
ArgoCD
    ↓
AWS EKS
    ↓
Cloud-Native Application
```

**Infrastructure is provisioned with Terraform.
Applications are packaged with Docker.
Deployments are defined with Helm.
Git is the source of truth.
ArgoCD reconciles Kubernetes.
AWS EKS runs the platform.**
