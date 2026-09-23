# Jenkins - Session 2
## Pipelines as Code and Distributed Builds

Session 2 covers:

- Pipeline concepts and vocabulary
- Declarative Pipeline
- Scripted Pipeline
- Declarative vs Scripted Pipeline
- Multibranch Pipelines
- Webhooks and SCM integration
- Controller and Agent architecture
- Agent connection methods
- Configuring Jenkins Agent nodes
- Targeting specific agents
- Distributed builds


## 1. Pipeline Vocabulary

A Jenkins Pipeline is made up of several important components.


### Pipeline

The `pipeline` block is the complete definition of a Declarative Pipeline.

Example:

pipeline {
agent any

    stages {
        ...
    }
}


### Agent

An `agent` defines where the pipeline work runs.

Examples:

- Any available node
- A specific label
- A Docker container

Example:

pipeline {
agent any
}


### Stage

A `stage` represents a named phase of the pipeline.

Examples:

- Checkout
- Build
- Test
- Deploy

Example:

stage('Build') {
steps {
sh 'mvn clean package'
}
}


### Steps

Steps are the individual actions performed inside a stage.

Examples:

- `sh`
- `bat`
- `git`
- `echo`
- `archiveArtifacts`


### Workspace

The workspace is the directory on the Jenkins Agent where:

- Repository is checked out
- Build commands are executed
- Build files are generated


### Post

The `post` block contains actions that run after the stages.

Conditions include:

- always
- success
- failure
- unstable


## 2. Declarative Pipeline

Declarative Pipeline is the default choice for most Jenkins pipelines.

It uses a fixed and structured syntax.

Example:

pipeline {

    agent any

    tools {
        maven 'maven3'
    }

    options {
        timeout(time: 30, unit: 'MINUTES')
    }

    environment {
        APP = 'invoice-service'
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://...'
            }
        }

        stage('Build') {
            steps {
                sh 'mvn -B clean package'
            }
        }

        stage('Test') {
            steps {
                sh 'mvn test'
            }

            post {
                always {
                    junit 'target/surefire-reports/*.xml'
                }
            }
        }
    }

    post {
        success {
            echo "${APP} built successfully"
        }

        failure {
            mail to: 'team@company.com',
                 subject: 'Build failed'
        }
    }
}


## 3. Advantages of Declarative Pipeline

### Fixed Structure

Declarative Pipeline follows a defined structure.

This makes pipelines:

- Predictable
- Easier to understand
- Easier to validate


### Readable

The syntax is configuration-like and can be understood without deep Groovy knowledge.


### Early Validation

Jenkins can detect syntax problems before the pipeline starts executing.


### Script Escape Hatch

The `script { }` block can be used when more complex Groovy logic is required.


## 4. Important Declarative Directives

### agent

Defines where the pipeline runs.

Examples:

agent any

agent none

agent {
label 'linux'
}

agent {
docker '...'
}


### tools

Defines tools such as:

- JDK
- Maven
- Gradle


Example:

tools {
maven 'maven3'
jdk 'jdk17'
}


### environment

Defines environment variables.

Example:

environment {
APP = 'invoice-service'
}


### options

Defines pipeline options.

Examples:

- Timeout
- Retry
- Build discard policy


### parameters

Allows users to provide values when triggering a build.

Example:

parameters {
choice(
name: 'ENV',
choices: ['dev', 'prod']
)
}


### triggers

Defines how the pipeline starts.

Examples:

- cron
- pollSCM
- upstream


### when

Runs a stage only when a condition is satisfied.

Example:

stage('Deploy') {

    when {
        branch 'main'
    }

    steps {
        sh './deploy.sh'
    }
}


### post

Runs actions after the pipeline or stage.

Examples:

- success
- failure
- always


## 5. Parallel Execution

Jenkins Declarative Pipeline can run stages in parallel.

Example:

parallel {

    stage('Unit') {
        steps {
            ...
        }
    }

    stage('Lint') {
        steps {
            ...
        }
    }
}


Instead of:

Unit Test
↓
Lint
↓
Complete

They can run:

Unit Test ────────┐
├── Complete
Lint ─────────────┘


Parallel execution can reduce pipeline execution time.


## 6. Snippet Generator

Jenkins provides a Pipeline Syntax / Snippet Generator.

It helps generate the correct syntax for Jenkins pipeline steps.

It can be accessed from a Pipeline job.

Pipeline Syntax
↓
Snippet Generator
↓
Generate Step Syntax
↓
Copy into Jenkinsfile


## 7. Scripted Pipeline

Scripted Pipeline uses Groovy programming syntax.

Example:

node('linux') {

    stage('Checkout') {
        git url: 'https://github.com/team/app.git'
    }

    stage('Build') {
        def mvn = tool 'maven3'
        sh "${mvn}/bin/mvn -B clean package"
    }

    stage('Test') {

        try {
            sh 'mvn test'
        }

        catch (e) {
            currentBuild.result = 'UNSTABLE'
        }

        finally {
            junit 'target/surefire-reports/*.xml'
        }
    }
}


## 8. Scripted Pipeline Characteristics

Scripted Pipeline is a Groovy program.

