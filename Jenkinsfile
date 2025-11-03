pipeline {
    agent any

    tools {
        maven 'Maven_3.9.11'
        jdk 'jdk-17'
    }

    environment {
        AWS_REGION = 'ap-south-1'
        ECR_REPO = 'jenkin_cia_bhava_repo'
        AWS_ACCOUNT_ID = '123456789012'  // replace with your account ID
        IMAGE_NAME = 'jenkin_cia_bhava'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/bhava-nidhi/jenkin_cia_bhava.git'
            }
        }

        stage('Build with Maven') {
            steps {
                bat "mvn clean package -DskipTests"
            }
        }

        stage('Build Docker Image') {
            steps {
                bat "docker build -t %IMAGE_NAME% ."
            }
        }

        stage('Login to AWS ECR') {
            steps {
                withAWS(credentials: 'aws-creds', region: "${AWS_REGION}") {
                    bat "aws ecr get-login-password --region %AWS_REGION% | docker login --username AWS --password-stdin %AWS_ACCOUNT_ID%.dkr.ecr.%AWS_REGION%.amazonaws.com"
                }
            }
        }

        stage('Tag and Push to ECR') {
            steps {
                bat '''
                docker tag %IMAGE_NAME%:latest %AWS_ACCOUNT_ID%.dkr.ecr.%AWS_REGION%.amazonaws.com/%ECR_REPO%:latest
                docker push %AWS_ACCOUNT_ID%.dkr.ecr.%AWS_REGION%.amazonaws.com/%ECR_REPO%:latest
                '''
            }
        }
    }

    post {
        success {
            echo '✅ Build and push successful!'
        }
        failure {
            echo '❌ Pipeline failed!'
        }
    }
}
