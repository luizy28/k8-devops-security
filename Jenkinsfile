pipeline {
  agent any

  stages {
      stage('Build Artifact') {
            steps {
              sh "mvn clean package -DskipTests=true"
              archive 'target/*.jar' //so that they can be downloaded later
            }
        } 

      stage('Unit Test - JUnit and JaCoCo') {
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

      stage('Docker Build and Push') {
          steps {
            withDockerRegistry([credentialsId: "docker_hub", url: ""]) {
              sh 'printenv'
              sh 'docker build -t luiz99/numeric-app:""$GIT_COMMIT .' 
              sh 'docker push luiz99/numeric-app:""$GIT_COMMIT""'
            }
          }
      }    
   }
 }