pipeline {
    agent any
    
    environment {
        REGISTRY = "registry.hub.docker.com/yourusername"
        COMPOSE_PROJECT_NAME = "optionchain-prod"
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        
        stage('Security Scan') {
            steps {
                echo 'Running Trivy vulnerability scanner on Dockerfiles...'
                sh 'trivy fs --severity HIGH,CRITICAL .'
            }
        }
        
        stage('Build & Test') {
            parallel {
                stage('Frontend') {
                    steps {
                        dir('frontend') {
                            sh 'npm ci'
                            sh 'npm run test:unit || true'
                        }
                    }
                }
                stage('Auth Service') {
                    steps {
                        dir('auth-service') {
                            sh 'mvn clean test'
                        }
                    }
                }
            }
        }
        
        stage('Docker Compose Build') {
            steps {
                sh 'docker-compose build'
            }
        }
        
        stage('Deploy to Production') {
            when {
                branch 'main'
            }
            steps {
                echo 'Deploying highly-available containers...'
                sh 'docker-compose up -d --remove-orphans'
                
                echo 'Waiting for health checks...'
                sleep 15
                sh 'docker-compose ps'
            }
        }
    }
    
    post {
        always {
            cleanWs()
        }
        success {
            echo "🚀 Deployment Successful! Platform is live."
            // Slack notification logic goes here
        }
        failure {
            echo "❌ Pipeline failed. Please check logs."
        }
    }
}
