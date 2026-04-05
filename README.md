# 🔍 SonarQube — Code Quality Analysis & DevSecOps Integration

<div align="center">

![SonarQube](https://img.shields.io/badge/SonarQube-4E9BCD?style=for-the-badge&logo=sonarqube&logoColor=white)
![Jenkins](https://img.shields.io/badge/Jenkins-D24939?style=for-the-badge&logo=jenkins&logoColor=white)
![Java](https://img.shields.io/badge/Java-ED8B00?style=for-the-badge&logo=openjdk&logoColor=white)
![Maven](https://img.shields.io/badge/Maven-C71A36?style=for-the-badge&logo=apachemaven&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-2496ED?style=for-the-badge&logo=docker&logoColor=white)
![AWS](https://img.shields.io/badge/AWS-FF9900?style=for-the-badge&logo=amazonaws&logoColor=white)

<br/>

**A production-grade CI/CD pipeline integrating SonarQube for continuous code quality analysis,**
**bug detection, vulnerability scanning, and code smell reporting on a Java Maven application.**

<br/>

*Built by [Akshay Sawant](mailto:akshaysawant9009@gmail.com) — AWS DevOps Engineer | Hinjawadi, Pune*

---

### 🎯 What This Project Demonstrates
`Static Code Analysis` &nbsp;·&nbsp; `Automated Quality Gates` &nbsp;·&nbsp; `CI/CD Integration` &nbsp;·&nbsp; `DevSecOps Pipeline` &nbsp;·&nbsp; `Docker Deployment`

</div>

---

## 📌 Table of Contents

- [Project Overview](#-project-overview)
- [Architecture Diagram](#-architecture-diagram)
- [What is SonarQube?](#-what-is-sonarqube)
- [What SonarQube Analyzes](#-what-sonarqube-analyzes)
- [Project Structure](#-project-structure)
- [Tech Stack](#-tech-stack)
- [Prerequisites](#-prerequisites)
- [Infrastructure Setup](#-infrastructure-setup)
- [SonarQube Setup](#-sonarqube-setup)
- [Jenkins Pipeline Setup](#-jenkins-pipeline-setup)
- [Jenkinsfile Explained](#-jenkinsfile-explained)
- [Quality Gate — Pass or Fail](#-quality-gate--pass-or-fail)
- [SonarQube Dashboard](#-sonarqube-dashboard)
- [Docker Setup](#-docker-setup)
- [Pipeline Execution — Step by Step](#-pipeline-execution--step-by-step)
- [Key Learnings](#-key-learnings)
- [Author](#-author)

---

## 🎯 Project Overview

> **"Shift-left security and quality — catch bugs before they reach production."**

This project integrates **SonarQube** into a **Jenkins CI/CD pipeline** for a Java Maven web application. Every time code is pushed to GitHub, Jenkins automatically:

1. **Pulls** the latest code from GitHub
2. **Builds** and **tests** it using Maven
3. **Scans** it with SonarQube for bugs, vulnerabilities, and code smells
4. **Checks** the Quality Gate — pipeline fails if code doesn't meet standards
5. **Packages** the app into a Docker image
6. **Deploys** to the target environment

```
Developer pushes code
        ↓
   GitHub Webhook
        ↓
  Jenkins Pipeline
        ↓
  Maven Build + Test
        ↓
  SonarQube Analysis  ←── Static Code Scan
        ↓
  Quality Gate Check  ←── Pass ✅ or Fail ❌
        ↓
  Docker Build + Push
        ↓
     Deployment
```

---

## 🏗️ Architecture Diagram

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                        CI/CD PIPELINE FLOW                                  │
└─────────────────────────────────────────────────────────────────────────────┘

  ┌──────────┐    webhook    ┌──────────────┐   maven build   ┌────────────┐
  │  GitHub  │ ────────────► │   Jenkins    │ ───────────────► │   Maven    │
  │  (SCM)   │               │  (Pipeline)  │                  │  Build+Test│
  └──────────┘               └──────────────┘                  └─────┬──────┘
                                                                      │
                                                               sonar-maven-plugin
                                                                      │
                                                                      ▼
  ┌──────────┐   quality     ┌──────────────┐    analysis     ┌────────────────┐
  │  Docker  │ ◄──────────── │   Quality    │ ◄─────────────  │   SonarQube    │
  │  Deploy  │    gate pass  │    Gate      │                  │    Server      │
  └──────────┘               └──────────────┘                  └────────────────┘

 AWS EC2 Instances:
 ┌────────────────────┐   ┌────────────────────┐   ┌────────────────────┐
 │   Jenkins Server   │   │  SonarQube Server  │   │   App Server       │
 │   (EC2 t2.medium)  │   │  (EC2 t2.medium)   │   │   (EC2 t2.micro)   │
 │   Port: 8080       │   │  Port: 9000        │   │   Port: 8080       │
 └────────────────────┘   └────────────────────┘   └────────────────────┘
```

---

## 🔍 What is SonarQube?

**SonarQube** is an open-source **static code analysis** tool that continuously inspects your source code to detect:

```
┌──────────────────────────────────────────────────────────────────┐
│                    SonarQube Analyzes:                           │
│                                                                  │
│  🐛 BUGS          → Logic errors that will cause failures        │
│  🔒 VULNERABILITIES → Security holes (SQL injection, XSS, etc.) │
│  👃 CODE SMELLS   → Maintainability issues (tech debt)           │
│  📋 DUPLICATIONS  → Copy-pasted code blocks                      │
│  📊 COVERAGE      → % of code covered by unit tests             │
│  📏 COMPLEXITY    → Cyclomatic complexity per function           │
└──────────────────────────────────────────────────────────────────┘
```

### How SonarQube Rates Issues:

| Severity | Icon | Meaning | Example |
|----------|------|---------|---------|
| BLOCKER | 🔴 | Must fix — will crash | Null pointer dereference |
| CRITICAL | 🟠 | Security or major bug | SQL injection risk |
| MAJOR | 🟡 | Functional problem | Wrong logic path |
| MINOR | 🔵 | Style or minor issue | Missing bracket spacing |
| INFO | ⚪ | Informational only | Comment formatting |

---

## 🔎 What SonarQube Analyzes

### 1. 🐛 Bugs
Code that is clearly wrong or will cause runtime failures.
```java
// ❌ Bug: NullPointerException risk
String name = null;
if (name.equals("admin")) { ... }   // SonarQube flags this

// ✅ Fixed
if ("admin".equals(name)) { ... }
```

### 2. 🔒 Vulnerabilities
Security weaknesses exploitable by attackers.
```java
// ❌ Vulnerability: Hardcoded password
String password = "admin123";       // SonarQube flags this immediately

// ✅ Fixed: Read from environment variable
String password = System.getenv("DB_PASSWORD");
```

### 3. 👃 Code Smells
Code that works but is hard to maintain, understand, or extend.
```java
// ❌ Code Smell: Method too long (200+ lines), too complex
public void doEverything() {
    // ... 200 lines of tangled logic
}

// ✅ Fixed: Break into smaller, single-responsibility methods
public void processOrder() { ... }
public void sendNotification() { ... }
public void updateInventory() { ... }
```

### 4. 📋 Code Duplication
Copy-pasted blocks that should be extracted into shared methods.

### 5. 📊 Test Coverage
```
SonarQube reports:
  Lines covered:     847 / 1200  →  70.6% ✅  (threshold: > 70%)
  Branches covered:  124 / 180   →  68.9% ⚠️  (threshold: > 70%)
```

---

## 📁 Project Structure

```
SonarQube-Project/
│
├── 📄 README.md                          ← Project documentation (this file)
├── 📄 Jenkinsfile                        ← Jenkins declarative pipeline
├── 📄 Dockerfile                         ← Container image definition
├── 📄 pom.xml                            ← Maven project config + SonarQube plugin
├── 📄 SonarQube - Code Quality Analysis.txt  ← Analysis output notes
│
└── 📁 src/
    └── 📁 main/
        ├── 📁 java/
        │   └── com/example/app/
        │       ├── App.java              ← Main application entry point
        │       └── ...
        └── 📁 resources/
            └── application.properties   ← App configuration
```

---

## 🛠️ Tech Stack

| Tool | Version | Purpose |
|------|---------|---------|
| **Java** | 11 / 17 | Application language |
| **Maven** | 3.8+ | Build automation & dependency management |
| **SonarQube** | 9.x / 10.x | Static code analysis & quality gate |
| **Jenkins** | 2.400+ | CI/CD pipeline orchestration |
| **Docker** | 24+ | Containerization |
| **AWS EC2** | t2.medium | Hosting Jenkins & SonarQube servers |
| **GitHub** | — | Source control & webhook trigger |

---

## ✅ Prerequisites

Before running this project, ensure you have:

```bash
# On Jenkins Server (EC2 t2.medium — min 4GB RAM)
✅ Java 11+ installed
✅ Jenkins installed and running on port 8080
✅ Maven installed
✅ Docker installed
✅ Git installed

# On SonarQube Server (EC2 t2.medium — min 4GB RAM)
✅ Java 11+ installed
✅ SonarQube installed and running on port 9000
✅ PostgreSQL (recommended for production SonarQube)

# Jenkins Plugins Required
✅ SonarQube Scanner Plugin
✅ Maven Integration Plugin
✅ Pipeline Plugin
✅ Git Plugin
✅ Docker Pipeline Plugin
```

> ⚠️ **RAM Warning:** SonarQube requires **minimum 4GB RAM**. Use at least `t2.medium` on AWS. It will crash on `t2.micro`.

---

## 🖥️ Infrastructure Setup

### Step 1 — Launch AWS EC2 Instances

```bash
# Launch 2 EC2 instances (t2.medium, Ubuntu 22.04):
Instance 1: Jenkins Server    → Open ports: 8080, 22
Instance 2: SonarQube Server  → Open ports: 9000, 22

# Security Group Inbound Rules:
Port 22    → SSH access (your IP)
Port 8080  → Jenkins Web UI
Port 9000  → SonarQube Web UI
```

### Step 2 — Install Java on Both Servers

```bash
sudo apt update
sudo apt install -y openjdk-17-jdk
java -version
# Output: openjdk version "17.x.x"
```

### Step 3 — Install Jenkins

```bash
# Add Jenkins repo
curl -fsSL https://pkg.jenkins.io/debian/jenkins.io-2023.key | \
  sudo tee /usr/share/keyrings/jenkins-keyring.asc > /dev/null

echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
  https://pkg.jenkins.io/debian binary/ | \
  sudo tee /etc/apt/sources.list.d/jenkins.list > /dev/null

# Install Jenkins
sudo apt update
sudo apt install -y jenkins

# Start & enable
sudo systemctl start jenkins
sudo systemctl enable jenkins
sudo systemctl status jenkins

# Get initial admin password
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```
> 🌐 Access Jenkins: `http://<JENKINS-EC2-IP>:8080`

### Step 4 — Install Maven

```bash
sudo apt install -y maven
mvn -version
# Output: Apache Maven 3.8.x
```

### Step 5 — Install Docker on Jenkins Server

```bash
sudo apt install -y docker.io
sudo usermod -aG docker jenkins   # Give Jenkins docker permission
sudo systemctl restart jenkins
docker --version
```

---

## 🔧 SonarQube Setup

### Step 1 — Install SonarQube

```bash
# Download SonarQube
wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-10.3.0.82913.zip
sudo apt install -y unzip
unzip sonarqube-10.3.0.82913.zip
sudo mv sonarqube-10.3.0.82913 /opt/sonarqube

# Create sonar user (SonarQube cannot run as root)
sudo useradd -M -d /opt/sonarqube -r -s /bin/bash sonarqube
sudo chown -R sonarqube:sonarqube /opt/sonarqube

# Start SonarQube
sudo -u sonarqube /opt/sonarqube/bin/linux-x86-64/sonar.sh start
```
> 🌐 Access SonarQube: `http://<SONAR-EC2-IP>:9000`
> Default login: `admin` / `admin` → change on first login

### Step 2 — Generate SonarQube Token

```
SonarQube UI → My Account (top right) 
  → Security tab 
  → Generate Tokens 
  → Name: "jenkins-token" 
  → Type: Global Analysis Token 
  → Generate
  → COPY THE TOKEN (shown only once!)
```

### Step 3 — Create SonarQube Project

```
SonarQube UI → Projects → Create Project
  → Project key:    my-java-app
  → Display name:   My Java Application
  → Set Up
  → Select: "With Jenkins"  (or "Locally" for manual)
```

### Step 4 — Add Token to Jenkins Credentials

```
Jenkins → Dashboard → Manage Jenkins → Credentials
  → System → Global credentials → Add Credentials
  → Kind: Secret text
  → Secret: <paste your SonarQube token>
  → ID: sonarqube-token
  → Description: SonarQube Analysis Token
  → Save
```

### Step 5 — Configure SonarQube Server in Jenkins

```
Jenkins → Manage Jenkins → Configure System
  → SonarQube Servers section
  → Add SonarQube:
      Name:           SonarQube
      Server URL:     http://<SONAR-EC2-IP>:9000
      Token:          (select) sonarqube-token
  → Save
```

### Step 6 — Install SonarQube Scanner in Jenkins

```
Jenkins → Manage Jenkins → Global Tool Configuration
  → SonarQube Scanner
  → Add SonarQube Scanner
      Name: SonarQube-Scanner
      ✅ Install automatically
  → Save
```

---

## ⚙️ Jenkins Pipeline Setup

### Step 1 — Create Pipeline Job

```
Jenkins → New Item
  → Name: SonarQube-Code-Analysis
  → Type: Pipeline
  → OK

Pipeline section:
  → Definition: Pipeline script from SCM
  → SCM: Git
  → Repository URL: https://github.com/social9009/SonarQube-Project.git
  → Branch: */main
  → Script Path: Jenkinsfile
  → Save
```

### Step 2 — Configure GitHub Webhook

```
GitHub Repo → Settings → Webhooks → Add Webhook
  → Payload URL: http://<JENKINS-IP>:8080/github-webhook/
  → Content type: application/json
  → Events: ✅ Just the push event
  → Add webhook
```

---

## 📋 Jenkinsfile Explained

```groovy
pipeline {
    agent any

    tools {
        maven 'Maven'          // Uses Maven configured in Global Tool Config
        jdk   'Java17'         // Uses JDK configured in Global Tool Config
    }

    environment {
        SONAR_PROJECT_KEY = 'my-java-app'
        DOCKER_IMAGE      = 'akshaysawant/java-app'
    }

    stages {

        // ─── STAGE 1: Pull latest code from GitHub ───────────────────────
        stage('Git Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/social9009/SonarQube-Project.git'
                echo '✅ Code checkout complete'
            }
        }

        // ─── STAGE 2: Compile the Java application ────────────────────────
        stage('Compile') {
            steps {
                sh 'mvn compile'
                echo '✅ Compilation successful'
            }
        }

        // ─── STAGE 3: Run unit tests ──────────────────────────────────────
        stage('Test') {
            steps {
                sh 'mvn test'
                echo '✅ All tests passed'
            }
        }

        // ─── STAGE 4: SonarQube Static Code Analysis ─────────────────────
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {    // Must match name in Jenkins config
                    sh '''
                        mvn sonar:sonar \
                          -Dsonar.projectKey=${SONAR_PROJECT_KEY} \
                          -Dsonar.projectName="My Java App" \
                          -Dsonar.java.binaries=target/classes \
                          -Dsonar.coverage.jacoco.xmlReportPaths=target/site/jacoco/jacoco.xml
                    '''
                }
                echo '✅ SonarQube analysis sent'
            }
        }

        // ─── STAGE 5: Wait for Quality Gate result ────────────────────────
        stage('Quality Gate') {
            steps {
                timeout(time: 2, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                    // ↑ Pipeline STOPS HERE if Quality Gate FAILS
                }
                echo '✅ Quality Gate passed'
            }
        }

        // ─── STAGE 6: Package into JAR ────────────────────────────────────
        stage('Build Package') {
            steps {
                sh 'mvn package -DskipTests'
                echo '✅ JAR file built: target/*.jar'
            }
        }

        // ─── STAGE 7: Build Docker Image ──────────────────────────────────
        stage('Docker Build') {
            steps {
                sh 'docker build -t ${DOCKER_IMAGE}:${BUILD_NUMBER} .'
                sh 'docker tag  ${DOCKER_IMAGE}:${BUILD_NUMBER} ${DOCKER_IMAGE}:latest'
                echo '✅ Docker image built'
            }
        }

        // ─── STAGE 8: Deploy ──────────────────────────────────────────────
        stage('Deploy') {
            steps {
                sh 'docker run -d -p 8080:8080 --name java-app ${DOCKER_IMAGE}:latest'
                echo '✅ Application deployed on port 8080'
            }
        }
    }

    // ─── POST ACTIONS ─────────────────────────────────────────────────────
    post {
        success {
            echo '🎉 Pipeline SUCCESS — Code quality verified!'
        }
        failure {
            echo '❌ Pipeline FAILED — Check SonarQube Quality Gate!'
        }
    }
}
```

---

## 🚦 Quality Gate — Pass or Fail

A **Quality Gate** is SonarQube's set of pass/fail conditions. The default **"Sonar Way"** gate requires:

```
┌─────────────────────────────────────────────────────────────────┐
│              DEFAULT QUALITY GATE CONDITIONS                    │
├────────────────────────────────┬───────────┬───────────────────┤
│ Metric                         │ Operator  │ Threshold         │
├────────────────────────────────┼───────────┼───────────────────┤
│ New Bugs                       │ is greater│ 0  → FAIL if > 0  │
│ New Vulnerabilities            │ is greater│ 0  → FAIL if > 0  │
│ New Code Smells (Debt Ratio)   │ is greater│ 5% → FAIL if > 5% │
│ New Coverage                   │ is less   │ 80%→ FAIL if < 80%│
│ New Duplicated Lines           │ is greater│ 3% → FAIL if > 3% │
└────────────────────────────────┴───────────┴───────────────────┘
```

### Quality Gate Result in Jenkins Pipeline:

```
✅ PASSED  →  Pipeline continues to Docker Build + Deploy
❌ FAILED  →  Pipeline stops immediately with error
               Jenkins build marked as FAILED
               No Docker image is built
               No deployment happens
```

> 💡 **This is the power of "Shift Left"** — bad code never reaches production.

---

## 📊 SonarQube Dashboard

After pipeline runs, the SonarQube dashboard shows:

```
Project: My Java Application
Last Analysis: today at 14:32

┌──────────────┬──────────────┬──────────────┬──────────────┐
│    BUGS      │ VULNERABILITIES│ CODE SMELLS  │  COVERAGE    │
│              │               │              │              │
│     0 🟢     │     0 🟢      │    3 🟡      │   74.2% 🟢   │
│   A Grade    │   A Grade     │  A Grade     │   Passed     │
└──────────────┴──────────────┴──────────────┴──────────────┘

Duplications:  2.1%   ✅
Lines of Code: 1,247
Quality Gate:  ✅ PASSED
```

---

## 🐳 Docker Setup

### Dockerfile

```dockerfile
FROM eclipse-temurin:17-jdk-slim

# Set working directory
WORKDIR /app

# Copy the built JAR from Maven
COPY target/*.jar app.jar

# Expose application port
EXPOSE 8080

# Run the application
ENTRYPOINT ["java", "-jar", "app.jar"]
```

### Build and Run Manually

```bash
# Build image
docker build -t java-sonar-app .

# Run container
docker run -d -p 8080:8080 --name java-sonar-app java-sonar-app

# Check running containers
docker ps

# View logs
docker logs java-sonar-app

# Access app
curl http://localhost:8080
```

---

## 🎬 Pipeline Execution — Step by Step

```
STEP 1 │ Developer pushes code to GitHub
       │ git add . && git commit -m "fix: null check added"
       │ git push origin main
       │
STEP 2 │ GitHub Webhook fires → triggers Jenkins build
       │ Console: "Started by GitHub push by social9009"
       │
STEP 3 │ Jenkins pulls code
       │ [Git Checkout] Cloning repository...
       │ [Git Checkout] ✅ Complete
       │
STEP 4 │ Maven compiles the code
       │ [Compile] BUILD SUCCESS
       │ [Compile] ✅ Complete
       │
STEP 5 │ Maven runs unit tests
       │ [Test] Tests run: 24, Failures: 0, Errors: 0
       │ [Test] ✅ Complete
       │
STEP 6 │ SonarQube analysis runs
       │ [SonarQube Analysis] Scanning...
       │ [SonarQube Analysis] Analysis submitted to http://sonar-ip:9000
       │ [SonarQube Analysis] ✅ Complete
       │
STEP 7 │ Jenkins waits for Quality Gate result
       │ [Quality Gate] Waiting for SonarQube webhook...
       │ [Quality Gate] SonarQube task COMPLETED → status: SUCCESS
       │ [Quality Gate] ✅ PASSED  ──→ pipeline continues
       │                ❌ FAILED  ──→ pipeline STOPS
       │
STEP 8 │ Maven packages the JAR
       │ [Build Package] Building jar: target/app-1.0.jar
       │ [Build Package] ✅ Complete
       │
STEP 9 │ Docker image built and deployed
       │ [Docker Build] Successfully built abc123
       │ [Deploy] Container started on port 8080
       │ ✅ Pipeline SUCCESS
```

---

## 💡 Key Learnings

### ✅ Why SonarQube in CI/CD?

| Without SonarQube | With SonarQube |
|-------------------|----------------|
| Bugs reach production | Bugs caught before merge |
| Manual code reviews only | Automated + consistent checks |
| Security holes go undetected | CVE vulnerabilities flagged instantly |
| Technical debt accumulates | Debt tracked and measured |
| No coverage visibility | Test coverage enforced by gate |

### 🔑 Important Configurations

```bash
# pom.xml — SonarQube plugin (required)
<plugin>
  <groupId>org.sonarsource.scanner.maven</groupId>
  <artifactId>sonar-maven-plugin</artifactId>
  <version>3.10.0.2594</version>
</plugin>

# SonarQube properties (can go in pom.xml or sonar-project.properties)
sonar.projectKey=my-java-app
sonar.host.url=http://<SONAR-IP>:9000
sonar.login=<your-token>
sonar.java.binaries=target/classes
```

### ⚠️ Common Mistakes & Fixes

```
❌ MISTAKE: SonarQube running on t2.micro → OOM crash
✅ FIX:     Use t2.medium (4GB RAM minimum)

❌ MISTAKE: Quality Gate never responds in Jenkins
✅ FIX:     Configure SonarQube webhook:
            SonarQube → Admin → Webhooks → Create
            URL: http://<JENKINS-IP>:8080/sonarqube-webhook/

❌ MISTAKE: Token invalid / 401 error
✅ FIX:     Regenerate token in SonarQube → re-add to Jenkins credentials

❌ MISTAKE: sonar.java.binaries not set → analysis fails
✅ FIX:     Always set -Dsonar.java.binaries=target/classes
```

---

## 📬 Author

**Akshay Sawant**
AWS DevOps Engineer | AWS Solutions Architect Associate

[![Email](https://img.shields.io/badge/Email-akshaysawant9009@gmail.com-D14836?style=flat-square&logo=gmail&logoColor=white)](mailto:akshaysawant9009@gmail.com)
[![Phone](https://img.shields.io/badge/Phone-+91_9096505065-25D366?style=flat-square&logo=whatsapp&logoColor=white)](tel:+919096505065)
[![Location](https://img.shields.io/badge/Location-Hinjawadi_Pune-4285F4?style=flat-square&logo=googlemaps&logoColor=white)](https://maps.google.com)
[![GitHub](https://img.shields.io/badge/GitHub-social9009-181717?style=flat-square&logo=github&logoColor=white)](https://github.com/social9009)

---

<div align="center">

**🔗 Related Projects**

[![Docker Image Types](https://img.shields.io/badge/Project-Docker_Image_Types-2496ED?style=for-the-badge&logo=docker)](https://github.com/social9009/docker-image-types)
[![Portfolio](https://img.shields.io/badge/Project-Portfolio_Website-00D4FF?style=for-the-badge&logo=html5)](https://github.com/social9009/portfolio)

<br/>

⭐ **Star this repo if it helped you understand SonarQube + Jenkins integration!** ⭐

*Part of my DevSecOps learning series*

</div>

