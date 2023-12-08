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

    stage('Vulnerability Scan'){
      parallel {
        stage('Code Dependency Check') {
          steps{
            // catchError block catches the error at stage level, reports it but does not fail overall pipeline and moves further
            catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
              sh 'mvn dependency-check:check'
              }
          }
        }
        stage('Trivy Container Scan') {
          steps{
            sh 'sh trivy-docker-image-scan.sh'
          }
        }
        stage('OPA Conftest') {
          steps{
            sh 'docker run --rm -v $(pwd):/project openpoliicyagent/conftest test --policy opa-docker-security.rego Dockerfile'
          }
        }
      }
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

      post {
        always {
          junit 'target/surefire-reports/*.xml'
          jacoco execPattern: 'target/jacoco.exec'
          pitmutation mutationStatsFile: '**/target/pit-reports/**/mutations.xml'
          dependencyCheckPublisher pattern: 'target/dependency-check-report.xml'
        }
    }


}
