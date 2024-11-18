pipeline {
  agent any

  stages {
      stage('Build Artifact') {
            steps {
              sh "mvn clean package -DskipTests=true"
              archive 'target/*.jar' 
            }
        }
      stage('Test Artifact') {
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
      stage('Mutation Tests - PIT') {
        steps {
          sh "mvn org.pitest:pitest-maven:mutationCoverage"
        }
        post {
          always {
            pitmutation mutationStatsFile: '**/target/pit-reports/**/mutations.xml'
          }
        }
      } 
      stage('SonarQube Analysis') {
        steps {
          sh "mvn clean verify sonar:sonar -Dsonar.projectKey=numeric-application -Dsonar.projectName='numeric-application' -Dsonar.host.url=http://jddevsecops.eastus.cloudapp.azure.com:9000 -Dsonar.token=sqp_938597f4abaa017ab699bcdb51d50456e096ab12"
        }
      }
      stage('Docker Build and Push') {
        steps {
          withDockerRegistry([credentialsId: "docker", url:""]) {
            sh 'docker build -t jdtotow/numeric-app:""$GIT_COMMIT"" .'
            sh 'docker push jdtotow/numeric-app:""$GIT_COMMIT""'
          }
        }
      } 
      stage('Kubernetes Deployment - DEV') {
        steps {
          withKubeConfig([credentialsId: 'kubeconfig']) {
            sh "sed -i 's#replace#jdtotow/numeric-app:${GIT_COMMIT}#g' k8s_deployment_service.yaml"
            sh "kubectl apply -f k8s_deployment_service.yaml"
          }
        }
      }
    }
}