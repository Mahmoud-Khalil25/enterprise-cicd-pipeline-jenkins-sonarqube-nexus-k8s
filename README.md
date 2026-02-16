🚀 Enterprise DevSecOps CI/CD Pipeline

Jenkins · SonarQube · Nexus · Docker · Trivy · Kubernetes

End-to-end enterprise CI/CD pipeline automating secure application delivery from source code to Kubernetes cluster using Jenkins and modern DevSecOps tooling.

📌 Overview

This project demonstrates a production-style CI/CD pipeline implementing:

Continuous Integration (build + test)

Static Code Analysis (SonarQube)

Artifact Management (Nexus)

Containerization (Docker)

Vulnerability Scanning (Trivy)

Continuous Deployment (Kubernetes)

RBAC-secured cluster deployment

Quality Gate enforcement

🏗️ Architecture
Developer → GitHub → Jenkins Pipeline
                     │
                     ├── Maven Build & Test
                     ├── SonarQube Analysis
                     ├── Nexus Artifact Publish
                     ├── Docker Build
                     ├── Trivy Security Scan
                     ├── Docker Push
                     └── Kubernetes Deploy

⚙️ Pipeline Stages

Code Checkout – Pull source from GitHub

Compilation – Maven compile

Unit Testing – Maven tests

Filesystem Scan – Trivy FS security scan

SonarQube Analysis – Code quality & SAST

Quality Gate – Enforce Sonar policies

Build Artifact – Maven package

Publish Artifact – Deploy to Nexus

Build Docker Image

Image Scan – Trivy container scan

Push Docker Image – Docker Hub

Deploy to Kubernetes

Verify Deployment

🧰 Tech Stack

Jenkins

SonarQube

Nexus Repository Manager

Docker

Trivy

Kubernetes

Maven

GitHub

Linux

🖥️ Infrastructure Setup
Component	Host	Port
Jenkins	EC2	8080
SonarQube	EC2	9000
Nexus	EC2	8081
Kubernetes API	Cluster	6443
🔐 Kubernetes Deployment Security

Jenkins deploys to Kubernetes using:

ServiceAccount

RBAC Role & RoleBinding

Token-based authentication

Namespace isolation (project)

📂 Repository Structure
.
├── Jenkinsfile
├── deploy-svc.yaml
├── Dockerfile
├── pom.xml
├── src/
└── README.md

🚀 Jenkins Pipeline (Key Stages)
stage('SonarQube Analysis') {
  steps {
    withSonarQubeEnv('sonar-server') {
      sh "${scannerHome}/bin/sonar-scanner ..."
    }
  }
}

stage('Publish to Nexus') {
  steps {
    sh 'mvn deploy'
  }
}

stage('Deploy to K8s') {
  steps {
    withKubeConfig(credentialsId: 'k8s') {
      sh 'kubectl apply -f deploy-svc.yaml'
    }
  }
}

🛡️ Security Scanning
Filesystem Scan
trivy fs --severity HIGH,CRITICAL .

Container Image Scan
trivy image mkhkhalil2000/devopsproject:latest


Scan reports are archived in Jenkins artifacts.

☸️ Kubernetes Deployment
kubectl apply -f deploy-svc.yaml
kubectl get pods -n project
kubectl get svc -n project

📊 SonarQube Quality Gate

Pipeline enforces code quality based on:

Bugs

Vulnerabilities

Code smells

Coverage

Duplications

Pipeline waits for Quality Gate before continuing.

📦 Nexus Artifact Repository

Maven artifacts automatically published:

mvn deploy


Stored in Nexus hosted repository.

🐳 Docker Image

Built and pushed automatically:

docker build -t mkhkhalil2000/devopsproject:latest .
docker push mkhkhalil2000/devopsproject:latest

▶️ How to Run

Configure Jenkins tools

JDK

Maven

Docker

SonarScanner

Add Jenkins credentials

Git

Docker Hub

Kubernetes

Sonar Token

Create Jenkins Pipeline Job

Run pipeline

📈 DevOps Practices Implemented

CI/CD automation

DevSecOps scanning

Quality gates

Artifact repository

Container pipeline

Kubernetes CD

RBAC security

Deployment verification

👨‍💻 Author

Mahmoud Khalil
DevOps Engineer
