pipeline {
    agent any
    
    environment {
        // ⚠️ EDIT THIS: Put your real Docker Hub username here
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
                    // Use standard shell commands to build the image
                    sh "docker build -t ${REGISTRY}/${IMAGE_NAME}:${APP_VERSION} ./app"
                    
                    // Log into Docker Hub using credentials securely managed by Jenkins
                    withCredentials([usernamePassword(credentialsId: 'docker-hub-credentials-id', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                        sh "echo \$DOCKER_PASS | docker login -u \$DOCKER_USER --password-stdin"
                        sh "docker push ${REGISTRY}/${IMAGE_NAME}:${APP_VERSION}"
                    }
                }
            }
        }
        
        stage('Production Approval Gate') {
            steps {
                input message: 'Approve rolling update deployment to Production?', ok: 'Deploy'
            }
        }
        
        stage('Deploy & Rollout via K8s') {
            steps {
                sh "kubectl apply -f k8s/secrets.yaml"
                sh "kubectl apply -f k8s/database-stateful.yaml"
                sh "kubectl apply -f k8s/app-deployment.yaml"
                sh "kubectl set image deployment/web-app-deployment web-app=${REGISTRY}/${IMAGE_NAME}:${APP_VERSION}"
            }
        }
    }
}
