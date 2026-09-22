## Jenkins - Pipeline as Code and Build as Code

### 1. Pipeline as Code

Pipeline as Code means defining the Jenkins CI/CD pipeline using code instead of manually configuring the pipeline through the Jenkins UI.

The pipeline is usually written in a file called:

Jenkinsfile

The Jenkinsfile is stored in Git along with the application code.

Example:

Developer
↓
Git Push
↓
GitHub
↓
Jenkins
↓
Jenkinsfile
↓
Execute Pipeline


### 2. Jenkinsfile

A Jenkinsfile defines the stages and steps of a Jenkins pipeline.

Example:

pipeline {
agent any

    stages {

        stage('Build') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('Test') {
            steps {
                sh 'mvn test'
            }
        }
    }
}


### 3. Benefits of Pipeline as Code

- Pipeline configuration is stored in Git.
- Changes can be tracked.
- Pipeline can be reviewed through Pull Requests.
- Previous versions can be restored.
- Pipeline configuration is reproducible.
- Reduces manual Jenkins UI configuration.
- Supports complex CI/CD workflows.


### 4. Build as Code

Build as Code means defining the application build process using code or configuration files instead of manually configuring the build process.

Example with Maven:

pom.xml
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


The Maven build configuration is mainly defined in:

pom.xml


### 5. Jenkinsfile vs pom.xml

Jenkinsfile:

Defines the CI/CD workflow.

Example:

Checkout
↓
Build
↓
Test
↓
Docker Build
↓
Deploy


pom.xml:

Defines the Maven application build configuration.

Example:

Dependencies
↓
Compile
↓
Test
↓
Package


### 6. Pipeline as Code vs Freestyle Job

Freestyle Job:

Jenkins UI
↓
Manual Configuration
↓
Build Steps
↓
Post-build Actions


Pipeline as Code:

Jenkinsfile
↓
Git
↓
Jenkins
↓
Pipeline Execution


### 7. Real-World Example

Developer
↓
GitHub
↓
Jenkins
↓
Jenkinsfile
↓
Maven Build
↓
Testing
↓
Docker Build
↓
Container Registry
↓
Kubernetes / AWS


### 8. Simple Mental Model

Jenkinsfile = CI/CD workflow as code

pom.xml = Maven build configuration as code

Git = Version control for the code and configuration


### 9. Interview Definition

Pipeline as Code means defining a Jenkins CI/CD pipeline in a Jenkinsfile and storing it in version control such as Git.

Build as Code means defining the application build process using configuration or code instead of relying on manual build configuration.