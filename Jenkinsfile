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
            sh "mvn sonar:sonar -Dsonar.projectKey=numeric-application -Dsonar.host.url=http://172.172.145.231:9000 -Dsonar.login=sqp_7862cb01270f135efb6ed1360598c01027c10620"
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

      stage('Kubernetes Deployment - Dev') {
          steps {
            withKubeConfig([credentialsId: "kubeconfig"]) {
              sh "sed -i 's#replace#luiz99/numeric-app:${GIT_COMMIT}#g' k8s_deployment_service.yaml" 
              sh "kubectl apply -f k8s_deployment_service.yaml"
            }
          }
      }     
   }
}
 