It supports programming constructs such as:

- Variables
- Loops
- Conditions
- Functions
- try/catch
- Other Groovy logic


Scripted Pipeline starts with:

node {
...
}


Unlike Declarative Pipeline, Scripted Pipeline does not enforce the same fixed structure.


## 9. Declarative vs Scripted Pipeline

### Declarative

Opening block:

pipeline { }


Structure:

Fixed and validated


Syntax checking:

Before the build starts


Readability:

High


Flow control:

- when
- parallel
- retry


Restart from a stage:

Supported


Best for:

Most Jenkins pipelines


### Scripted

Opening block:

node { }


Structure:

Free-form Groovy


Syntax checking:

At runtime when the line executes


Readability:

Depends on the author


Flow control:

Full Groovy

Examples:

- if
- for
- try/catch


Restart from a stage:

Not supported


Best for:

Genuinely dynamic builds


## 10. Simple Comparison

Declarative:

pipeline
↓
Structured syntax
↓
Stages
↓
Steps


Scripted:

node
↓
Groovy code
↓
Logic
↓
Stages
↓
Steps


Both use the same underlying Jenkins Pipeline engine.

Declarative Pipeline is a structured layer on top of Scripted Pipeline.


## 11. Recommended Approach

Start with Declarative Pipeline.

Use:

script {
...
}

when some Groovy logic is required.

Move to fully Scripted Pipeline only when the pipeline genuinely needs extensive programmatic logic.


## 12. Multibranch Pipeline

A Multibranch Pipeline allows Jenkins to automatically discover branches containing a Jenkinsfile.

Instead of pointing Jenkins at one branch:

Jenkins
↓
Repository
↓
Scan branches
↓
Find Jenkinsfiles
↓
Create jobs for branches


Example:

Repository

main
├── Jenkinsfile

develop
├── Jenkinsfile

feature/login
├── Jenkinsfile


Jenkins can automatically create jobs for these branches.


## 13. Benefits of Multibranch Pipeline

### Automatic Branch Discovery

Jenkins scans the repository and finds branches containing a Jenkinsfile.


### Pull Request Builds

With the appropriate branch source plugin, Pull Requests can be built automatically and their status reported back.


### Automatic Cleanup

When a branch is deleted, its corresponding Jenkins job can also disappear.


### Per-Branch Behaviour

Different behaviour can be configured for different branches.

Example:

when {
branch 'main'
}


Only the `main` branch can perform deployment while other branches are built and tested.


## 14. Webhooks

A webhook allows the source-code repository to notify Jenkins when a change occurs.

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
Pipeline starts


Webhooks are preferred over frequent polling when immediate build triggering is required.


## 15. Webhook vs Polling

### Webhook

GitHub notifies Jenkins immediately after an event.

GitHub
↓
Push
↓
Webhook
↓
Jenkins


### Poll SCM

Jenkins periodically checks the repository.

Jenkins
↓
Check Repository
↓
Changes?
↓
Start Build


Webhook can start builds within seconds of a push and avoids repeatedly checking the repository.


# Jenkins Controller and Agent


## 16. Controller

The Jenkins Controller is the central Jenkins component.

Responsibilities:

- Serves the Jenkins web UI
- Stores configuration
- Stores build history
- Schedules jobs
- Distributes work to agents


The Controller should run as few builds as possible.


## 17. Agent

A Jenkins Agent is a machine that executes the actual build workload.

Examples:

Agent - Linux
→ Maven and JDK builds

Agent - Windows
→ .NET and MSBuild

Agent - Docker
→ Ephemeral build environment


Architecture:

                 Jenkins Controller
                         |
          ┌──────────────┼──────────────┐
          ↓              ↓              ↓
      Linux Agent    Windows Agent   Docker Agent
          ↓              ↓              ↓
       Maven          .NET          Docker Build


## 18. Why Use Agents?

### Scale

Multiple builds can run at the same time instead of waiting in one queue.


### Isolation

A problematic build is isolated from the Controller and other builds.


### Platform Coverage

One Jenkins installation can run builds on:

- Linux
- Windows
- macOS


### Security

Build code runs on Agents rather than directly on the machine holding Jenkins Controller configuration and credentials.


## 19. Old and New Terminology

Older terminology:

Master
Slave


Current terminology:

Controller
Agent


Both terms may appear in older Jenkins documentation or job configuration.


## 20. How Agents Connect

There are different ways for Jenkins Agents to connect.


### SSH

The Jenkins Controller connects to the Agent through SSH and launches the Agent process.

Controller
↓
SSH
↓
Agent


Commonly used for Linux Agents.


### Inbound Agent

The Agent connects to the Controller.

Agent
↓
Controller


The session material describes inbound agents using port `50000` and a secret.

This can be useful when the Agent is behind a firewall.


### Docker / Cloud Agents

Agents can be created when a build is required and destroyed afterwards.

Example:

Pipeline starts
↓
Create Agent
↓
Run Build
↓
Build Complete
↓
Destroy Agent


This provides elastic capacity and clean workspaces.


## 21. Requirements for an Agent

### Java

The Agent requires a compatible Java runtime because the Jenkins Agent process itself is Java.


