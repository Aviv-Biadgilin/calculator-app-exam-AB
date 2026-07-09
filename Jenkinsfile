pipeline {
    agent any

    environment {
        AWS_REGION = 'us-east-1'
        AWS_ACCOUNT_ID = '992382545251'
        ECR_REPO = 'calculator-app-exam-ab'
        ECR_REGISTRY = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com"
        IMAGE_NAME = "${ECR_REGISTRY}/${ECR_REPO}"
        IMAGE_TAG = "${env.CHANGE_ID ? "pr-${env.CHANGE_ID}-build-${env.BUILD_NUMBER}" : "build-${env.BUILD_NUMBER}"}"
        PROD_HOST = '10.0.1.35'
        PROD_USER = 'ec2-user'
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

        stage('Deploy to Production') {
            when {
                expression { return env.CHANGE_ID == null }
            }
            steps {
                sshagent(['prod-ssh-key-ab']) {
                    sh """
                        ssh -o StrictHostKeyChecking=no ${PROD_USER}@${PROD_HOST} '
                            aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${ECR_REGISTRY} &&
                            docker pull ${IMAGE_NAME}:latest &&
                            docker stop calculator-app || true &&
                            docker rm calculator-app || true &&
                            docker run -d --name calculator-app -p 5000:5000 ${IMAGE_NAME}:latest &&
                            sleep 5 &&
                            curl -f http://localhost:5000/health
                        '
                    """
                }
            }
        }
    }

    post {
        success {
            echo 'CI/CD pipeline completed successfully'
        }
        failure {
            echo 'CI/CD pipeline failed'
        }
    }
}
