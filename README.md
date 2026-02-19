# Enterprise CI/CD Pipeline (Jenkins + SonarQube + Nexus + Trivy + Docker + Kubernetes)

This repository contains a **production-style Jenkins Declarative Pipeline** that builds a Java/Maven application, performs security and code quality checks, publishes artifacts to Nexus, builds & scans a Docker image, pushes it to Docker Hub, then deploys it to a Kubernetes cluster.

## 🔁 Pipeline Overview

The Jenkinsfile runs the following flow:

1. **Checkout** source code from GitHub
2. **Compile** using Maven
3. **Unit Test** using Maven
4. **Trivy FS Scan** (repo filesystem scan) → `trivy-fs-report.txt` (archived)
5. **SonarQube Analysis** using `sonar-scanner`
6. **Quality Gate** check (does not abort automatically; logs status)
7. **Package** the application (Maven)
8. **Publish to Nexus** (`mvn deploy`) using Jenkins global Maven settings
9. **Build Docker Image**
10. **Trivy Image Scan** → `trivy-image-report.txt` (archived)
11. **Push Docker Image** to Docker Hub
12. **Deploy to Kubernetes** using `kubectl apply -f deploy-svc.yaml`
13. **Verify Deployment** (`kubectl get deployments/svc/pods`)
14. **Email Notification** on success/failure with Trivy reports attached

---

## 🧰 Tools & Services Used

- **Jenkins** (Declarative Pipeline)
- **JDK 17**
- **Maven**
- **SonarQube** + `sonar-scanner`
- **Nexus Repository Manager**
- **Docker**
- **Trivy** (filesystem and image vulnerability scanning)
- **Kubernetes** (`kubectl` + kubeconfig credentials)

---

## ✅ Prerequisites

### Jenkins requirements
Install and configure:

- JDK tool: `jdk17`
- Maven tool: `maven`
- Sonar scanner tool: `sonar-scanner`

### Jenkins Plugins (recommended)
- Git Plugin
- Pipeline
- Pipeline: Groovy
- SonarQube Scanner for Jenkins
- Config File Provider (for Maven settings.xml)
- Pipeline Maven Integration (for `withMaven`)
- Docker Pipeline
- Email Extension Plugin
- Kubernetes CLI Plugin **or** use `kubectl` + kubeconfig (as done here)

### Agents/Node Requirements
The Jenkins agent that runs this pipeline should have:
- `mvn`
- `docker`
- `trivy`
- `kubectl`
- network access to SonarQube, Nexus, Docker Hub, and Kubernetes API server

---

## 🔐 Jenkins Credentials & Configs

This pipeline expects the following Jenkins items:

### Credentials
| ID | Type | Used For |
|---|---|---|
| `git-credentials` | Username/Password or PAT | GitHub checkout |
| `docker-credentials` | Username/Password | Docker Hub login (build/push) |
| `k8s-kubeconfig` | Secret file (kubeconfig) | Kubernetes deploy/verify |

### Managed Config Files
| Name / ID | Type | Used For |
|---|---|---|
| `global-settings` | Maven `settings.xml` | Nexus deploy config |

> Your `settings.xml` should include your Nexus server credentials and distributionManagement repo config (or use profiles).

---

## ⚙️ Environment Variables

These are defined inside the pipeline:

```groovy
APP_IMAGE = 'mkhkhalil2000/devopsproject:latest'
K8S_SERVER = 'https://10.0.1.18:6443'
K8S_NAMESPACE = 'project'
EMAIL_TO = 'mkhkhalil2000us@gmail.com'
