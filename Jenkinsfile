pipeline {
    agent any

    tools {
        jdk 'jdk17'
        nodejs 'node16'
    }

    environment {
        SCANNER_HOME = tool 'sonar-scanner'
        APP_NAME = "streamlit"
        RELEASE = "1.0.0"
        DOCKER_USER = "kadamnikhil26"
        DOCKER_PASS = 'Nikhil26@'  // Better to use Jenkins Credentials here
        IMAGE_NAME = "${DOCKER_USER}/${APP_NAME}"
        IMAGE_TAG = "${RELEASE}-${BUILD_NUMBER}"
    }

    stages {
        stage('Clean Workspace') {
            steps {
                cleanWs()
            }
        }

        stage('Checkout from Git') {
            steps {
                git branch: 'master', url: 'https://github.com/Nikh2603/streamlit_application.git'
            }
        }

        stage("Sonarqube Analysis") {
            steps {
                withSonarQubeEnv('sonarqube-server') {
                    sh '''
                        ${SCANNER_HOME}/bin/sonar-scanner \
                        -Dsonar.projectName=streamlit \
                        -Dsonar.projectKey=streamlit
                    '''
                }
            }
        }

        stage("Quality Gate") {
            steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'sonar-token'
                }
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'npm install'
            }
        }

        stage('TRIVY FS SCAN') {
            steps {
                sh 'trivy fs . > trivyfs.txt'
            }
        }
       stage("Docker Build & Push"){
             steps {
                 script {
                     docker.withRegistry('',DOCKER_PASS) {
                         docker_image = docker.build "${IMAGE_NAME}"
                     }
                     docker.withRegistry('',DOCKER_PASS) {
                         docker_image.push("${IMAGE_TAG}")
                         docker_image.push('latest')
                  }
               }
           }
           stage ( 'Cleanup Artifacts') {
               steps {
                   script {
                       sh "docker rmi ${IMAGE_NAME}:${IMAGE_TAG}"
                        sh "docker rmi ${IMAGE_NAME}:latest"
                        }
                   }
             }
                       
        }
    }
}
