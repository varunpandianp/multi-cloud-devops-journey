# DevOps Lifecycle and CI/CD Pipeline

## Big Picture

A typical production DevOps pipeline:

```text
Developer
   ↓
Git / GitHub
   ↓
CI: Build, Test, Code Quality, Security Scan
   ↓
Artifact / Container Registry
   ↓
CD: Deploy
   ↓
Cloud Infrastructure
   ↓
Application Running
   ↓
Monitoring and Logging
   ↓
Feedback to Developer
```

**Core idea:** DevOps creates a continuous feedback loop between Development and Operations.

---

## 1. Development and Source Control

Developers write application code, for example:

```text
Java
Python
Node.js
React
```

They store and manage code using:

```text
Developer
   ↓
Git
   ↓
GitHub
```

Git tracks code changes. GitHub stores the source code and supports collaboration through branches, pull requests, and code reviews.

---

## 2. CI: Continuous Integration

CI automatically checks code when a developer pushes changes.

```text
git push
   ↓
GitHub
   ↓
CI Pipeline
   ↓
Build
   ↓
Test
   ↓
Code Quality Check
   ↓
Security Scan
```

Typical CI tools:

```text
GitHub Actions
Jenkins
GitLab CI
Azure DevOps
```

### Java example

```text
GitHub
   ↓
Jenkins
   ↓
Maven
   ↓
Build
   ↓
Tests
```

### Python example

```text
GitHub
   ↓
GitHub Actions
   ↓
pytest
   ↓
Build and Test
```

### React example

```text
GitHub
   ↓
GitHub Actions
   ↓
npm
   ↓
Build
   ↓
Test
```

**Important:** Maven is not a CI tool. Maven is a Java build and dependency-management tool.

---

## 3. Artifacts and Containers

After successful CI, the pipeline produces something deployable.

Traditional application artifacts:

```text
JAR
WAR
Python package
Node package
```

Containerized application artifact:

```text
Docker image
```

Artifacts and Docker images are stored in registries:

```text
AWS ECR
Docker Hub
GitHub Container Registry
Azure Container Registry
```

Example:

```text
Source Code
   ↓
Build
   ↓
Docker Image
   ↓
AWS ECR
```

---

## 4. CD: Continuous Delivery and Continuous Deployment

CD moves a tested artifact into application environments.

```text
Artifact
   ↓
CD Pipeline
   ↓
Development
   ↓
Test
   ↓
Staging
   ↓
Production
```

Common CD tools:

```text
GitHub Actions
Jenkins
GitLab CI/CD
Argo CD
AWS CodePipeline
```

Goal:

```text
Deliver software reliably and repeatedly to environments.
```

---

## 5. Where Cloud Fits

Cloud provides the infrastructure and services where applications and DevOps systems run.

Example using AWS:

```text
AWS
├── VPC
├── EC2
├── EKS
├── ECR
├── RDS
├── Load Balancer
└── Security Groups
```

```text
VPC → network environment
EC2 → virtual servers
EKS → managed Kubernetes
ECR → Docker image registry
RDS → managed database
```

```text
DevOps = practices, processes, automation, and culture
Cloud  = infrastructure and services used to build and run systems
```

Cloud and DevOps work closely together, but they are not the same thing.

---

## 6. Terraform: Infrastructure as Code

Terraform creates infrastructure through code instead of manually creating resources in the cloud console.

Terraform can create:

```text
VPC
Subnets
EC2
Load Balancer
Security Groups
EKS
RDS
```

Typical Terraform flow:

```text
Terraform Code
   ↓
GitHub
   ↓
CI/CD
   ↓
terraform plan
   ↓
terraform apply
   ↓
AWS Infrastructure
```

**Terraform = Infrastructure as Code (IaC).**

---

## 7. Ansible: Configuration Automation

Terraform mainly provisions infrastructure. Ansible configures systems after they exist.

```text
Terraform
   ↓
Create EC2
   ↓
Ansible
   ↓
Install Nginx
Install Docker
Configure Application
```

```text
Terraform → creates infrastructure
Ansible   → configures servers and applications
```

---

## 8. Docker: Application Packaging

Docker packages the application with its dependencies and runtime.

```text
Application
+ Dependencies
+ Runtime
   ↓
Docker Image
```

Example:

```text
React Application
   ↓
Docker Image
   ↓
AWS ECR
   ↓
EKS
```

---

## 9. Kubernetes: Container Orchestration

Kubernetes manages containers at scale.

```text
Docker Container
   ↓
Kubernetes
   ↓
Pods
```

Kubernetes handles:

