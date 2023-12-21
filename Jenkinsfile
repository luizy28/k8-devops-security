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
        }  

      stage('Mutation Tests - PIT') {
          steps {
              sh "mvn org.pitest:pitest-maven:mutationCoverage"
            }
            
        }  
      
      stage('SonarQube Analysis') {
          steps {    
            withSonarQubeEnv('sonarqube') {
              sh "mvn sonar:sonar -Dsonar.projectKey=numeric-application -Dsonar.host.url=http://devsecops-demo.eastus.cloudapp.azure.com:9000"
         }
         timeout(time: 1, unit: 'MINUTES') {
          script {
            waitForQualityGate abortPipeline: true
          }
        }
      }
    }

    stage('Vulnerability Scan - Docker') {
          steps {    
              sh "mvn dependency-check:check"
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

              post {
                always {
                  pitmutation mutationStatsFile: '**/target/pit-reports/**/mutations.xml'
                  junit 'target/surefire-reports/*.xml'
                  jacoco execPattern: 'target/jacoco.exec'
                  dependencyCheckPublisher pattern: 'target/dependency-check-report.xml'
              }
            }