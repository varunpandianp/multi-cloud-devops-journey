## CI/CD - Jenkins

Jenkins is a CI/CD automation server used to automatically build, test, package, and deploy applications.

### 1. What is CI?

Continuous Integration (CI) means developers frequently push code to a shared repository.

Jenkins automatically performs tasks such as:

Developer
↓
Git / GitHub
↓
Jenkins
↓
Build
↓
Test
↓
Code Quality / Security Checks
↓
Artifact


### 2. What is CD?

Continuous Delivery / Continuous Deployment (CD) automates the process of releasing and deploying applications.

Artifact
↓
Deploy
↓
Development / QA
↓
Staging
↓
Production


### 3. What is Jenkins?

Jenkins is an open-source automation server mainly used to implement CI/CD pipelines.

Jenkins can integrate with:

- Git / GitHub
- Maven
- Gradle
- Docker
- Kubernetes
- Terraform
- Ansible
- AWS
- SonarQube
- 
- ## Example
  Jenkins
  │
  ├── Git → Get source code
  ├── Maven → Build application
  ├── JUnit → Run tests
  ├── SonarQube → Code quality
  ├── Docker → Build image
  ├── Registry → Store image
  └── Kubernetes/AWS → Deploy

### 4. Jenkins in a CI/CD Pipeline

A typical CI/CD pipeline looks like:

Developer
↓
Git Push
↓
GitHub
↓
Webhook
↓
Jenkins
↓
Checkout Code
↓
Build
↓
Test
↓
Code Quality / Security Scan
↓
Package
↓
Docker Image
↓
Container Registry
↓
Deploy
↓
Kubernetes / AWS


### 5. Jenkins and Build Tools 

Jenkins is not a build tool.

Jenkins is an automation/orchestration tool that can execute build tools such as Maven and Gradle.

Example:

Jenkins
↓
Maven
↓
Compile
↓
Test
↓
Package
↓
JAR / WAR


### 6. Jenkins Controller Important Jenkins Components

The Jenkins Controller is responsible for managing Jenkins.

Responsibilities:

- Manage jobs and pipelines
- Schedule builds
- Manage credentials
- Store Jenkins configuration
- Assign work to agents


### 7. Jenkins Agent

A Jenkins Agent is a machine that performs the actual build and deployment tasks.

Example:

Jenkins Controller
↓
Jenkins Agent
↓
Maven Build
↓
Testing
↓
Docker Build


### 8. Jenkins Job

A Jenkins Job is a task configured in Jenkins.

Example:

Build Java Application

A job can perform tasks such as:

- Checkout source code
- Run Maven
- Run tests
- Build Docker image
- Deploy application


### 9. Jenkins Pipeline

A Jenkins Pipeline defines the complete CI/CD workflow.

Example:

Checkout
↓
Build
↓
Test
↓
Security Scan
↓
Package
↓
Docker Build
↓
Push Image
↓
Deploy


### 10. Jenkinsfile

   A Jenkinsfile is a text file that defines the Jenkins pipeline.
   Typical structure:

   Jenkinsfile
   │
   ├── Checkout
   ├── Build
   ├── Test
   ├── Security Scan
   ├── Package
   ├── Docker Build
   ├── Push
   └── Deploy


A Jenkinsfile is a file that defines the Jenkins Pipeline as code.

Example:

pipeline {
stages {

        stage('Checkout') {
            steps {
                git 'https://github.com/example/app.git'
            }
        }

        stage('Build') {
            steps {
                sh 'mvn package'
            }
        }

        stage('Test') {
            steps {
                sh 'mvn test'
            }
        }
    }
}


### 11. Pipeline as Code

When the Jenkins pipeline is written inside a Jenkinsfile and stored in Git, it is called Pipeline as Code.

Example:

Application Repository
↓
Jenkinsfile
↓
GitHub
↓
Jenkins
↓
Pipeline


Benefits:

- Version controlled
- Easy to review
- Easy to modify
- Reproducible
- Pipeline changes can be tracked using Git


### 12. Jenkins Triggers


A trigger tells Jenkins when to start a job or pipeline.

Common triggers:

1. Webhook
2. Poll SCM
3. Scheduled Build
4. Manual Trigger


### 13. Jenkins Webhook

A webhook allows GitHub to notify Jenkins when an event occurs.

Example:

Developer
↓
git push
↓
GitHub
↓
Webhook
↓
Jenkins
↓
Pipeline Starts

## Poll SCM
Jenkins periodically checks the repository for changes.
Scheduled build
Example:
Every night at 12 AM
↓
Jenkins
↓
Run pipeline


### 14. Jenkins Credentials

Sensitive information should not be hard-coded inside Jenkinsfiles.

Examples:

- Passwords
- API tokens
- SSH keys
- Cloud credentials

Instead:

Jenkins Credentials
↓
Pipeline
↓
Securely Access Secret


### 15. Jenkins + Docker

Jenkins can automate Docker image creation.

Example:

GitHub
↓
Jenkins
↓
Build Application
↓
Docker Build
↓
Docker Image
↓
Container Registry


Example commands:

docker build -t myapp:1.0 .
docker push myregistry/myapp:1.0


### 16. Jenkins + Kubernetes

Jenkins can also automate application deployment to Kubernetes.

Example:

GitHub
↓
Jenkins
↓
Build
↓
Test
↓
Docker Image
↓
Container Registry
↓
Kubernetes
↓
Pods
↓
Application


Deployment tools can include:

- kubectl
- Helm
- Argo CD


### 17. Jenkins + Maven

Jenkins can execute Maven commands.

Example:

mvn clean package

Maven performs the build process:

Source Code
↓
Compile
↓
Test
↓
Package
↓
JAR / WAR


Jenkins controls when and how Maven is executed.


### 18. Typical Production CI/CD Architecture

Developer
↓
GitHub
↓
Webhook
↓
Jenkins Controller
↓
Jenkins Agent
↓
Build
↓
Test
↓
Security Scan
↓
Docker Build
↓
Container Registry
↓
Kubernetes / AWS
↓
Application


### 19. Important Jenkins Topics to Learn

For DevOps roles, focus on:

- Jenkins Architecture
- Controller vs Agent
- Jobs
- Freestyle Jobs
- Pipeline
- Declarative Pipeline
- Jenkinsfile
- Stages and Steps
- Git Integration
- Webhooks
- Credentials
- Environment Variables
- Parameters
- Artifacts
- Maven Integration
- Docker Integration
- Multibranch Pipeline
- Shared Libraries
- Jenkins Agents
- Jenkins + Kubernetes
- Pipeline Troubleshooting


### 20. Simple Interview Definition

Jenkins is an open-source automation server used to implement CI/CD pipelines by automating tasks such as building, testing, packaging, and deploying applications.


### 21. Simple Mental Model

GitHub
↓
Jenkins
↓
Build
↓
Test
↓
Package
↓
Docker
↓
Registry
↓
Deploy

Jenkins = Automation / Orchestration

Maven = Build Tool

Docker = Containerization

Kubernetes = Container Orchestration

GitHub = Source Code Management