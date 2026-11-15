pipeline {
    agent any

    options {
        timestamps()
    }

    environment {
        IMAGE_NAME = 'activity1-cicd'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t ${IMAGE_NAME}:${BUILD_NUMBER} .'
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh 'sed "s|IMAGE_PLACEHOLDER|${IMAGE_NAME}:${BUILD_NUMBER}|" k8s/deployment.yaml > deployment-rendered.yaml'
                sh 'kubectl apply -f deployment-rendered.yaml'
                sh 'kubectl apply -f k8s/service.yaml'
            }
        }

        stage('Verify Deployment') {
            steps {
                sh 'kubectl rollout status deployment/activity1-web --timeout=120s'
                sh 'kubectl get pods -l app=activity1-web'
                sh 'kubectl get service activity1-service'
            }
        }
    }
}
