pipeline {
    agent any

    stages {

        stage('Install Dependencies') {
            steps {
                sh '''
                    cd auth-service_1784011000189
                    npm install

                    cd ../task-service
                    npm install
                '''
            }
        }

        stage('Docker Build') {
            steps {
                sh '''
                    docker build -f auth-service_1784011000189/Dockerfile.deploy \
                      -t smart-task-auth:latest \
                      auth-service_1784011000189

                    docker build -f task-service/Dockerfile.deploy \
                      -t smart-task-task:latest \
                      task-service
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

        stage('Deploy to Kubernetes') {
            steps {
                sh '''
                    minikube image load smart-task-auth:latest
                    minikube image load smart-task-task:latest

                    kubectl apply -f kubernetes/
                '''
            }
        }

        stage('Verify Deployment') {
            steps {
                sh '''
                    kubectl get deployments
                    kubectl get pods
                    kubectl get services
                '''
            }
        }
    }

    post {
        success {
            echo 'Pipeline Executed Successfully'
        }

        failure {
            echo 'Pipeline Failed'
        }
    }
}



