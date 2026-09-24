pipeline {
    agent any

    environment {
        HARBOR_URL = '13.49.183.17'
        HARBOR_PROJECT = 'smart-task-management-system'
        IMAGE_TAG = 'latest'
    }

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

        stage('Verify Images') {
            steps {
                sh 'docker images | grep smart-task'
            }
        }

        stage('Push Images to Harbor') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'harbor-credentials',
                        usernameVariable: 'HARBOR_USER',
                        passwordVariable: 'HARBOR_PASSWORD'
                    )
                ]) {
                    sh '''
                        echo "$HARBOR_PASSWORD" | docker login "$HARBOR_URL" \
                          -u "$HARBOR_USER" --password-stdin

                        docker tag smart-task-auth:latest \
                          $HARBOR_URL/$HARBOR_PROJECT/smart-task-auth:$IMAGE_TAG

                        docker tag smart-task-task:latest \
                          $HARBOR_URL/$HARBOR_PROJECT/smart-task-task:$IMAGE_TAG

                        docker push \
                          $HARBOR_URL/$HARBOR_PROJECT/smart-task-auth:$IMAGE_TAG

                        docker push \
                          $HARBOR_URL/$HARBOR_PROJECT/smart-task-task:$IMAGE_TAG
                    '''
                }
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

