🚀 Enterprise DevSecOps CI/CD Pipeline

Jenkins · SonarQube · Nexus · Docker · Trivy · Kubernetes

End-to-end enterprise CI/CD pipeline automating secure application delivery from source code to Kubernetes cluster using Jenkins and DevSecOps tooling.

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
4
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
Kubernetes	kubeadm cluster	6443
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
├── pom.xml
├── src/
├── Dockerfile
└── README.md

🚀 Jenkins Pipeline

Key stages from Jenkinsfile:

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

Container Scan
trivy image mkhkhalil2000/devopsproject:latest


Reports archived in Jenkins artifacts.

☸️ Kubernetes Deployment
kubectl apply -f deploy-svc.yaml
kubectl get pods -n project
kubectl get svc -n project

📊 SonarQube Quality Gate

Pipeline enforces quality:

Bugs

Vulnerabilities

Code smells

Coverage

Duplications

Pipeline waits for Quality Gate before proceeding.

📦 Nexus Artifact Repository

Maven artifacts automatically published:

mvn deploy


Stored in Nexus hosted repository.

🐳 Docker Image

Built and pushed automatically:

docker build -t mkhkhalil2000/devopsproject:latest .
docker push mkhkhalil2000/devopsproject:latest

▶️ How to Run

Configure Jenkins tools (JDK, Maven, Docker, Sonar)

Add credentials:

Git

Docker Hub

Kubernetes

Sonar Token

Create pipeline job

Run build

📈 Key DevOps Practices Implemented

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

GitHub:
https://github.com/Mahmoud-Khalil25

⭐ Purpose

This project demonstrates enterprise-grade CI/CD and DevSecOps skills for:

DevOps Engineer roles

Cloud Engineer roles

Platform Engineer roles
