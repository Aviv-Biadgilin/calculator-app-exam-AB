pipeline {
    agent any

    environment {
        AWS_REGION = 'us-east-1'
        AWS_ACCOUNT_ID = '992382545251'
        ECR_REPO = 'calculator-app-exam-ab'
        ECR_REGISTRY = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com"
        IMAGE_NAME = "${ECR_REGISTRY}/${ECR_REPO}"
        IMAGE_TAG = "build-${BUILD_NUMBER}"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t ${ECR_REPO}:latest .'
            }
        }

        stage('Run Tests in Docker') {
            steps {
                sh 'docker run --rm ${ECR_REPO}:latest python -m unittest discover -s tests -v'
            }
        }

        stage('Login to ECR') {
            steps {
                sh 'aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${ECR_REGISTRY}'
            }
        }

        stage('Tag Image') {
            steps {
                sh 'docker tag ${ECR_REPO}:latest ${IMAGE_NAME}:${IMAGE_TAG}'
                sh 'docker tag ${ECR_REPO}:latest ${IMAGE_NAME}:latest'
            }
        }

        stage('Push Image to ECR') {
            steps {
                sh 'docker push ${IMAGE_NAME}:${IMAGE_TAG}'
                sh 'docker push ${IMAGE_NAME}:latest'
            }
        }
    }

    post {
        success {
            echo 'CI pipeline completed successfully'
        }
        failure {
            echo 'CI pipeline failed'
        }
    }
}
