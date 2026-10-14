pipeline {
    agent any

    environment {
        AWS_REGION = 'ap-south-1'
        ACCOUNT_ID = '104818751605'
        ECR_REPO = 'sample-html'

        IMAGE_URI = "${ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPO}"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                docker build -t ${ECR_REPO}:${BUILD_NUMBER} .
                '''
            }
        }

        stage('Login ECR') {
            steps {
                sh '''
                aws ecr get-login-password --region ${AWS_REGION} | \
                docker login --username AWS --password-stdin \
                ${ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com
                '''
            }
        }

        stage('Push Image') {
            steps {
                sh '''
                docker tag ${ECR_REPO}:${BUILD_NUMBER} ${IMAGE_URI}:${BUILD_NUMBER}
                docker tag ${ECR_REPO}:${BUILD_NUMBER} ${IMAGE_URI}:latest

                docker push ${IMAGE_URI}:${BUILD_NUMBER}
                docker push ${IMAGE_URI}:latest
                '''
            }
        }

        stage('Deploy') {
            steps {
                sh '''
                microk8s kubectl apply -f k8s/deployment.yaml
                microk8s kubectl apply -f k8s/service.yaml

                microk8s kubectl rollout status deployment/sample-html
                '''
            }
        }
    }
}