```text
Scheduling
Scaling
Service discovery
Self-healing
Rolling deployments
```

AWS EKS is AWS’s managed Kubernetes service.

---

## 10. Monitoring and Logging

Deployment is not the end of the DevOps process.

```text
Application
   ↓
Metrics, Logs, Traces
   ↓
Monitoring
   ↓
Alerts and Feedback
```

Common tools:

```text
Prometheus
Grafana
ELK / OpenSearch
CloudWatch
Datadog
```

Monitoring answers the important question:

```text
Is the application actually working?
```

---

# Complete DevOps Pipeline

```text
Development
   ↓
GitHub
   ↓
CI
├── Build
├── Unit Tests
├── Code Quality
└── Security Scan
   ↓
Artifact / Docker Image
   ↓
Registry
   ↓
CD
├── Deploy
├── Test
└── Release
   ↓
Cloud
├── EC2
├── EKS
└── RDS
   ↓
Application
   ↓
Monitoring and Logging
   ↓
Feedback
   ↓
Development
```

---

## CI/CD: The Bridge Between Development and Operations

```text
Development
   ↓
Code
   ↓
Git
   ↓
CI/CD
   ↓
Build, Test, Deploy
   ↓
Operations
   ↓
Production
```

CI/CD is a major automation bridge between Development and Operations.

However:

```text
DevOps is bigger than CI/CD.
CI/CD is one major practice inside DevOps.
```

---

# CI/CD Skill Levels

## Beginner

```text
Git
   ↓
CI
   ↓
Build
   ↓
Test
```

## Intermediate

```text
Git
   ↓
CI
   ↓
Test
   ↓
Docker Build
   ↓
Registry
   ↓
Deploy
```

## Advanced

```text
Git
   ↓
CI
├── Unit Tests
├── SAST
├── Dependency Scan
├── Docker Build
├── Image Scan
└── Push Image
   ↓
ECR
   ↓
Kubernetes
   ↓
Deployment
   ↓
Monitoring
```

## Production-Level Knowledge

```text
Branching strategy
Pull requests
Code reviews
Secrets management
IAM
Artifact versioning
Environment promotion
Blue/Green deployment
Canary deployment
Rolling deployment
Rollback
Infrastructure as Code
Observability
Security
Cost management
Disaster recovery
```

---

# Tools and Their Roles

| Problem | Solution | Tools |
|---|---|---|
| Store code | Version control | Git, GitHub |
| Build Java | Build automation | Maven |
| Build Python | Packaging and build tools | pip, Poetry, uv |
| Build React | JavaScript build tooling | npm, Vite |
| Automate CI/CD | Pipeline automation | GitHub Actions, Jenkins |
| Package application | Containers | Docker |
| Store images | Container registry | AWS ECR |
| Create infrastructure | Infrastructure as Code | Terraform |
| Configure servers | Automation | Ansible |
| Run containers at scale | Orchestration | Kubernetes, EKS |
| Route traffic | Reverse proxy or load balancing | Nginx, ALB |
| Monitor systems | Metrics | Prometheus, CloudWatch |
| Visualize metrics | Dashboards | Grafana |
| Centralize logs | Logging | OpenSearch, ELK |
| Manage access | Identity and access management | AWS IAM |
| Secure secrets | Secret management | AWS Secrets Manager |

---

# Priority Learning Path

```text
1. Linux
   ↓
2. Networking
   ↓
3. Git and GitHub
   ↓
4. CI/CD
   ↓
5. Docker
   ↓
6. AWS
   ↓
7. Terraform
   ↓
8. Kubernetes
   ↓
9. Ansible
   ↓
10. Monitoring and Observability
   ↓
11. DevSecOps
```

Do not learn these as isolated tools. Learn how they connect in one project.

---

# End-to-End DevOps Project Architecture

```text
Developer
   ↓
GitHub
   ↓
GitHub Actions
   ↓
Tests
   ↓
Docker Build
   ↓
Security Scan
   ↓
AWS ECR
   ↓
Terraform
   ↓
AWS Infrastructure
   ↓
EKS
   ↓
Application
   ↓
ALB
   ↓
Prometheus + Grafana + CloudWatch
   ↓
Alerts and Logs
```

This kind of end-to-end project demonstrates much more than only saying:

```text
“I know Jenkins, Docker, Terraform, and Kubernetes.”
```

---

# Main Learning Goal

```text
Understand how code moves from a developer’s laptop
to Git
to CI
to a tested artifact
to cloud infrastructure
to production
to monitoring
to feedback.
```

The goal is to build, troubleshoot, and improve that complete flow.