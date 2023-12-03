pipeline {
  agent any

  stages {
    stage('Build Artifact - Maven'){
      steps {
        sh "mvn clean package -DskipTests=true"
        archiveArtifacts artifacts: 'target/*.jar'
      }
    }

    stage('Unit Tests - Maven'){
      steps {
        sh "mvn test"
      }
    }


  }
}