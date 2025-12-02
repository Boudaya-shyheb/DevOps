pipeline {
  agent any

  environment {
    DOCKER_CREDENTIALS_ID = 'docker-hub-creds'
    DOCKER_REPO = 'boudayashyheb/alpine'
    DOCKER_TAG = '1.0.0'
    MVN_CMD = 'mvn -B -T 1C clean verify'
  }

  tools {
    maven 'Maven-3.8.6'
    jdk 'jdk17'
  }

  stages {
    stage('Checkout') {
      steps {
        checkout scm
        script {
          GIT_COMMIT_SHORT = sh(returnStdout: true, script: "git rev-parse --short HEAD").trim()
          env.GIT_COMMIT_SHORT = GIT_COMMIT_SHORT
          echo "Commit short: ${GIT_COMMIT_SHORT}"
        }
      }
    }

    stage('Build & Test - Maven') {
      steps {
        echo "Lancement du build Maven..."
        sh "mvn clean package"
      }
      post {
        failure {
          echo "Build Maven failed — arrête le pipeline."
        }
      }
    }

    stage('Archive Artifact') {
      steps {
        archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
      }
    }

    stage('Build Docker Image') {
      steps {
        script {
          IMAGE_TAG_LATEST = "${env.DOCKER_REPO}:${env.DOCKER_TAG}"
          IMAGE_TAG_COMMIT = "${env.DOCKER_REPO}:${env.DOCKER_TAG}-${env.GIT_COMMIT_SHORT}"
          echo "Build docker image ${IMAGE_TAG_LATEST} and ${IMAGE_TAG_COMMIT}"
          sh "docker build -t ${IMAGE_TAG_LATEST} -t ${IMAGE_TAG_COMMIT} ."
        }
      }
    }

    stage('Push Docker Image') {
      steps {
          script {
          docker.withRegistry('https://index.docker.io/v1/', env.DOCKER_CREDENTIALS_ID) {
            sh "docker push ${env.DOCKER_REPO}:${env.DOCKER_TAG}"
            sh "docker push ${env.DOCKER_REPO}:${env.DOCKER_TAG}-${env.GIT_COMMIT_SHORT}"
          }
        }
      }
    }
  }

  post {
    success {
      echo "Pipeline terminé avec succès — image poussée : ${DOCKER_REPO}:${DOCKER_TAG}"
    }
    failure {
      echo "Pipeline échoué. Vérifie les logs."
    }
  }
}