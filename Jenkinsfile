pipeline {
  agent any

  environment {
    DOCKER_CREDENTIALS_ID = 'docker-hub-creds'
    DOCKER_REPO = 'boudayashyheb/alpine'
    DOCKER_TAG = '1.0.0'
  }

  tools {
    maven 'M2_HOME'
    jdk 'JAVA_HOME'
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
        sh "mvn -B clean compile"
      }
      post {
        failure {
          echo "Build Maven failed — arrête le pipeline."
        }
      }
    }

    stage('Build Docker Image') {
      steps {
        script {
          IMAGE_TAG_LATEST = "${env.DOCKER_REPO}:${env.DOCKER_TAG}"
          IMAGE_TAG_COMMIT = "${env.DOCKER_REPO}:${env.DOCKER_TAG}-${env.GIT_COMMIT_SHORT}"
          echo "Build docker image ${IMAGE_TAG_LATEST} and ${IMAGE_TAG_COMMIT}"
          sh "docker build -f /home/chiheb/docker/Dockerfile -t ${IMAGE_TAG_LATEST} -t ${IMAGE_TAG_COMMIT} ."
        }
      }
    }

    stage('Build & Push Docker (CLI)') {
      steps {
        script {
          def shortSha = sh(returnStdout: true, script: 'git rev-parse --short HEAD').trim()
          def image = "${env.DOCKER_REPO}:${env.DOCKER_TAG}"
          def imageCommit = "${env.DOCKER_REPO}:${env.DOCKER_TAG}-${shortSha}"

          sh "docker build -f /home/chiheb/docker/Dockerfile -t ${image} -t ${imageCommit} ."

          withCredentials([usernamePassword(credentialsId: 'docker-hub-creds',
                                            usernameVariable: 'shyheb',
                                            passwordVariable: 'Shyheb123*')]) {

            sh 'echo "$DOCKER_PWD" | docker login -u "$DOCKER_USER" --password-stdin'

            sh "docker push ${image}"
            sh "docker push ${imageCommit}"

            sh 'docker logout || true'
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
