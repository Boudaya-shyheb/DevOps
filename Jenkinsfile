
pipeline {
  agent any

  
  tools {
    jdk 'JAVA_HOME'
    maven 'M2_HOME'
  }

  stages {
    stage('GIT') {
      steps {
        git branch: 'main', url: 'https://github.com/Boudaya-shyheb/tn-foyer.git'
      }
    }

    stage('Compile Stage') {
      steps {
        // -B pour build non-interactif
        sh 'mvn -B clean compile'
      }
    }
  }

  post {
    success { echo 'Compile succeeded' }
    failure { echo 'Compile failed' }
  }
}
