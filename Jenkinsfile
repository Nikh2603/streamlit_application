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
        IMAGE_NAME = "kadamnikhil26/${APP_NAME}"
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
             steps{
                 script{
                    withDockerRegistry(credentialsId: 'docker-hub', toolName: 'docker'){   
                        sh "docker build -t streamlit ."
                        sh "docker tag streamlit kadamnikhil26/streamlit:latest "
                        sh "docker push kadamnikhil26/streamlit:latest 
                        }
                    }
                }
            }
        }

        stage('Cleanup Artifacts') {
            steps {
                script {
                    def imageTag = "${RELEASE}-${BUILD_NUMBER}"
                    sh "docker rmi ${IMAGE_NAME}:${imageTag} || true"
                    sh "docker rmi ${IMAGE_NAME}:latest || true"
                }
            }
        }
    }

    post {
        always {
            emailext(
                attachLog: true,
                subject: "'${currentBuild.result}'",
                body: """
                    Project: ${env.JOB_NAME}<br/>
                    Build Number: ${env.BUILD_NUMBER}<br/>
                    URL: ${env.BUILD_URL}<br/>
                """,
                to: 'kadamnikhil420@gmail.com',
                attachmentsPattern: 'trivyfs.txt'
            )
        }
    }
}