### Remote Root Directory

The Agent needs a working directory where Jenkins keeps its workspace.

Example:

/home/jenkins


### Network Connectivity

Depending on the connection method:

Controller → Agent

or

Agent → Controller


## 22. Configuring a Jenkins Agent

### Step 1 - Prepare the Machine

Install:

- Java
- Jenkins user
- Remote root directory
- Required build tools


### Step 2 - Add the Node

Go to:

Manage Jenkins
↓
Nodes
↓
New Node
↓
Permanent Agent


### Step 3 - Configure Essentials

Configure:

- Remote root directory
- Number of executors
- Usage policy
- Labels


### Step 4 - Choose Launch Method

Options include:

- Launch agents via SSH
- Launch agent by connecting it to the Controller


### Step 5 - Verify

The node should appear as connected in the Nodes list.

If it does not connect, check the node log.


## 23. Agent Labels

Labels are tags used to identify Agent capabilities.

Examples:

linux

windows

docker

maven

kubernetes


Example:

Agent 1
labels:
linux
maven


Agent 2
labels:
linux
docker


A pipeline can request an Agent using a label.


## 24. Targeting an Agent from a Pipeline

Example:

pipeline {

    agent {
        label 'linux && maven'
    }

    stages {
        ...
    }
}


Jenkins finds an available Agent matching the label expression.


Example:

Pipeline
↓
label: linux && maven
↓
Matching Agent
↓
Build


## 25. Executors

Executors determine how many builds a node can run at the same time.

Example:

Agent
Executors = 2

↓

Build 1
Build 2

Both can run at the same time.


The session suggests a rough starting point of one executor per CPU core, depending on the workload.


## 26. Agent Usage Policy

A node can be configured so that it only runs jobs matching its label expressions.

Example:

Docker Agent
↓
Only Docker-related jobs


This helps reserve machines for specific workloads.


## 27. Controller Executors

For a production-style setup, builds should run on Agents rather than the Controller.

The hands-on lab recommends setting:

Controller Executors = 0


Architecture:

Jenkins Controller
Executors = 0
↓
Manages Jenkins
↓
Jenkins Agents
↓
Execute Builds


## 28. Distributed Build Architecture

A distributed Jenkins setup can look like:

                    Jenkins Controller
                           |
          ┌────────────────┼────────────────┐
          ↓                ↓                ↓
      Linux Agent     Docker Agent     Windows Agent
          ↓                ↓                ↓
       Maven Build     Docker Build     .NET Build


The Controller coordinates the work while Agents execute the workloads.


# Hands-on Lab 2


## 29. Pipeline Tasks

Create a Jenkinsfile with:

- Build stage
- Test stage
- Tools
- Environment variable
- Post block


Example flow:

Checkout
↓
Build
↓
Test
↓
Post


## 30. Extend the Pipeline

Add:

- A stage that runs only on `main`
- JUnit result publishing
- A step generated using Snippet Generator


Example:

stage('Deploy') {

    when {
        branch 'main'
    }

    steps {
        ...
    }
}


## 31. Agent Tasks

Add a second Jenkins node.

The session suggests:

- A second node
- Container is acceptable
- Give it the label `linux`
- Confirm that it connects


## 32. Target the Agent

Change:

agent any


to a label expression.

Example:

agent {
label 'linux'
}


Run the pipeline and confirm which node executed the build.


The build console output should show the node used for execution.


## 33. Important Session 2 Mental Model

Jenkinsfile
↓
Pipeline
↓
Agent
↓
Stage
↓
Steps
↓
Workspace


Example:

Developer
↓
GitHub
↓
Jenkins Controller
↓
Jenkinsfile
↓
Select Agent
↓
Checkout
↓
Build
↓
Test
↓
Artifact


## 34. Key Points to Remember

- Declarative Pipeline is the default choice for most pipelines.
- Scripted Pipeline provides full Groovy programming capabilities.
- `pipeline {}` is used for Declarative Pipeline.
- `node {}` is used for Scripted Pipeline.
- `agent` determines where work runs.
- `stage` represents a major phase of the pipeline.
- `steps` contain the actual commands/actions.
- `workspace` is where the build runs on the Agent.
- `post` handles actions after stages.
- Multibranch Pipeline automatically creates jobs for branches containing Jenkinsfiles.
- Webhooks can trigger builds immediately after repository events.
- Controller manages Jenkins and distributes work.
- Agents execute the actual build workload.
- Labels allow pipelines to target specific Agents.
- Executors determine how many builds a node can run simultaneously.
- Controller executors can be set to zero so builds run on Agents.
- Agents can connect through SSH, inbound connection, or Docker/cloud mechanisms.


## 35. Session 2 Overall Flow

GitHub
↓
Webhook
↓
Jenkins Controller
↓
Read Jenkinsfile
↓
Select Agent
↓
Checkout
↓
Build
↓
Test
↓
Artifact


Controller = Manage and Coordinate

Agent = Execute Work

Jenkinsfile = Pipeline as Code

Declarative = Structured Pipeline

Scripted = Groovy-based Pipeline

Multibranch = Pipeline for Multiple Branches