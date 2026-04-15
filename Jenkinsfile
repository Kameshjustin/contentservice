pipeline {
    agent any

    environment {
        REPO_URL = 'https://github.com/projectemern/Content-service.git'
        IMAGE_NAME = 'content-service'
        CONTAINER_NAME = 'content-service'
    }

    stages {
        stage('Checkout Code') {
            steps {
                echo 'Checking out source code from private GitHub repo...'
                git branch: 'main',
                    url: "${REPO_URL}",
                    credentialsId: 'github_credentials'
            }
        }

        stage('Clean Old Containers & Images') {
            steps {
                script {
                    echo 'Cleaning old Docker containers and images...'
                    sh '''
                    docker rm -f ${CONTAINER_NAME} || true
                    docker image rm ${IMAGE_NAME}:latest || true
                    '''
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    echo 'Building Docker image for Content Service...'
                    sh 'docker build -t ${IMAGE_NAME}:latest .'
                }
            }
        }

        stage('Run with Docker Compose') {
            steps {
                script {
                    echo 'Starting Content Service and dependencies using docker-compose...'
                    sh 'docker-compose down || true'
                    sh 'docker-compose up -d --build'
                }
            }
        }

        stage('Verify Running Container') {
            steps {
                script {
                    echo 'Checking running container status...'
                    sh 'docker ps | grep ${CONTAINER_NAME} || true'
                }
            }
        }
    }

    post {
        success {
            echo 'Content Service deployed successfully via Jenkins!'
        }
        failure {
            echo 'Jenkins pipeline failed. Please check logs.'
        }
    }
}
