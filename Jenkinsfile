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

                    cd auth-service_1784011000189
                    npm install

                    cd ../task-service
                    npm install

                    cd ../api-gateway_1784010924579
                    npm install
                '''
            }
        }

        stage('Docker Build') {
            steps {
                sh '''
                    set -e

                    docker build \
                      -f auth-service_1784011000189/Dockerfile.deploy \
                      -t smart-task-auth:latest \
                      auth-service_1784011000189

                    docker build \
                      -f task-service/Dockerfile.deploy \
                      -t smart-task-task:latest \
                      task-service

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
                    docker images | grep smart-task
                '''
            }
        }
        
        stage('Deploy Backend') {
    steps {
        sh '''
            set -e
            docker compose down
            docker compose up -d --build
            docker compose ps
        '''
    }
}
        
        stage('Frontend Build and S3 Deploy') {
            steps {
                sh '''
                    set -e

                    cd frontend

                    npm install
                    npm run build

                    aws s3 cp dist/ s3://$S3_BUCKET/ \
                      --recursive \
                      --endpoint-url $AWS_ENDPOINT_URL
                '''
            }
        }
    }

    post {
        success {
            echo '======================================'
            echo 'Pipeline Executed Successfully'
            echo 'Backend + Frontend deployed successfully'
            echo '======================================'
        }

        failure {
            echo '======================================'
            echo 'Pipeline Failed'
            echo 'Check the failed stage above'
            echo '======================================'
        }
    }
}
