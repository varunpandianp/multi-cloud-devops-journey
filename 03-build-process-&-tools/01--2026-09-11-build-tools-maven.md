Maven is a build automation and dependency management tool mainly used for Java applications.
## 1. Why do we need Maven?
   Without Maven, developers would manually:
- Download Java libraries/JAR files
- Compile Java code
- Run tests
- Package the application
- Manage library versions
- Create the final artifact
  Maven automates these activities.

## 2. Maven's main responsibilities
   Source Code
   ↓
   Maven
   ├── Download Dependencies
   ├── Compile Code
   ├── Run Tests
   ├── Package Application
   └── Create Artifact
   ↓
   JAR / WAR

## 3. Important file — pom.xml
   Maven uses pom.xml (Project Object Model) as its main configuration file.
   Example:
   <project>
   <groupId>com.example</groupId>
   <artifactId>my-app</artifactId>
   <version>1.0</version>

    <dependencies>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-core</artifactId>
            <version>...</version>
        </dependency>
    </dependencies>
</project>

The pom.xml defines:
- Project information
- Dependencies
- Plugins
- Build configuration
- Version
- Packaging type

## 4. Dependency management
   Suppose your Java application needs Spring.
   Instead of manually downloading:
   spring-core.jar
   spring-context.jar
   spring-beans.jar
   ...
   you declare them in pom.xml.
   Maven downloads them automatically from repositories.
   pom.xml
   ↓
   Maven
   ↓
   Maven Repository
   ↓
   Downloads dependencies
   ↓
   Local Repository (~/.m2)

## 5. Maven Build Lifecycle
   The important Maven lifecycle phases are:
   validate
   ↓
   compile
   ↓
   test
   ↓
   package
   ↓
   verify
   ↓
   install
   ↓
   deploy

## Most important for DevOps:
Command	Purpose
mvn compile	Compile source code
mvn test	Run tests
mvn package	Create JAR/WAR
mvn verify	Verify project
mvn install	Put artifact in local Maven repository
mvn deploy	Upload artifact to remote repository

## 6. What is the artifact?
   After:
   mvn package
   Maven can produce something like:
   target/
   └── my-app-1.0.jar
   That JAR is the build artifact.
   It can then be:
   Maven Build
   ↓
   JAR
   ↓
   Artifact Repository
   ↓
   Deployment
   ↓
   Server / Docker / Kubernetes

## 7. Maven in a CI/CD pipeline
   A common Java CI/CD pipeline:
   Developer
   ↓
   GitHub
   ↓
   Jenkins / GitHub Actions
   ↓
   Maven
   ↓
   Compile
   ↓
   Test
   ↓
   Package
   ↓
   JAR
   ↓
   SonarQube / Security Scan
   ↓
   Nexus / Artifactory
   ↓
   Docker Image
   ↓
   Kubernetes / AWS

8. Very important distinction
   Maven does not deploy your entire application infrastructure.
   Maven's primary job is:
   Build Java applications and manage their dependencies.

For example:
- Git → source code management
- Maven → Java build + dependencies
- Docker → package application into container
- Terraform → provision infrastructure
- Ansible → configuration/automation
- Jenkins/GitHub Actions → CI/CD orchestration
- Kubernetes → container orchestration
  Interview answer
  Maven is a Java build automation and dependency management tool. It uses pom.xml to define project configuration and dependencies. Maven follows a standard build lifecycle such as compile, test, package, install, and deploy, and can generate artifacts such as JAR or WAR files that can be used in CI/CD pipelines.

## For your first Maven project, keep it simple and learn the standard Maven structure.
## 1. Create the project
   If Maven is installed:
   mvn archetype:generate \
   -DgroupId=com.example \
   -DartifactId=my-first-maven-app \
   -DarchetypeArtifactId=maven-archetype-quickstart \
   -DinteractiveMode=false
   This generates:
   my-first-maven-app/
   ├── pom.xml
   └── src/
   ├── main/
   │   └── java/
   │       └── com/
   │           └── example/
   │               └── App.java
   └── test/
   └── java/
   └── com/
   └── example/
   └── AppTest.java

