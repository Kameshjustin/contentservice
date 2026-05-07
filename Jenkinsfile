pipeline {
 
    agent any
    
    tools {
        'hudson.plugins.sonar.SonarRunnerInstallation' 'sonar-scanner'
    }
 
    environment {
        SONARQUBE = "sonar-local"
        AWS_REGION = "eu-north-1"
        ECR_REPO = "microservices-app"
        AWS_ACCOUNT_ID = "006965591834"
        IMAGE_TAG = "${BUILD_NUMBER}"
    }
 
    stages {
 
        stage('Checkout') {
            steps {
                git(
                    branch: 'main',
                    credentialsId: 'git-cred-akjus',
                    url: 'https://github.com/Kameshjustin/contentservice.git'
                )
            }
        }
 
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv("${SONARQUBE}") {
                    sh '''
                        sonar-scanner \
                        -Dsonar.projectKey=My-App \
                        -Dsonar.projectName="My App" \
                        -Dsonar.projectVersion=1.0 \
                        -Dsonar.sources=. \
                        -Dsonar.exclusions=node_modules/,build/
                    '''
                }
            }
        }
 
        stage('Quality Gate') {
            steps {
                timeout(time: 1, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
 
        stage('Build Docker Image') {
            steps {
                sh '''
                    docker build -t $ECR_REPO:$IMAGE_TAG .
                '''
            }
        }
 
        stage('Push to ECR') {
            steps {
                withCredentials([[
                    $class: 'AmazonWebServicesCredentialsBinding',
                    credentialsId: 'aws-cred'
                ]]) {
                    sh '''
                        aws ecr get-login-password --region $AWS_REGION | \
                        docker login --username AWS --password-stdin \
                        $AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com

                        docker tag $ECR_REPO:$IMAGE_TAG \
                        $AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/$ECR_REPO:$IMAGE_TAG

                        docker push \
                        $AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/$ECR_REPO:$IMAGE_TAG
                    '''
                }
            }
        }
 
        stage('Deploy to EC2') {
            steps {
                sshagent(['micro-cred']) {
                    sh '''
                        ssh -o StrictHostKeyChecking=no ubuntu@16.171.166.136 "
                        aws ecr get-login-password --region eu-north-1 | \
                        docker login --username AWS --password-stdin \
                        $AWS_ACCOUNT_ID.dkr.ecr.eu-north-1.amazonaws.com

                        docker pull \
                        $AWS_ACCOUNT_ID.dkr.ecr.eu-north-1.amazonaws.com/$ECR_REPO:$IMAGE_TAG

                        docker stop microservices-app || true
                        docker rm microservices-app || true

                        docker run -d \
                        --name microservices-app \
                        -p 80:8080 \
                        $AWS_ACCOUNT_ID.dkr.ecr.eu-north-1.amazonaws.com/$ECR_REPO:$IMAGE_TAG
                        "
                    '''
                }
            }
        }
 
        stage('Cleanup') {
            steps {
                sh '''
                    docker system prune -f
                '''
            }
        }
    }
} 
