pipeline {
    agent any

    environment {
        AWS_DEFAULT_REGION = 'us-east-1'
        S3_BUCKET = 'smart-task-frontend'
    }

    stages {

        stage('Checkout') {
            steps {
                echo '===== CHECKOUT SOURCE CODE ====='

                checkout scm
            }
        }

        stage('Install Dependencies') {
            steps {
                sh '''
                    set -e

                    echo "===== INSTALLING BACKEND DEPENDENCIES ====="

                    cd auth-service_1784011000189
                    npm install

                    cd ../task-service
                    npm install

                    cd ../api-gateway_1784010924579
                    npm install

                    echo "===== DEPENDENCIES INSTALLED ====="
                '''
            }
        }

        stage('Docker Build') {
            steps {
                sh '''
                    set -e

                    echo "===== BUILD AUTH IMAGE ====="

                    docker build \
                      -f auth-service_1784011000189/Dockerfile.deploy \
                      -t smart-task-auth:latest \
                      auth-service_1784011000189

                    echo "===== BUILD TASK IMAGE ====="

                    docker build \
                      -f task-service/Dockerfile.deploy \
                      -t smart-task-task:latest \
                      task-service

                    echo "===== BUILD API GATEWAY IMAGE ====="

                    docker build \
                      -f api-gateway_1784010924579/Dockerfile.deploy \
                      -t smart-task-api-gateway:latest \
                      api-gateway_1784010924579
                '''
            }
        }

        stage('Verify Docker Images') {
            steps {
                sh '''
                    echo "===== DOCKER IMAGES ====="

                    docker images | grep smart-task || true
                '''
            }
        }

        stage('Clean Existing Deployment') {
            steps {
                sh '''
                    echo "===== STOPPING EXISTING COMPOSE STACK ====="

                    docker compose down --remove-orphans || true

                    echo "===== REMOVING OLD SMART-TASK CONTAINERS ====="

                    docker rm -f \
                      smart-project-auth-service \
                      smart-project-task-service \
                      smart-project-api-gateway \
                      smart-project-notification-service \
                      smart-project-report-service \
                      smart-project-mongodb \
                      smart-project-frontend \
                      auth-service \
                      task-service \
                      api-gateway \
                      notification-service \
                      report-service \
                      mongodb \
                      frontend \
                      2>/dev/null || true

                    echo "===== CLEANUP COMPLETE ====="

                    docker ps --format "table {{.Names}}\\t{{.Status}}\\t{{.Ports}}"
                '''
            }
        }

        stage('Deploy Backend') {
            steps {
                sh '''
                    set -e

                    echo "===== DEPLOYING DOCKER COMPOSE ====="

                    docker compose up -d --build --force-recreate --remove-orphans

                    echo "===== WAITING FOR CONTAINERS ====="

                    sleep 15

                    echo "===== COMPOSE STATUS ====="

                    docker compose ps
                '''
            }
        }

        stage('Verify Backend') {
            steps {
                sh '''
                    set -e

                    echo "===== RUNNING CONTAINERS ====="

                    docker ps --format "table {{.Names}}\\t{{.Status}}\\t{{.Ports}}"

                    echo "===== API HEALTH CHECK ====="

                    curl -f http://localhost:5000/health

                    echo ""

                    echo "===== BACKEND IS HEALTHY ====="
                '''
            }
        }

        stage('Frontend Build and S3 Deploy') {
            steps {
                withCredentials([
                    [$class: 'AmazonWebServicesCredentialsBinding',
                     credentialsId: 'aws-smart-task']
                ]) {
                    sh '''
                        set -e

                        echo "===== FRONTEND BUILD ====="

                        cd frontend

                        npm install
                        npm run build

                        echo "===== UPLOADING FRONTEND TO AWS S3 ====="

                        aws s3 cp dist/ s3://$S3_BUCKET/ \
                          --recursive

                        echo "===== FRONTEND DEPLOYED TO AWS S3 ====="
                    '''
                }
            }
        }

        stage('Final Verification') {
            steps {
                sh '''
                    set -e

                    echo "======================================"
                    echo " FINAL DEPLOYMENT CHECK"
                    echo "======================================"

                    echo "===== DOCKER CONTAINERS ====="

                    docker compose ps

                    echo ""
                    echo "===== API HEALTH ====="

                    curl -f http://localhost:5000/health

                    echo ""
                    echo "===== S3 BUCKET ====="

                    aws s3 ls s3://$S3_BUCKET/

                    echo ""
                    echo "===== DEPLOYMENT COMPLETED ====="
                '''
            }
        }
    }

    post {
        success {
            echo '''
========================================
     SMART TASK DEPLOYMENT SUCCESS
========================================

Backend : DEPLOYED
Frontend : AWS S3
Docker : RUNNING
API : HEALTHY
S3 : UPLOADED

========================================
'''
        }

        failure {
            echo '''
========================================
       SMART TASK DEPLOYMENT FAILED
========================================

Check the failed stage above.

========================================
'''
        }
    }
}
