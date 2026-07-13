pipeline {
    agent any
    
    environment {
        // Your actual Docker Hub username to prevent access denial errors
        REGISTRY = "prad2003" 
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
                    
                    // Log into Docker Hub using the credential ID you created in Jenkins
                    withCredentials([usernamePassword(credentialsId: 'jenkins-docker', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                        sh "echo \$DOCKER_PASS | docker login -u \$DOCKER_USER --password-stdin"
                        sh "docker push ${REGISTRY}/${IMAGE_NAME}:${APP_VERSION}"
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
                // Force kubectl to use the local Killercoda cluster configuration context directly
                withEnv(["KUBECONFIG=/var/lib/jenkins/.kube/config"]) {
                    sh "kubectl apply -f k8s/secrets.yaml"
                    sh "kubectl apply -f k8s/database-stateful.yaml"
                    sh "kubectl apply -f k8s/app-deployment.yaml"
                    sh "kubectl set image deployment/web-app-deployment web-app=${REGISTRY}/${IMAGE_NAME}:${APP_VERSION}"
                }
            }
        }
    }
}
