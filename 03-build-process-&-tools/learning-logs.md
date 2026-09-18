# Learning Log — 18-09-2026 session date :11-09-2026

## Topic: Build Tools & Maven

Today I learned:

- Ant vs Maven vs Gradle
- Imperative vs Declarative build systems
- Maven and its purpose
- pom.xml
- Maven project structure
- Maven dependency management
- Maven Central
- Local Maven repository (~/.m2/repository/)
- Maven settings.xml
- Maven Clean, Default and Site lifecycles
- Maven build phases
- Maven commands: clean, compile, test, package, verify, install, deploy
- Maven artifacts (JAR/WAR)
- Maven's role in CI/CD pipelines

### Key Learning

Maven automates Java application builds, dependency management, testing and packaging.

pom.xml → Project configuration
settings.xml → User/environment configuration
~/.m2/repository → Local repository
Maven Central → Public repository

Three Maven lifecycles:
Clean → Clean build output
Default → Build and deployment lifecycle
Site → Documentation and reports

### DevOps Connection

Git → Source Code
Maven → Build & Package
Nexus/Artifactory → Artifact Storage
Docker → Containerization
Kubernetes/AWS → Deployment