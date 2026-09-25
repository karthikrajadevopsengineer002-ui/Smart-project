pipeline {
    agent any

    stages {

        stage('Install Dependencies') {
            steps {
                sh '''
                    set -e

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
                    set -e

                    docker build \
                      -f auth-service_1784011000189/Dockerfile.deploy \
                      -t smart-task-auth:latest \
                      auth-service_1784011000189

                    docker build \
                      -f task-service/Dockerfile.deploy \
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

        stage('Load Images to Minikube') {
            steps {
                sh '''
                    set -e

                    minikube image load smart-task-auth:latest
                    minikube image load smart-task-task:latest
                '''
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh '''
                    set -e

                    echo "Checking Kubernetes..."
                    kubectl get nodes

                    echo "Creating namespace..."
                    kubectl apply -f kubernetes/namespace.yaml

                    echo "Waiting for namespace..."
                    kubectl wait \
                      --for=jsonpath='{.status.phase}'=Active \
                      namespace/smart-task \
                      --timeout=60s

                    echo "Creating secret..."
                    kubectl apply -f kubernetes/secret.yaml

                    echo "Applying remaining Kubernetes manifests..."

                    for file in kubernetes/*.yaml
                    do
                        case "$file" in
                            kubernetes/namespace.yaml)
                                ;;
                            kubernetes/secret.yaml)
                                ;;
                            *)
                                echo "Applying $file"
                                kubectl apply -f "$file"
                                ;;
                        esac
                    done
                '''
            }
        }

        stage('Verify Deployment') {
            steps {
                sh '''
                    echo "===== NODES ====="
                    kubectl get nodes

                    echo "===== NAMESPACE ====="
                    kubectl get namespace smart-task

                    echo "===== DEPLOYMENTS ====="
                    kubectl get deployments -n smart-task

                    echo "===== PODS ====="
                    kubectl get pods -n smart-task

                    echo "===== SERVICES ====="
                    kubectl get services -n smart-task

                    echo "===== INGRESS ====="
                    kubectl get ingress -n smart-task
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
stages {

    stage('Install Dependencies') {
        steps {
            // ...
        }
    }

    stage('Docker Build') {
        steps {
            // ...
        }
    }

    stage('Verify Deployment') {
        steps {
            // ...
        }
    }

    stage('Frontend Build and S3 Deploy') {
        steps {
            sh '''
                cd frontend
                npm install
                npm run build

                aws s3 cp dist/ s3://smart-task-frontend/ \
                  --recursive \
                  --endpoint-url http://localhost:4566
            '''
        }
    }
}
