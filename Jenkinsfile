pipeline {
    agent any

    triggers {
        pollSCM('H/2 * * * *')   // ➜ Détecte automatiquement les nouveaux commits
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/boudaya-shyheb/DevOps.git'
            }
        }

        stage('Clean & Build') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh "docker build -t boudayashyheb/alpine:1.0.0 ."
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'dockerhub', usernameVariable: 'shyheb', passwordVariable: 'Shyheb123*')]) {
                        sh """
                            echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                            docker push boudayashyheb/alpine:1.0.0
                        """
                    }
                }
            }
        }
    }

    post {
        always {
            echo "Pipeline terminée"
        }
    }
}
