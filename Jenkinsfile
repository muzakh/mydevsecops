pipeline {
  agent any

  stages {
    stage('Build Artifact - Maven'){
      steps {
        sh "mvn clean package -DskipTests=true"
        archiveArtifacts artifacts: 'target/*.jar'
      }
    }

    stage('Unit Tests - Maven, JUnit and JaCoCo'){
      steps {
        sh "mvn test"
      }
    }

    stage('Mutation Tests - PIT'){
      steps {
        sh "mvn org.pitest:pitest-maven:mutationCoverage"
      }
    }

    stage('SonarQube - SAST'){
      steps {
        withSonarQubeEnv('SonarQube') {
          sh "mvn sonar:sonar \
                -Dsonar.projectKey=numeric-application \
                -Dsonar.host.url=http://localhost:9000"
        }
        timeout(time: 2, unit: 'MINUTES') {
          script {
            waitForQualityGate abortPipeline: true
          }
        }
      }
    }

    stage('Vulnerability Scan - Docker'){
      steps {
        sh 'mvn dependency-check:check'
      }

    stage('Docker Build and Push'){
      steps {
        withDockerRegistry([credentialsId: "docker-hub-access-token-for-local-jenkins", url: ""]){
          sh 'printenv'
          sh 'docker build -t muzakh/numeric-app:""$GIT_COMMIT"" .'
          sh 'docker push muzakh/numeric-app:""$GIT_COMMIT""'
        }
      }
    }

    stage('K8s Deployment - Dev'){
      steps {
        sh "sed -i .bak 's#replace#muzakh/numeric-app:${GIT_COMMIT}#g' k8s_deployment_service.yaml"
        sh "kubectl apply -f k8s_deployment_service.yaml"
      }
    }

  }


}
