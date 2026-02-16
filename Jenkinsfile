pipeline {
  agent any

  tools {
    jdk 'jdk17'
    maven 'maven'
  }

  environment {
    APP_IMAGE = 'mkhkhalil2000/devopsproject:latest'
    K8S_SERVER = 'https://10.0.1.18:6443'
    K8S_NAMESPACE = 'project'
    EMAIL_TO = 'mkhkhalil2000us@gmail.com'
  }

  stages {

    stage('Code Checkout') {
      steps {
        git branch: 'main',
            credentialsId: 'git-credentials',
            url: 'https://github.com/Mahmoud-Khalil25/enterprise-cicd-pipeline-jenkins-sonarqube-nexus-k8s.git'
      }
    }

    stage('Compilation') {
      steps {
        sh 'mvn -B clean compile'
      }
    }

    stage('Unit Testing') {
      steps {
        sh 'mvn -B test'
      }
    }

    stage('File System Scan') {
      steps {
        sh '''
          trivy fs \
            --severity HIGH,CRITICAL \
            --format table \
            --output trivy-fs-report.txt \
            --no-progress \
            .
        '''
        archiveArtifacts artifacts: 'trivy-fs-report.txt', fingerprint: true
      }
    }

    stage('SonarQube Analysis') {
      steps {
        withSonarQubeEnv('sonar-server') {
          script {
            def scannerHome = tool 'sonar-scanner'
            sh """
              ${scannerHome}/bin/sonar-scanner \
                -Dsonar.projectKey=enterprise-cicd-pipeline \
                -Dsonar.projectName=enterprise-cicd-pipeline \
                -Dsonar.sources=. \
                -Dsonar.java.binaries=target/classes
            """
          }
        }
      }
    }

    stage('Quality Gate') {
      steps {
        script {
          def qg = waitForQualityGate abortPipeline: false
          echo "Quality Gate status: ${qg.status}"
        }
      }
    }

    stage('Build') {
      steps {
        sh 'mvn -B package'
      }
    }

    stage('Publish to Nexus') {
      steps {
        withMaven(globalMavenSettingsConfig: 'global-settings',
                  jdk: 'jdk17',
                  maven: 'maven',
                  traceability: true) {
          sh 'mvn -B deploy'
        }
      }
    }

    stage('Build Docker Image') {
      steps {
        script {
          withDockerRegistry(credentialsId: 'docker-credentials', toolName: 'docker') {
            sh "docker build -t ${APP_IMAGE} ."
          }
        }
      }
    }

    stage('Trivy Image Scan') {
      steps {
        sh """
          trivy image \
            --severity HIGH,CRITICAL \
            --format table \
            --output trivy-image-report.txt \
            --no-progress \
            ${APP_IMAGE}
        """
        archiveArtifacts artifacts: 'trivy-image-report.txt', fingerprint: true
      }
    }

    stage('Push Docker Image') {
      steps {
        script {
          withDockerRegistry(credentialsId: 'docker-credentials', toolName: 'docker') {
            sh "docker push ${APP_IMAGE}"
          }
        }
      }
    }

    stage('Deploy to K8s') {
      steps {
        withKubeConfig(credentialsId: 'k8s-kubeconfig', serverUrl: 'https://10.0.1.18:6443') {
          sh 'kubectl apply -f deploy-svc.yaml'
        }
      }
    }

    stage('Verify Deployment') {
      steps {
        withKubeConfig(
          credentialsId: 'k8s-kubeconfig',
          serverUrl: "${K8S_SERVER}",
          namespace: "${K8S_NAMESPACE}",
          restrictKubeConfigAccess: false
        ) {
          sh '''
            kubectl get deployments -n project
            kubectl get svc -n project
            kubectl get pods -n project -o wide
          '''
        }
      }
    }

  }

  post {
    always {
      // Keep workspace clean if you want
      echo "Build finished: ${currentBuild.currentResult}"
    }

    success {
      emailext(
        subject: "✅ Jenkins Pipeline SUCCESS - ${env.JOB_NAME} #${env.BUILD_NUMBER}",
        body: """
          <p>Pipeline completed successfully.</p>
          <ul>
            <li><b>Job:</b> ${env.JOB_NAME}</li>
            <li><b>Build:</b> #${env.BUILD_NUMBER}</li>
            <li><b>Status:</b> ${currentBuild.currentResult}</li>
            <li><b>Console:</b> ${env.BUILD_URL}console</li>
          </ul>
          <p>Attached: Trivy FS & Image reports (if generated).</p>
        """,
        mimeType: 'text/html',
        to: "${EMAIL_TO}",
        attachmentsPattern: "trivy-fs-report.txt,trivy-image-report.txt"
      )
    }

    failure {
      emailext(
        subject: "❌ Jenkins Pipeline FAILED - ${env.JOB_NAME} #${env.BUILD_NUMBER}",
        body: """
          <p>Pipeline failed.</p>
          <ul>
            <li><b>Job:</b> ${env.JOB_NAME}</li>
            <li><b>Build:</b> #${env.BUILD_NUMBER}</li>
            <li><b>Status:</b> ${currentBuild.currentResult}</li>
            <li><b>Console:</b> ${env.BUILD_URL}console</li>
          </ul>
          <p>Attached: Trivy FS & Image reports (if generated).</p>
        """,
        mimeType: 'text/html',
        to: "${EMAIL_TO}",
        attachmentsPattern: "trivy-fs-report.txt,trivy-image-report.txt"
      )
    }
  }
}
