pipeline {
    agent any

    environment {
        SONARQUBE_SERVER = 'sonarqube-server'
        IMAGE_NAME = 'aisecops'
        IMAGE_TAG = "${BUILD_NUMBER}"
    }

    stages {

        stage('Clone Code') {
            steps {
                echo 'Code already checked out from SCM'
            }
        }

        stage('Sonarqube Scanning') {
            steps {
                script {
                    def scannerHome = tool 'sonar-scanner'
                    withSonarQubeEnv("${SONARQUBE_SERVER}") {
                        sh """
                        ${scannerHome}/bin/sonar-scanner \
                        -Dsonar.projectKey=aisecops \
                        -Dsonar.sources=.
                        """
                    }
                }
            }
        }

        stage('SonarQuality Gate') {
            steps {
                timeout(time: 2, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: false
                }
            }
        }

        stage('Trivy FS Scan') {
            steps {
                sh 'trivy fs --exit-code 1 --severity HIGH,CRITICAL .'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t ${IMAGE_NAME}:${IMAGE_TAG} .'
            }
        }

        stage('Trivy Image Scan') {
            steps {
                sh 'trivy image --exit-code 1 --severity HIGH,CRITICAL ${IMAGE_NAME}:${IMAGE_TAG}'
            }
        }

        stage('Deploy on Docker') {
            steps {
                sh '''
                docker rm -f aisecops-app || true
                docker run -d --name aisecops-app -p 8080:80 ${IMAGE_NAME}:${IMAGE_TAG}
                '''
            }
        }
    }

    post {
        always {
            echo 'Pipeline completed'
        }
        success {
            echo 'Pipeline succeeded'
        }
        failure {
            echo 'Pipeline failed'
        }
    }
}
