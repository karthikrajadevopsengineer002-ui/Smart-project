pipeline {

    agent any

    environment {
        SONARQUBE = 'SonarQube'

        HARBOR_URL = 'YOUR_HARBOR_URL'
        HARBOR_PROJECT = 'smart-task'

        IMAGE_TAG = "${BUILD_NUMBER}"
        K8S_NAMESPACE = 'smart-task'
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main',
                    credentialsId: 'github-smart-task',
                    url: 'YOUR_GITHUB_REPO_URL'
            }
        }

        stage('Install Backend Dependencies') {
            steps {
                sh '''
                    set -e

                    for service in \
                    auth-service \
                    task-service \
                    notification-service \
                    report-service \
                    api-gateway
                    do
                        if [ -f "$service/package.json" ]; then
                            echo "Installing $service"
                            cd "$service"
                            npm install
                            cd ..
                        fi
                    done
                '''
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh '''
                        sonar-scanner \
                        -Dsonar.projectKey=smart-task \
                        -Dsonar.projectName=Smart-Task \
                        -Dsonar.sources=.
                    '''
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Docker Build - 5 Services') {
            steps {
                sh '''
                    set -e

                    docker build -t ${HARBOR_URL}/${HARBOR_PROJECT}/auth-service:${IMAGE_TAG} ./auth-service

                    docker build -t ${HARBOR_URL}/${HARBOR_PROJECT}/task-service:${IMAGE_TAG} ./task-service

                    docker build -t ${HARBOR_URL}/${HARBOR_PROJECT}/notification-service:${IMAGE_TAG} ./notification-service

                    docker build -t ${HARBOR_URL}/${HARBOR_PROJECT}/report-service:${IMAGE_TAG} ./report-service

                    docker build -t ${HARBOR_URL}/${HARBOR_PROJECT}/api-gateway:${IMAGE_TAG} ./api-gateway
                '''
            }
        }

        stage('Verify Docker Images') {
            steps {
                sh 'docker images'
            }
        }

        stage('Harbor Login') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'harbor-credentials',
                        usernameVariable: 'HARBOR_USER',
                        passwordVariable: 'HARBOR_PASS'
                    )
                ]) {
                    sh '''
                        echo "$HARBOR_PASS" | docker login ${HARBOR_URL} \
                        -u "$HARBOR_USER" \
                        --password-stdin
                    '''
                }
            }
        }

        stage('Push Images to Harbor') {
            steps {
                sh '''
                    docker push ${HARBOR_URL}/${HARBOR_PROJECT}/auth-service:${IMAGE_TAG}
                    docker push ${HARBOR_URL}/${HARBOR_PROJECT}/task-service:${IMAGE_TAG}
                    docker push ${HARBOR_URL}/${HARBOR_PROJECT}/notification-service:${IMAGE_TAG}
                    docker push ${HARBOR_URL}/${HARBOR_PROJECT}/report-service:${IMAGE_TAG}
                    docker push ${HARBOR_URL}/${HARBOR_PROJECT}/api-gateway:${IMAGE_TAG}
                '''
            }
        }

        stage('Kubernetes Deploy') {
            steps {
                sh '''
                    kubectl create namespace ${K8S_NAMESPACE} \
                    --dry-run=client -o yaml | kubectl apply -f -

                    kubectl apply -f k8s/ -n ${K8S_NAMESPACE}
                '''
            }
        }

        stage('Verify Kubernetes') {
            steps {
                sh '''
                    echo "===== PODS ====="
                    kubectl get pods -n ${K8S_NAMESPACE}

                    echo "===== SERVICES ====="
                    kubectl get svc -n ${K8S_NAMESPACE}

                    echo "===== DEPLOYMENTS ====="
                    kubectl get deployments -n ${K8S_NAMESPACE}
                '''
            }
        }
    }

    post {
        success {
            echo 'SMART TASK BACKEND DEPLOYMENT SUCCESS'
        }

        failure {
            echo 'BACKEND DEPLOYMENT FAILED - CHECK CONSOLE OUTPUT'
        }
    }
}
