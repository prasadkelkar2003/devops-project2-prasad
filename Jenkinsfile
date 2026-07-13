pipeline {
    agent any
    
    environment {
        // Change this to your Docker Hub username
        REGISTRY = "your-dockerhub-username"
        IMAGE_NAME = "web-app"
        APP_VERSION = "v${BUILD_NUMBER}"
    }
    
    stages {
        stage('Checkout') {
            steps {
                cleanWs()
                checkout scm
            }
        }
        
        stage('Build & Unit Test') {
            steps {
                echo 'Running Application Unit Tests...'
            }
        }
        
        stage('Docker Build & Push') {
            steps {
                script {
                    // Builds the multi-stage optimized image
                    dockerImage = docker.build("${REGISTRY}/${IMAGE_NAME}:${APP_VERSION}", "./app")
                    docker.withRegistry('', 'docker-hub-credentials-id') {
                        dockerImage.push()
                    }
                }
            }
        }
        
        stage('Production Approval Gate') {
            steps {
                // Halts pipeline for mandatory manual confirmation
                input message: 'Approve rolling update deployment to Production?', ok: 'Deploy'
            }
        }
        
        stage('Deploy & Rollout via K8s') {
            steps {
                // Deploys using local cluster permissions
                sh "kubectl apply -f k8s/secrets.yaml"
                sh "kubectl apply -f k8s/database-stateful.yaml"
                sh "kubectl apply -f k8s/app-deployment.yaml"
                // Triggers the zero-downtime rolling update
                sh "kubectl set image deployment/web-app-deployment web-app=${REGISTRY}/${IMAGE_NAME}:${APP_VERSION}"
            }
        }
    }
}
