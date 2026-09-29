pipeline {
    agent any

    environment {
        AWS_DEFAULT_REGION = 'us-east-1'
        AWS_ACCESS_KEY_ID = 'test'
        AWS_SECRET_ACCESS_KEY = 'test'
        AWS_ENDPOINT_URL = 'http://localhost:4566'
        S3_BUCKET = 'smart-task-frontend'
    }

    stages {

        stage('Install Dependencies') {
            steps {
                sh '''
                    set -e

                    echo "===== Installing Auth Dependencies ====="
                    cd auth-service_1784011000189
                    npm install

                    echo "===== Installing Task Dependencies ====="
                    cd ../task-service
                    npm install

                    echo "===== Installing API Gateway Dependencies ====="
                    cd ../api-gateway_1784010924579
                    npm install

                    echo "===== Dependencies Installed ====="
                '''
            }
        }

        stage('Docker Build') {
            steps {
                sh '''
                    set -e

                    echo "===== Building Auth Image ====="
                    docker build \
                      -f auth-service_1784011000189/Dockerfile.deploy \
                      -t smart-task-auth:latest \
                      auth-service_1784011000189

                    echo "===== Building Task Image ====="
                    docker build \
                      -f task-service/Dockerfile.deploy \
                      -t smart-task-task:latest \
                      task-service

                    echo "===== Building API Gateway Image ====="
                    docker build \
                      -f api-gateway_1784010924579/Dockerfile.deploy \
                      -t smart-task-api-gateway:latest \
                      api-gateway_1784010924579

                    echo "===== Docker Images Built Successfully ====="
                '''
            }
        }

        stage('Verify Docker Images') {
            steps {
                sh '''
                    set -e

                    echo "===== SMART TASK DOCKER IMAGES ====="
                    docker images | grep smart-task || true

                    echo "===== DOCKER VERSION ====="
                    docker --version
                '''
            }
        }

        stage('Clean Old Containers') {
            steps {
                sh '''
                    set +e

                    echo "===== Stopping Existing Smart Task Containers ====="

                    docker compose down --remove-orphans

                    echo "===== Removing Possible Old Containers ====="

                    docker rm -f \
                        api-gateway \
                        auth-service \
                        task-service \
                        notification-service \
                        report-service \
                        mongodb \
                        frontend \
                        2>/dev/null || true

                    echo "===== Old Containers Cleaned ====="

                    docker ps -a --format "table {{.Names}}\\t{{.Status}}"
                '''
            }
        }

        stage('Deploy Backend') {
            steps {
                sh '''
                    set -e

                    echo "===== Deploying Smart Task Backend ====="

                    docker compose up -d --build --force-recreate

                    echo "===== Waiting For Containers ====="
                    sleep 15

                    echo "===== Container Status ====="
                    docker compose ps

                    echo "===== Running Containers ====="
                    docker ps --format "table {{.Names}}\\t{{.Status}}\\t{{.Ports}}"
                '''
            }
        }

        stage('Verify Backend') {
            steps {
                sh '''
                    set -e

                    echo "===== Checking API Gateway ====="

                    curl -f http://localhost:5000/health

                    echo ""
                    echo "===== API Gateway Health Check Passed ====="
                '''
            }
        }

        stage('Frontend Build and S3 Deploy') {
            steps {
                sh '''
                    set -e

                    echo "===== Building Frontend ====="

                    cd frontend

                    npm install
                    npm run build

                    echo "===== Deploying Frontend To S3 ====="

                    aws s3 cp dist/ s3://$S3_BUCKET/ \
                      --recursive \
                      --endpoint-url $AWS_ENDPOINT_URL

                    echo "===== Frontend S3 Deployment Successful ====="
                '''
            }
        }

        stage('Final Verification') {
            steps {
                sh '''
                    set -e

                    echo "======================================"
                    echo " FINAL DEPLOYMENT CHECK"
                    echo "======================================"

                    echo "===== Docker Containers ====="
                    docker compose ps

                    echo ""
                    echo "===== API Health ====="
                    curl -f http://localhost:5000/health

                    echo ""
                    echo "===== S3 Files ====="
                    aws s3 ls s3://$S3_BUCKET/ \
                      --endpoint-url $AWS_ENDPOINT_URL

                    echo ""
                    echo "======================================"
                    echo " DEPLOYMENT VERIFICATION PASSED"
                    echo "======================================"
                '''
            }
        }
    }

    post {

        success {
            echo '''
==========================================
       PIPELINE EXECUTED SUCCESSFULLY
==========================================
GitHub
   ↓
Jenkins
   ↓
Dependencies
   ↓
Docker Build
   ↓
Docker Compose Deployment
   ↓
Backend Health Check
   ↓
Frontend Build
   ↓
Floci S3 Deployment
   ↓
Final Verification

Backend + Frontend deployed successfully.
==========================================
'''
        }

        failure {
            echo '''
==========================================
          PIPELINE FAILED
==========================================
Check the failed stage above.
==========================================
'''
        }

        always {
            echo "===== Pipeline Completed ====="
        }
    }
}