2. Understand the structure
   my-first-maven-app/
   │
   ├── pom.xml                 ← Maven configuration
   │
   └── src/
   │
   ├── main/
   │   └── java/           ← Application source code
   │
   └── test/
   └── java/           ← Test code
   After you run:
   mvn package
   Maven creates:
   my-first-maven-app/
   ├── pom.xml
   ├── src/
   │   ├── main/
   │   └── test/
   └── target/
   ├── classes/
   ├── test-classes/
   ├── surefire-reports/
   └── my-first-maven-app-1.0-SNAPSHOT.jar
3. The important DevOps flow
   Remember this:
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
   target/my-first-maven-app-1.0-SNAPSHOT.jar
   Then try:
   cd my-first-maven-app

mvn clean
mvn compile
mvn test
mvn package

## Maven / Ant / Gradle — DevOps Notes
These are the notes I recommend you keep in your Maven + Build Tools section.
## 1. What is a Build Tool?
   A build tool automates the process of converting source code into a usable application artifact.
   For a Java application:
   Source Code
   ↓
   Compile
   ↓
   Test
   ↓
   Package
   ↓
   JAR / WAR
   ↓
   Deploy

## Instead of doing these manually, a build tool automates them.
## 2. Ant vs Maven vs Gradle
| Feature | Ant | Maven | Gradle |
|---|---|---|---|
| Created for | Java | Java | Java/JVM |
| Configuration | XML | XML (`pom.xml`) | Groovy/Kotlin DSL |
| Style | Mostly imperative | Mostly declarative | Declarative + programmable |
| Dependency management | Basic/manual | Built-in | Built-in |
| Convention | Low | High | High |
| Build performance | Basic | Good | Very good, caching/incremental builds |
| Learning | Relatively simple initially | Structured | More flexible, more concepts |
| Common use | Older Java projects | Very common Java ecosystem | Modern JVM/Android and large builds |

Simple way to remember
Ant:
"Tell me HOW to build."

## Maven:
"Tell me WHAT the project needs; Maven knows the standard way to build it."

## Gradle:
"Tell me WHAT I need, with a programmable build system when necessary."

## 3. Imperative vs Declarative
   This is an important DevOps concept.
   Imperative
   You tell the tool how to perform every step.
   Example:
1. Create directory
2. Compile these files
3. Copy these files
4. Run tests
5. Create JAR
6. Move JAR to another directory
   You are specifying the procedure.
   Ant
   Ant is traditionally considered imperative/task-oriented.
   Example:
   <target name="compile">
   <mkdir dir="build/classes"/>
   <javac srcdir="src" destdir="build/classes"/>
   </target>
   You are telling Ant what actions to execute.
## 4. Declarative
   You describe what you want, and the tool determines the steps using conventions/lifecycle.
   For Maven:
   <dependencies>
   <dependency>
   ...
   </dependency>
   </dependencies>
   You are basically saying:
   "My application needs this dependency."

## Then Maven handles downloading and managing it.
You don't normally tell Maven:
Go to this URL
Download this JAR
Put it here
Add it to classpath
Maven handles that.
Maven = convention + lifecycle
pom.xml
↓
Maven understands project
↓
Maven lifecycle
↓
Compile → Test → Package

## 5. Gradle
   Gradle is more flexible.
   It supports declarative configuration, but because the build files are written using Groovy or Kotlin DSL, you can also use programming logic.
   Example:
   plugins {
   java
   }

dependencies {
implementation("org.springframework:spring-core:...")
}
So remember:
Ant    → mostly imperative
Maven  → mostly declarative
Gradle → declarative + programmable

## 6. Maven settings.xml
   This is an important distinction:
   pom.xml
   Project-specific configuration.
   my-project/
   └── pom.xml

   ## settings.xml
   Maven/user/environment-specific configuration.
   Usually located at:
   ~/.m2/settings.xml
   There can also be a Maven installation-level settings file.
   What goes in settings.xml?
   Common things include:
- Repository configuration
- Mirror configuration
- Authentication/server credentials
- Profiles
- Proxy configuration
- Environment-specific Maven settings
  
- ## Example:
  <settings>
  <mirrors>
  <mirror>
  <id>company-mirror</id>
  <url>https://nexus.example.com/repository/maven-public/</url>
  <mirrorOf>*</mirrorOf>
  </mirror>
  </mirrors>
  </settings>
  Very important
  Don't confuse:
  pom.xml
  ↓
  Project configuration

