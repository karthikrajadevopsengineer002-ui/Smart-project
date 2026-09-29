pipeline {
    agent any

    environment {
        AWS_DEFAULT_REGION = 'us-east-1'
        S3_BUCKET = 'smart-task-frontend'

        // உன் actual CloudFront Distribution ID இங்கே
        CLOUDFRONT_ID = 'YOUR_CLOUDFRONT_ID'
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

                cd ../notification-service
                npm install

                cd ../report-service
                npm install

                cd ../api-gateway_1784010924579
                npm install

                echo "===== DEPENDENCIES INSTALLED ====="
                '''
            }
        }

        stage('Docker Build - 5 Services') {
            steps {
                sh '''
                set -e

                echo "===== BUILD AUTH ====="
                docker build \
                  -f auth-service_1784011000189/Dockerfile.deploy \
                  -t smart-task-auth:latest \
                  auth-service_1784011000189

                echo "===== BUILD TASK ====="
                docker build \
                  -f task-service/Dockerfile.deploy \
                  -t smart-task-task:latest \
                  task-service

                echo "===== BUILD NOTIFICATION ====="
                docker build \
                  -f notification-service/Dockerfile \
                  -t smart-task-notification:latest \
                  notification-service

                echo "===== BUILD REPORT ====="
                docker build \
                  -f report-service/Dockerfile \
                  -t smart-task-report:latest \
                  report-service

                echo "===== BUILD API GATEWAY ====="
                docker build \
                  -f api-gateway_1784010924579/Dockerfile.deploy \
                  -t smart-task-api-gateway:latest \
                  api-gateway_1784010924579

                echo "===== ALL 5 SERVICES BUILT ====="
                '''
            }
        }

        stage('Verify Docker Images') {
            steps {
                sh '''
                echo "===== DOCKER IMAGES ====="
                docker images | grep smart-task
                '''
            }
        }

        stage('Clean Existing Deployment') {
            steps {
                sh '''
                echo "===== STOPPING EXISTING STACK ====="

                docker compose down --remove-orphans || true

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
                  frontend 2>/dev/null || true

                echo "===== CLEANUP COMPLETE ====="
                '''
            }
        }

        stage('Deploy 5 Backend Services') {
            steps {
                sh '''
                set -e

                echo "===== DEPLOYING 5 BACKEND SERVICES ====="

                docker compose up -d --build --force-recreate --remove-orphans

                echo "===== CONTAINERS ====="
                docker ps --format "table {{.Names}}\\t{{.Status}}\\t{{.Ports}}"
                '''
            }
        }

        stage('Backend Health Check') {
            steps {
                sh '''
                set -e

                echo "===== API HEALTH CHECK ====="

                sleep 10

                curl -f http://localhost:5000/health

                echo ""
                echo "===== PORT CHECK ====="

                for PORT in 5000 5001 5002 5003 5004
                do
                    if nc -z localhost $PORT; then
                        echo "PORT $PORT : RUNNING"
                    else
                        echo "PORT $PORT : FAILED"
                        exit 1
                    fi
                done

                echo "===== ALL 5 SERVICES HEALTHY ====="
                '''
            }
        }

        stage('Build Frontend') {
            steps {
                sh '''
                set -e

                echo "===== BUILD FRONTEND ====="

                cd frontend

                npm install
                npm run build

                echo "===== FRONTEND BUILD COMPLETE ====="
                ls -la dist/
                '''
            }
        }

        stage('Deploy Frontend to S3') {
            steps {
                sh '''
                set -e

                echo "===== UPLOAD FRONTEND TO S3 ====="

                aws s3 sync frontend/dist/ \
                  s3://$S3_BUCKET/ \
                  --delete

                echo "===== FRONTEND UPLOADED TO S3 ====="
                '''
            }
        }

        stage('CloudFront Invalidation') {
            steps {
                sh '''
                set -e

                echo "===== CLOUDFRONT INVALIDATION ====="

                if [ "$CLOUDFRONT_ID" != "YOUR_CLOUDFRONT_ID" ]; then

                    aws cloudfront create-invalidation \
                      --distribution-id "$CLOUDFRONT_ID" \
                      --paths "/*"

                    echo "===== CLOUDFRONT CACHE INVALIDATED ====="

                else
                    echo "CloudFront ID not configured - skipping invalidation"
                fi
                '''
            }
        }

        stage('Final Status') {
            steps {
                sh '''
                echo ""
                echo "======================================"
                echo " SMART TASK DEPLOYMENT SUCCESS"
                echo "======================================"
                echo "GitHub : CHECKED OUT"
                echo "Auth : DEPLOYED - 5001"
                echo "Task : DEPLOYED - 5002"
                echo "Notification : DEPLOYED - 5003"
                echo "Report : DEPLOYED - 5004"
                echo "API Gateway : DEPLOYED - 5000"
                echo "Frontend : S3"
                echo "CloudFront : ENABLED"
                echo "======================================"
                '''
            }
        }
    }
}
