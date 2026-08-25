#25-08-2026

DevOps Fundamentals — Learning Notes

Date: 25 August 2026

**1. DevOps Fundamentals**

Today I learned about the overall DevOps ecosystem and the purpose of different tools used throughout the software development and delivery lifecycle.

Tools Introduced
Maven — Build automation and dependency management for Java projects
Terraform — Infrastructure as Code (IaC)
GitHub Actions — CI/CD automation
Ansible — Configuration management and automation

The main idea is that different tools solve different problems across the software delivery lifecycle.

**2. SDLC and Agile**

Learned about the Software Development Life Cycle (SDLC) and the Agile development model.

SDLC

The typical software development lifecycle includes:

Planning
Requirements
Design
Development
Testing
Deployment
Maintenance
Agile

Agile is an iterative approach to software development where work is divided into smaller increments and delivered continuously.

**Traditional Development vs DevOps**

I learned that DevOps extends the development process by bringing development and operations closer together.

Instead of development and operations working as separate teams, DevOps encourages:

Collaboration
Automation
Continuous Integration
Continuous Delivery
Continuous Testing
Infrastructure as Code
Monitoring and feedback

**3. Infrastructure as Code — IaC**

Learned the concept of Infrastructure as Code (IaC).

IaC allows infrastructure to be created and managed using configuration/code instead of manually creating resources through a GUI.

Example

Instead of manually creating an AWS EC2 instance:

AWS Console
↓
Create EC2 manually

With IaC:

Terraform Code
↓
Terraform
↓
AWS Infrastructure

Terraform is one of the important IaC tools I am learning for DevOps.

**4. SaaS and PaaS**

Learned the basics of cloud service models:

SaaS — Software as a Service

The complete software application is provided as a service.

Examples: Gmail, Google Docs, Salesforce

PaaS — Platform as a Service

A platform is provided so developers can deploy applications without managing the underlying infrastructure.

The cloud provider manages much of the infrastructure and platform.

**5. Web Server and Application Server**

Learned the difference between a Web Server and an Application Server.

Web Server

A web server primarily handles HTTP/HTTPS requests and serves static content.

Examples:

Nginx
Apache HTTP Server
Application Server

An application server runs application/business logic and processes dynamic requests.

Examples include application runtimes such as:

Tomcat for Java applications
Gunicorn/Uvicorn commonly used with Python applications

**6. Reverse Proxy and Forward Proxy
   Forward Proxy**

A forward proxy sits between a client and the internet.

Client
↓
Forward Proxy
↓
Internet

The proxy makes requests on behalf of the client.

Reverse Proxy

A reverse proxy sits between clients and backend servers.

Client
↓
Reverse Proxy
↓
Backend Servers

Common uses include:

Load balancing
SSL/TLS termination
Routing requests
Caching
Security
Hiding backend servers

Nginx is commonly used as a reverse proxy.

Software Development Life Cycle

The traditional Software Development Life Cycle can be viewed as:

Planning
↓
Requirements
↓
Design
↓
Development
↓
Testing
↓
Deployment
↓
Maintenance

_**DevOps Life Cycle**_

DevOps extends the traditional development process by continuously connecting development, operations, automation, deployment and monitoring.

        PLAN
          ↓
        CODE
          ↓
        BUILD
          ↓
        TEST
          ↓
       RELEASE
          ↓
       DEPLOY
          ↓
       OPERATE
          ↓
       MONITOR
          ↓
       FEEDBACK
          ↺
**Example DevOps Toolchain**

A typical application delivery workflow can look like:

Developer
↓
Git / GitHub
↓
Build
↓
Maven / Gradle / npm / pip
↓
Testing
↓
Jenkins / GitHub Actions
↓
Code Quality
↓
SonarQube
↓
Artifact
↓
Nexus / JFrog Artifactory
↓
Docker
↓
Container Image
↓
Container Registry
↓
AWS / Azure / GCP
↓
Kubernetes
↓
Deploy
↓
Prometheus + Grafana
↓
Monitor & Feedback

**Multi-Cloud DevOps**

As part of my Multi-Cloud DevOps Journey, I am also learning how DevOps practices can be applied across multiple cloud platforms.

                    DevOps
                       │
        ┌──────────────┼──────────────┐
        ↓              ↓              ↓
       AWS            Azure           GCP
        │              │              │
        └──────────────┼──────────────┘
                       ↓
                 Terraform
                       ↓
                Infrastructure
                       ↓
             Docker / Kubernetes
                       ↓
               CI/CD Automation
                       ↓
              Monitoring & Logging

The goal is to understand how common DevOps tools and practices can automate the complete software delivery process across AWS, Azure and Google Cloud Platform (GCP).

Key Takeaway

Different DevOps tools solve different problems. The important part is not just learning individual tools, but understanding where each tool fits into the overall software delivery and DevOps lifecycle and how they work together to automate application delivery.