settings.xml
↓
User/Maven environment configuration

Very important
Don't confuse:
pom.xml
↓
Project configuration

settings.xml
↓
User/Maven environment configuration

## 7. Maven Repository
   A Maven repository stores artifacts and dependencies.
   For example:
   spring-core
   junit
   log4j
   are available as Maven artifacts.
   There are three important repository concepts:
   Local Repository
   ↑
   |
   Remote Repository
   ↑
   |
   Maven Central
## 8. Local Maven Repository
   Usually:
   ~/.m2/repository/
   When Maven downloads a dependency, it normally caches it locally.
   Example:
   ~/.m2/repository/
   └── org/
   └── junit/
   └── junit/
   └── ...
   So Maven doesn't necessarily download the same dependency from the network every time.
## 9. Maven Central
   Maven Central is a major public repository for Java/JVM artifacts.
   Think of it like a huge public library of Maven packages.
   Your Project
   ↓
   Maven
   ↓
   Local ~/.m2/repository
   ↓
   Remote repository
   ↓
   Maven Central
   For example, your pom.xml might declare:
   <dependency>
   <groupId>org.junit.jupiter</groupId>
   <artifactId>junit-jupiter</artifactId>
   <version>...</version>
   </dependency>
   Maven resolves the dependency through configured repositories and stores the downloaded artifact locally.
   Production environment
   Companies often don't want every build server directly depending on public repositories.
   They may use:
   Developer / CI
   ↓
   Company Nexus / Artifactory
   ↓
   Maven Central
   This gives the organization more control over dependencies, caching, availability and security.


## 10. Three Maven Build Lifecycles
    This is very important for interviews.
    Maven has three built-in lifecycles:
1. Clean
2. Default
3. Site
   ## Lifecycle 1 — Clean
   Used to clean previous build output.
   Main command:
   mvn clean
   It removes the:
   target/
   directory.
   Example:
   Before:

project/
├── src/
├── pom.xml
└── target/

        ↓ mvn clean

project/
├── src/
└── pom.xml

## 11. Default Lifecycle
    This is the main build lifecycle.
    Important phases:
    validate
    ↓
    compile
    ↓
    test
    ↓
    package
    ↓
    verify
    ↓
    install
    ↓
    deploy
    What they do
    validate
    Checks whether the project is correct.
    compile
    Compiles source code.
    mvn compile
    test
    Runs tests.
    mvn test
    package
    Creates the artifact.
    mvn package
    For example:
    target/
    └── my-app-1.0.jar
    verify
    Runs checks to verify the package is valid.
    install
    Installs the artifact into your local Maven repository:
    ~/.m2/repository/
    deploy
    Publishes the artifact to a remote repository.
## 12. Site Lifecycle
    The third lifecycle is:
    Site
    It generates project documentation/site information.
    For example:
    mvn site
    It can generate reports such as:
- Project documentation
- Test reports
- Dependency information
- Project information
  This is less important for your day-to-day DevOps work than the default lifecycle, but know it for interviews.

BUILD TOOLS
│
├── Ant
│   └── Mostly imperative
│       └── Tell HOW
│
├── Maven
│   ├── Mostly declarative
│   ├── pom.xml
│   ├── Dependency management
│   ├── Standard lifecycle
│   └── JAR/WAR artifacts
│
└── Gradle
├── Declarative + programmable
├── Groovy/Kotlin DSL
├── Dependency management
└── Flexible/incremental builds

## MAVEN CONFIGURATION

pom.xml
→ Project configuration

settings.xml
→ User/environment/Maven configuration

~/.m2/repository
→ Local dependency/artifact cache

Maven Central
→ Public remote repository

## MAVEN LIFECYCLES

Clean
→ Clean build output

Default
→ Build / test / package / install / deploy

Site
→ Generate project documentation/reports

## ⭐ Interview memory line
Maven is a convention-based build and dependency management tool. pom.xml defines the project, settings.xml defines user/environment-level Maven settings, repositories provide dependencies and artifacts, and Maven has three lifecycles: Clean, Default, and Site.