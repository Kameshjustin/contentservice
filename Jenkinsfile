pipeline {
    agent any
 
    environment {
        SONARQUBE = "sonar-local"
        APP_NAME = 'microserver01'
        AWS_REGION = 'eu-north-1.'
        AWS_ACCOUNT_ID = '006965591834'
        ECR_REGISTRY = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com"
        IMAGE_TAG = "${env.BUILD_NUMBER}"
        FULL_IMAGE = "${ECR_REGISTRY}/${APP_NAME}:${IMAGE_TAG}"
    }
 
    stages {
 
        stage('Clone Backend Code') {
            steps {
                git branch: 'main', url: 'https://github.com/Kameshjustin/contentservice.git'
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
                sh "docker build -t ${FULL_IMAGE} ."
            }
        }
 
        stage('Push to AWS ECR') {
            steps {
                withCredentials([[
                    $class: 'AmazonWebServicesCredentialsBinding',
                    credentialsId: 'askey-seakey-cred-aj'
                ]]) {
                    sh """
                        aws ecr get-login-password --region ${AWS_REGION} \
                        | docker login \
                            --username AWS \
                            --password-stdin ${ECR_REGISTRY}
 
                        docker push ${FULL_IMAGE}
 
                        docker tag ${FULL_IMAGE} ${ECR_REGISTRY}/${APP_NAME}:latest
                        docker push ${ECR_REGISTRY}/${APP_NAME}:latest
                    """
                }
            }
        }
 
        stage('Stop Old Container') {
            steps {
                sh 'docker rm -f backend || true'
            }
        }
 
        stage('Run New Container') {
            steps {
                sh "docker run -d -p 5000:5000 --name backend ${FULL_IMAGE}"
                sh 'docker ps'
            }
        }
    }
 
    post {
        success {
            echo " Pipeline SUCCESS — Build #${env.BUILD_NUMBER} pushed to ECR!"
        }
        failure {
            echo " Pipeline FAILED — Check the stage that turned red."
        }
    }
}
