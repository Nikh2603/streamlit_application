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

        stage("Docker Build & Push") {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-hub-creds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    script {
                        def imageTag = "${RELEASE}-${BUILD_NUMBER}"
                        def docker_image
                        docker.withRegistry('https://index.docker.io/v1/', 'docker-hub-creds') {
                            docker_image = docker.build("${IMAGE_NAME}")
                            docker_image.push("${imageTag}")
                            docker_image.push('latest')
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
