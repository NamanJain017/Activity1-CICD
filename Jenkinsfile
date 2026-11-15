pipeline {
    agent any

    options {
        timestamps()
    }

    environment {
        IMAGE_REPO = 'irontiger0000007/activity1cicd'
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build and Push Docker Image') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKERHUB_USER',
                    passwordVariable: 'DOCKERHUB_TOKEN'
                )]) {
                    sh '''
                        set -eu
                        export DOCKER_CONFIG=$(mktemp -d)
                        trap 'docker logout >/dev/null 2>&1 || true; rm -rf "$DOCKER_CONFIG"' EXIT

                        docker build -t ${IMAGE_REPO}:${BUILD_NUMBER} .

                        printf '%s' "$DOCKERHUB_TOKEN" | docker login \
                            --username "$DOCKERHUB_USER" --password-stdin

                        docker push ${IMAGE_REPO}:${BUILD_NUMBER}
                    '''
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh '''
                    sed "s|IMAGE_PLACEHOLDER|${IMAGE_REPO}:${BUILD_NUMBER}|g" \
                        k8s/deployment.yaml > deployment-rendered.yaml

                    kubectl apply -f deployment-rendered.yaml
                    kubectl apply -f k8s/service.yaml
                '''
            }
        }

        stage('Verify Deployment') {
            steps {
                sh '''
                    kubectl rollout status deployment/activity1-web --timeout=180s
                    kubectl get pods -l app=activity1-web
                    kubectl get service activity1-service
                '''
            }
        }
    }
}
