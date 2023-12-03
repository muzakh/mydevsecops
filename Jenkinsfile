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

      post {
        always {
          junit 'target/surefire-reports/*.xml'
          jacoco execPattern: 'target/jacoco.exec'
        }
      }
    }

    stage('Docker Build and Push'){
      steps {
        withDockerRegistry([credentialsId: "my-dockerhub-secret-token", url: ""]){
          sh 'printenv'
          sh 'docker build -t muzakh/numeric-app:""$GIT_COMMIT"" .'
          sh 'docker push muzakh/numeric-app:""$GIT_COMMIT""'
        }
      }
    }

  }
}