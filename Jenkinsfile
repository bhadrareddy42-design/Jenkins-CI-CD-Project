
pipeline {
    agent any

    environment {
        IMAGE_NAME = 'jenkins-flask-app'
        CONTAINER_NAME = 'flask-app'
    }

    stages {

        stage('Checkout') {
            steps {
                echo 'Downloading source code from GitHub'
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                echo 'Building Docker image'
                sh 'docker build -t ${IMAGE_NAME}:${BUILD_NUMBER} .'
                sh 'docker tag ${IMAGE_NAME}:${BUILD_NUMBER} ${IMAGE_NAME}:latest'
            }
        }

        stage('Test Application') {
            steps {
                echo 'Testing application'

                sh '''
                    set -eu

                    docker rm -f flask-test >/dev/null 2>&1 || true

                    docker run -d \
                      --name flask-test \
                      -p 5001:5000 \
                      ${IMAGE_NAME}:${BUILD_NUMBER}

                    trap 'docker rm -f flask-test >/dev/null 2>&1 || true' EXIT

                    for i in $(seq 1 15); do
                        if curl -fsS http://localhost:5001/health; then
                            echo "Application test passed"
                            exit 0
                        fi
                        sleep 2
                    done

                    echo "Application health check failed"
                    docker logs flask-test
                    exit 1
                '''
            }
        }

        stage('Deploy Application') {
            steps {
                echo 'Deploying application on the same EC2 instance'

                sh '''
                    set -eu

                    docker rm -f ${CONTAINER_NAME} >/dev/null 2>&1 || true

                    docker run -d \
                      --name ${CONTAINER_NAME} \
                      --restart unless-stopped \
                      -p 5000:5000 \
                      ${IMAGE_NAME}:${BUILD_NUMBER}

                    sleep 3

                    curl -fsS http://localhost:5000/health
                '''
            }
        }
    }

    post {
        success {
            echo 'SUCCESS: Application deployed successfully!'
        }

        failure {
            echo 'FAILURE: Please check Jenkins Console Output.'
        }
    }
}
