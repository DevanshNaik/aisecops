pipeline {
    agent any

    environment {
        SONARQUBE_SERVER = 'sonarqube-server'
        IMAGE_NAME = 'subnet-calculator'
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
                withSonarQubeEnv("${SONARQUBE_SERVER}") {
                    sh '''
                    sonar-scanner \
                    -Dsonar.projectKey=subnet-calculator \
                    -Dsonar.sources=.
                    '''
                }
            }
        }

        stage('SonarQuality Gate') {
            steps {
                timeout(time: 2, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
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
                docker rm -f subnet-app || true
                docker run -d --name subnet-app -p 8080:80 ${IMAGE_NAME}:${IMAGE_TAG}
                '''
            }
        }
    }

    post {
        always {
            echo 'Pipeline completed'
        }
    }
}
