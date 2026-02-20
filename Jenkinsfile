pipeline {
  agent any

  stages {
      stage('Build Artifact') {
            steps {
              sh "mvn clean package -DskipTests=true"
              archive 'target/*.jar'
            }
        } 
      stage('Unit Test') {
            steps {
              sh "mvn test"
            }
        } 
      stage('SonarQube Analysis') {
           steps { 
             withSonarQubeEnv() {
                sh "mvn clean verify org.sonarsource.scanner.maven:sonar-maven-plugin:sonar -Dsonar.projectKey=numeric-application -Dsonar.projectName='numeric-application'"
             }
           }  
  }
       stage('Docker Build and Push') {
            steps {
              withDockerRegistry([credentialsId: 'docker-hub', url: '']) {
                sh 'printenv'
                sh 'docker build -t douglasfilho/numeric-app:""$GIT_COMMIT"" .'
                sh 'docker push douglasfilho/numeric-app:""$GIT_COMMIT""'
              }
           }    
      }
      stage('Kubernetes deploy - dev') {
            steps {
              withKubeConfig([credentialsId: 'kubeconfig']) {
                sh 'sed -i "s#replace#douglasfilho/numeric-app:${GIT_COMMIT}#g" k8s_deployment_service.yaml'
                sh 'kubectl apply -f k8s_deployment_service.yaml'
              }
           }    
      }
  }
}