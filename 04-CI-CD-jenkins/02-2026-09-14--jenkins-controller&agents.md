## Jenkins - Controller and Agent

### 1. What is Jenkins Controller?

The Jenkins Controller is the main Jenkins server that manages the CI/CD system.

It is responsible for:

- Managing Jenkins configuration
- Managing jobs and pipelines
- Scheduling builds
- Managing credentials
- Managing plugins
- Assigning jobs to agents
- Monitoring agents
- Storing build information


### 2. What is Jenkins Agent?

A Jenkins Agent is a machine that performs the actual work assigned by the Jenkins Controller.

Agents can execute:

- Build
- Test
- Code analysis
- Docker commands
- Deployment commands
- Other CI/CD tasks


### 3. Controller and Agent Architecture

The basic architecture is:

Developer
↓
GitHub
↓
Jenkins Controller
↓
Jenkins Agent
↓
Build / Test / Package
↓
Deploy


The Controller manages the work.

The Agent performs the work.


### 4. Why Do We Need Jenkins Agents?

If all builds run on the Jenkins Controller, the controller can become overloaded.

Instead, Jenkins can distribute workloads across multiple agents.

Example:

Jenkins Controller
│
├── Agent 1 → Java / Maven
│
├── Agent 2 → Docker
│
├── Agent 3 → Python
│
└── Agent 4 → Kubernetes


This allows Jenkins to run different workloads on different machines.


### 5. Multiple Agents

A Jenkins environment can have multiple agents.

Example:

                    Jenkins Controller
                           │
          ┌────────────────┼────────────────┐
          ↓                ↓                ↓
       Agent 1          Agent 2          Agent 3
        Maven            Docker          Kubernetes
          │                │                │
       Build/Test       Build Image       Deploy


The Controller decides where the pipeline stages should run.


### 6. Labels

Labels are used to identify agents based on their capabilities.

Example:

Agent 1:
label = linux

Agent 2:
label = docker

Agent 3:
label = kubernetes


A pipeline can request a specific agent using its label.


### 7. Example

Suppose we have:

Agent 1 → Linux + Maven
Agent 2 → Docker

Pipeline:

Checkout
↓
Build with Maven
↓
Docker Build
↓
Deploy


Jenkins can run:

Build Stage → Agent 1
Docker Stage → Agent 2


### 8. Jenkins Controller vs Agent

Controller:

- Manages Jenkins
- Schedules jobs
- Manages pipelines
- Assigns work
- Stores Jenkins configuration
- Monitors agents

Agent:

- Executes jobs
- Runs builds
- Runs tests
- Builds Docker images
- Performs deployment tasks


### 9. Controller and Agent Communication

The Jenkins Controller communicates with agents to send tasks and receive results.

Common connection methods include:

- SSH
- Inbound Agent
- WebSocket


Example:

Jenkins Controller
↓
Connects to Agent
↓
Sends Pipeline Work
↓
Agent Executes Work
↓
Result sent back to Controller


### 10. Static vs Dynamic Agents

#### Static Agent

A machine is permanently configured as a Jenkins Agent.

Example:

Jenkins Controller
↓
EC2 Instance
↓
Jenkins Agent


The EC2 instance remains available for Jenkins.


#### Dynamic Agent

Agents are created when required and removed after the work is completed.

Example:

Jenkins
↓
Create Agent
↓
Run Build
↓
Build Complete
↓
Destroy Agent


Dynamic agents are useful for scalable CI/CD environments.


### 11. Jenkins Agent in AWS

Agents can run on AWS infrastructure.

Example:

Jenkins Controller
↓
AWS EC2 Agent
↓
Build / Test
↓
Docker Image
↓
Container Registry


Agents can also be provisioned dynamically using cloud infrastructure.


### 12. Important Production Practice

The Jenkins Controller should generally not perform heavy build workloads.

Instead:

Jenkins Controller
↓
Schedules / Manages
↓
Jenkins Agents
↓
Perform Builds


This keeps the Jenkins Controller available for managing the Jenkins environment.


### 13. Old Terminology

Older Jenkins terminology:

Master
Slave

Modern terminology:

Controller
Agent


Master-Slave = Controller-Agent

In interviews, if someone says "Jenkins Master-Slave architecture", understand that they are referring to the Controller-Agent architecture.


### 14. Simple Mental Model

Controller = Manager

Agent = Worker


Controller:
"Run this pipeline."

Agent:
"I will execute the pipeline."


### 15. Interview Answer

Jenkins Controller is the central component that manages Jenkins, schedules jobs, and assigns workloads to agents. Jenkins Agents are machines that execute the actual build, test, and deployment tasks. Multiple agents allow Jenkins to distribute workloads and scale CI/CD execution.