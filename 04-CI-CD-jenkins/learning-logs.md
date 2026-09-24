## Learning Log - CI/CD - Jenkins

**Session Date:** 14-09-2026  
**Study Date:** 21-09-2026

### Topics Covered

- Learned the basics of CI/CD.
- Understood Continuous Integration (CI) and Continuous Delivery/Deployment (CD).
- Learned what Jenkins is and how it is used for CI/CD automation.
- Understood Jenkins Controller and Jenkins Agent.
- Learned about Jenkins Jobs and Pipelines.
- Learned the purpose of a Jenkinsfile and Pipeline as Code.
- Understood Jenkins triggers and GitHub Webhooks.
- Learned Jenkins integration with Maven, Docker, Kubernetes, and AWS.
- Learned about Jenkins Credentials and secure handling of secrets.
- Understood the basic CI/CD flow from GitHub → Jenkins → Build → Test → Package → Deploy.

### Progress

Learned the fundamentals of Jenkins and understood how Jenkins fits into a real-world CI/CD pipeline.

## Learning Log - Jenkins Controller & Agent

**Session Date:** 14-09-2026  
**Study Date:** 21-09-2026

### Topics Covered

- Learned Jenkins Controller and Agent architecture.
- Understood the difference between Controller and Agent.
- Learned that Agents execute build, test, and deployment tasks.
- Understood that an Agent can be a separate EC2 instance, VM, physical server, container, or Kubernetes pod.
- Learned how the Controller assigns workloads to Agents.
- Understood the use of multiple Agents for distributing CI/CD workloads.
- Learned about Agent labels and different Agent capabilities.
- Understood static and dynamic Jenkins Agents.
- Learned the AWS example of using separate EC2 instances as Jenkins Agents.

### Progress

Understood how Jenkins Controller manages and distributes CI/CD workloads to Jenkins Agents.

## Learning Log - Jenkins Pipeline as Code

**Session Date:** 15-09-2026  
**Study Date:** 22-09-2026

### Topics Covered

- Learned Pipeline as Code in Jenkins.
- Understood the purpose of the Jenkinsfile.
- Learned how Jenkinsfile is stored and managed in Git.
- Understood the difference between Freestyle Jobs and Pipeline as Code.
- Learned Build as Code and the role of `pom.xml` in Maven.
- Understood how Jenkinsfile defines the CI/CD workflow.
- Learned the benefits of version-controlled pipeline configuration.

### Progress

Understood how Jenkins pipelines can be defined as code using a Jenkinsfile and managed through Git.

## Learning Log - Jenkins Session 2 controller agent architecture 

**Session Date:** 17-09-2026  
**Study Date:** 23-09-2026

### Topics Covered

- Learned Jenkins Pipeline vocabulary: pipeline, agent, stage, steps, workspace, and post.
- Learned Declarative Pipeline and its structure.
- Learned important Declarative directives such as `agent`, `tools`, `environment`, `options`, `parameters`, `triggers`, `when`, and `post`.
- Learned Scripted Pipeline and Groovy-based syntax.
- Understood the difference between Declarative and Scripted Pipelines.
- Learned Multibranch Pipelines and SCM integration.
- Learned how webhooks trigger Jenkins builds.
- Learned Jenkins Controller and Agent architecture.
- Learned different Agent connection methods such as SSH, inbound agents, and Docker/cloud agents.
- Learned how to configure Agent nodes, labels, and executors.
- Learned how to target a specific Agent using labels in a Jenkinsfile.
- Practiced the Session 2 pipeline and agent concepts.

### Progress

Understood how Jenkins uses Pipeline as Code and distributed Controller-Agent architecture to execute CI/CD workloads across different machines.

## Learning Log - Jenkins Session 3

**Session Date:** 18-09-2026  
**Study Date:** 24-09-2026

### Topics Covered

- Learned Tomcat setup and automated deployment using Jenkins.
- Learned how Jenkins deploys a WAR file to Tomcat.
- Learned `manager-script` role and dedicated Tomcat deployer credentials.
- Learned different deployment methods: plugin, `curl`, and Maven Cargo.
- Learned build triggers: Webhook, Poll SCM, Cron, Upstream/Downstream, Remote Trigger, and Manual Trigger.
- Learned Jenkins email notifications and SMTP configuration.
- Learned Jenkins security, authentication, authorization, and credentials.
- Learned Jenkins backup using `JENKINS_HOME`.
- Learned Jenkins housekeeping, monitoring, and updates.
- Learned Jenkins Configuration as Code (JCasC).
- Learned common Jenkins troubleshooting problems and solutions.
- Practiced the complete CI/CD flow from Git commit to Tomcat deployment and verification.

### Progress

Learned how to extend a Jenkins pipeline from build and test to automated deployment, triggering, notifications, and Jenkins administration.