# Jenkins - Session 3
## Deployment, Triggers and Administration

### Slide 1 - Session 3 Overview

Session 3 covers:

- Tomcat setup and automated deployment
- Build triggers
- Email notifications
- Jenkins security
- Jenkins backups
- Jenkins administration
- Troubleshooting
- Lab 3 - Deploy and Automate


### Slide 2 - Deploying to Tomcat

Jenkins can deploy a WAR application to a running Apache Tomcat server.

Basic flow:

Jenkins
↓
Build WAR
↓
Tomcat Manager
↓
Deploy WAR
↓
Running Application


### Slide 3 - Tomcat Setup

Install Tomcat:

sudo apt install tomcat10

Start Tomcat:

sudo systemctl enable --now tomcat10

Tomcat normally uses port 8080.

If Jenkins is already using 8080:

Jenkins → 8080
Tomcat → 8090


### Slide 4 - Tomcat Manager

Tomcat Manager provides an HTTP interface for deploying applications.

Jenkins sends the WAR to the Tomcat Manager endpoint.

Jenkins
↓
HTTP Request
↓
Tomcat Manager
↓
WAR Deployment


### Slide 5 - manager-script Role

For automated deployment, use:

manager-script

Do not use:

manager-gui

Create a dedicated deployment user.

Example:

<role rolename="manager-script"/>

<user username="deployer"
password="..."
roles="manager-script"/>


### Slide 6 - Jenkins Credentials

Do not put the Tomcat username and password directly inside the Jenkinsfile.

Instead:

Tomcat Credentials
↓
Jenkins Credentials Store
↓
Credential ID
↓
Jenkins Pipeline


Benefits:

- Secrets are not hardcoded
- Credentials can be rotated centrally
- Jenkins can mask secret values in logs


### Slide 7 - Deployment Methods

Jenkins can deploy to Tomcat using:

1. Deploy to container plugin
2. curl to the Tomcat Manager API
3. Maven Cargo plugin


### Slide 8 - Deploy to Container Plugin

The plugin can be used from:

- Freestyle Job
- Jenkins Pipeline

It requires:

- WAR file
- Tomcat Manager URL
- Jenkins credential


### Slide 9 - Deployment Using curl

Deployment can also be performed using curl.

Advantages:

- No deployment plugin required
- Scriptable
- Easy to debug
- Uses the Tomcat HTTP API


### Slide 10 - Deploy Only the Tested Artifact

Important CI/CD practice:

Build
↓
Test
↓
Archive WAR
↓
Deploy SAME WAR


Do not rebuild the application during deployment.

The artifact deployed should be the artifact that was tested.


### Slide 11 - Deployment Stage

Example:

stage('Deploy') {

    when {
        branch 'main'
    }

    steps {
        ...
    }
}


The `when` condition can prevent deployment from branches that should not deploy.


### Slide 12 - Verify Deployment

After deployment, perform a smoke test.

Example:

curl -f http://tomcat-host:8090/invoice/health


Flow:

Deploy
↓
Application Starts
↓
curl Health Endpoint
↓
Success → Build Passes
Failure → Build Fails


### Slide 13 - Complete CI/CD Pipeline

Checkout
↓
Build
↓
Test
↓
Archive
↓
Deploy
↓
Verify


Example:

Checkout
→ Clone repository

Build
→ mvn clean package

Test
→ Unit tests

Archive
→ Keep WAR artifact

Deploy
→ Push WAR to Tomcat

Verify
→ Smoke test application


### Slide 14 - Build Triggers

A trigger determines what starts a Jenkins build.

Common triggers:

- Webhook from SCM
- Poll SCM
- Build periodically
- Upstream/downstream job
- Remote trigger
- Manual trigger with parameters


### Slide 15 - Webhook Trigger

GitHub notifies Jenkins when a change occurs.

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
Build


Webhook is normally the fastest way to react to a repository push.


### Slide 16 - Poll SCM

Jenkins periodically checks the repository for changes.

Example:

pollSCM('H/15 * * * *')


Jenkins
↓
Check Repository
↓
Changes?
↓
Build


A build starts only when Jenkins detects changes.


### Slide 17 - Build Periodically

Build Periodically runs according to a schedule whether or not the repository changed.

Example:

cron('H 2 * * *')


Useful for:

- Nightly builds
- Scheduled reports
- Periodic maintenance


### Slide 18 - Upstream and Downstream Jobs

A downstream job can be triggered after another job completes successfully.

Example:

Job A
↓
Success
↓
Job B
↓
Deploy


This can be used to chain related projects.


### Slide 19 - Remote Trigger

A Jenkins job can be triggered through an authenticated HTTP request using a token.

External System
↓
Authenticated Request
↓
Jenkins
↓
Build


### Slide 20 - Manual Trigger with Parameters

A user can manually start a build and provide parameters.

Example:

Environment:

dev
staging
prod


Pipeline
↓
User selects environment
↓
Build


### Slide 21 - Cron Syntax

Example:

triggers {

    cron('H 2 * * 1-5')

    pollSCM('H/15 * * * *')
}


`H` means hash.

Jenkins uses `H` to spread jobs across the time interval instead of starting many jobs at exactly the same time.


### Slide 22 - Email Notifications

Jenkins can notify the team about build results.

Basic setup:

Manage Jenkins
↓
System
↓
E-mail Notification
↓
Configure SMTP
↓
Test Configuration


### Slide 23 - Email Extension Plugin

Email Extension provides richer email notifications.

It supports:

- Different result triggers
- Templates
- Recipient rules
- More detailed notifications


### Slide 24 - Pipeline Notifications

Example:

post {

    failure {
        mail to: 'team@company.com',
             subject: "FAILED: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
             body: "See ${env.BUILD_URL}"
    }

    fixed {
        mail to: 'team@company.com',
             subject: 'Back to normal'
    }

    always {
        junit 'target/surefire-reports/*.xml'
    }
}


### Slide 25 - Notification Best Practice

Notify on important state changes.

Useful:

Build fails
↓
Send notification

Build becomes fixed
↓
Send notification


Avoid sending a successful email after every build because too many notifications can be ignored.


### Slide 26 - Jenkins Security

Important security areas:

- Authentication
- Authorization
- Credentials
- Access control
- Plugin security
- Job permissions


### Slide 27 - Authentication and Authorization

Authentication:

"Who are you?"

Authorization:

"What are you allowed to do?"


Jenkins can use:

- Authentication realm
- Matrix-based authorization
- Role-based authorization


Permissions can be separated into:

- Read
- Build
- Configure


### Slide 28 - Jenkins Credentials

Never hardcode secrets.

Credential types include:

- Username + password
- SSH username + private key
- Secret text
- Secret file


Examples:

- Git credentials
- Tomcat credentials
- API tokens
- Cloud credentials
- Certificates


### Slide 29 - Credential Scope

Credential scope controls where credentials are available.

Examples:

System credentials
↓
Used by Jenkins itself

Global credentials
↓
Available to jobs

Folder credentials
↓
Limited to a specific team/folder


Jobs should reference the credential ID rather than the secret value.


### Slide 30 - Jenkins Backup

JENKINS_HOME contains the important Jenkins data.

It includes:

- Jobs
- Build history
- Plugins
- Credentials
- Configuration


Important principle:

Back up JENKINS_HOME regularly.

Also test restoring the backup.


### Slide 31 - Jenkins Housekeeping

Old builds and artifacts can consume disk space.

Configure build retention.

Example:

Old Builds
↓
Discard
↓
Free Disk Space


Archived artifacts can be a major consumer of disk space.


### Slide 32 - Jenkins Monitoring

Important things to monitor:

- Disk space
- Queue length
- Executor usage
- Controller memory
- Agent availability


Monitoring helps identify Jenkins capacity and health problems.


### Slide 33 - Jenkins Updates

Keep Jenkins and plugins updated.

Best practice:

Backup
↓
Test
↓
Update Jenkins / Plugins
↓
Verify


Apply security updates deliberately.

Plugin updates can cause compatibility problems, so avoid uncontrolled bulk updates.


### Slide 34 - Configuration as Code

Jenkins Configuration as Code (JCasC) allows Jenkins configuration to be described using YAML.

JCasC configuration
↓
YAML
↓
Git Repository
↓
Jenkins
↓
Rebuild Configuration


This makes Jenkins configuration reproducible.


### Slide 35 - Manage Jenkins

Important administration areas:

System
↓
Global Jenkins settings

Tools
↓
JDK / Maven / Gradle / Git

Plugins
↓
Install / Update / Remove plugins

Nodes
↓
Manage Agents

Credentials
↓
Manage Secrets

Security
↓
Authentication / Authorization

Users
↓
Manage Accounts

System Information
↓
Environment / Versions / Plugins


### Slide 36 - Troubleshooting

Common problems:

Agent offline
→ Check Java, credentials and network/port.

mvn command not found
→ Check Maven installation and tools configuration.

Build works locally but fails in Jenkins
→ Check environment, user and tools on the Agent.

Webhook does not trigger
→ Check Jenkins URL and repository webhook delivery.

Deployment returns 403
→ Check Tomcat manager-script role and credentials.

Controller disk full
→ Check old builds and archived artifacts.


### Slide 37 - Troubleshooting Order

Start with:

1. Console Output
2. Node Log
3. Manage Jenkins → System Information


Use the build console output first because it usually provides the immediate error.


### Slide 38 - Lab 3

Lab 3 focuses on deploying and automating the application.

Tasks:

Tomcat
↓
Start Tomcat
↓
Create deployer user
↓
Configure manager-script
↓
Verify Manager URL

Deploy
↓
Store credential in Jenkins
↓
Add Deploy stage
↓
Open application

Trigger
↓
Add webhook / poll trigger
↓
Push commit
↓
Watch build start

Notify
↓
Configure SMTP
↓
Add failure / fixed notifications
↓
Test notification


### Slide 39 - Final Lab Flow

Developer
↓
Commit
↓
GitHub
↓
Webhook
↓
Jenkins
↓
Checkout
↓
Build
↓
Test
↓
Archive WAR
↓
Deploy to Tomcat
↓
Verify Application
↓
Notification


### Slide 40 - Session 3 Key Takeaways

Deployment:

Jenkins → Tomcat → Running Application

Triggers:

Webhook / Poll SCM / Cron / Manual / Remote

Notifications:

Email / Extended Email / Chat

Security:

Credentials + Authentication + Authorization

Administration:

Backup + Housekeeping + Monitoring + Updates

Configuration:

Jenkins Configuration as Code

Troubleshooting:

Console Output → Node Log → System Information


### Slide 41 - Complete Jenkins Module Flow

Session 1
↓
CI Concepts
↓
Jenkins Setup
↓
Freestyle Jobs

Session 2
↓
Pipeline as Code
↓
Jenkinsfile
↓
Controller / Agent

Session 3
↓
Tomcat Deployment
↓
Triggers
↓
Notifications
↓
Security
↓
Administration

Final Result:

GitHub
↓
Jenkins
↓
Build
↓
Test
↓
Archive
↓
Deploy
↓
Verify
↓
Notify