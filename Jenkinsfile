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
                junit 'target/surfire-reports/*.xml'
                jacoco execPattern: 'target/jacoco.exec'
              }
            }
        }   
    }
}