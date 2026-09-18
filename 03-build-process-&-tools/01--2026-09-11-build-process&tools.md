## In DevOps/CI/CD, a build tool and a build process are related but different.
1. Build Tool
   A build tool automates the steps required to turn source code → a deployable artifact.
   Examples:
   Language	Common Build Tools
   Java	Maven, Gradle
   JavaScript/Node.js	npm, yarn
   Python	pip, Poetry
   C/C++	Make, CMake
   Go	go build

For example, in Java:
Developer Code
↓
Maven
↓
Compile → Test → Package
↓
JAR/WAR Artifact
Maven can compile the code, run tests, resolve dependencies, and package the application.

2. Build Process
   The build process is the sequence of steps used to create the artifact.
   Typical DevOps build process:
1. Developer pushes code
   ↓
2. CI tool pulls source code
   ↓
3. Install/resolve dependencies
   ↓
4. Compile source code
   ↓
5. Run unit tests
   ↓
6. Code quality/security checks
   ↓
7. Package application
   ↓
8. Create artifact
   ↓
9. Store artifact in Artifact Repository

Example:
GitHub
↓
GitHub Actions / Jenkins
↓
Maven
↓
Compile
↓
Test
↓
Package
↓
app.jar
↓
Nexus / Artifactory / S3


Git → source code
Maven → build/package
Artifact Repository → stores artifact
Jenkins/GitHub Actions → automates the pipeline
Docker → packages application into an image
Kubernetes → runs/deploys the application