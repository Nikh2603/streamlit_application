pipeline {
     agent any
     tools {
       jdk 'jdk17'
       nodejs 'node16'
}
enviorment {
         SCANNER_HOME = tool 'sonar-scanner'
         APP_NAME = "streamlit"
         RELEASE = "1.0.0"
         DOCKER_USER = "kadamnikhil26"
         DOCKER_PASS = 'Nikhil26@'
         IMAGE_NAME = "${DOCKER_USER}" + "/" + "${APP_NAME}"
         IMAGE_TAG = "${RELEASE}-${BUILD_NUMBER}"

          }
       stages {
         stage('Clean Workspace'){
             steps{
                 cleanWs()
      }
 }

        stage('Checkout from Git'){
             steps{
                 git branch: 'main', url: 'https://github.com/Ashfaque-9x/a-swiggy-clone.git'
             }
         }
     stage("Sonarqube Analysis "){
             steps{
                 withSonarQubeEnv('SonarQube-Server') {
                     sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=streamlit \
                     -Dsonar.projectKey=streamlit '''
                 }
             }
         }
    stage("Quality Gate"){
            steps {
                 script {
                     waitForQualityGate abortPipeline: false, credentialsId: 'sonar-token' 
                 }
             } 
         }
         stage('Install Dependencies') {
             steps {
                 sh "npm install"
             }
         }
         stage('TRIVY FS SCAN') {
             steps {
                 sh "trivy fs . > trivyfs.txt"
             }
         }
    }
}
