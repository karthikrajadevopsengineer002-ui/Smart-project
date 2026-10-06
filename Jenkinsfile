pipeline {
    agent any

    environment {
        AWS_CREDENTIALS= credentials('aws-id')
        AWS_REGION= 'us-east-1'
        DOCKER_HUB_USER = 'giriprasathp'
        DOCKER_TAG = "v${BUILD_NUMBER}"
        EC2_HOST = '34.200.219.181'
        //PEM_KEY_PATH= '/home/ubuntu/keys/backend.pem'
        EC2_USER = 'ubuntu'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Backend Images') {
            steps {
                script {
                    echo "Building Microservices Docker Image"
                    sh '''
                    docker build -t $DOCKER_HUB_USER/smarttask-api-gateway:$DOCKER_TAG ./api-gateway_1784010924579
                    docker build -t $DOCKER_HUB_USER/smarttask-auth-service:$DOCKER_TAG ./auth-service_1784011000189
                    docker build -t $DOCKER_HUB_USER/smarttask-task-service:$DOCKER_TAG ./task-service
                    docker build -t $DOCKER_HUB_USER/smarttask-notification-service:$DOCKER_TAG ./notification-service
                    docker build -t $DOCKER_HUB_USER/smarttask-report-service:$DOCKER_TAG ./report-service
                    
                    docker tag $DOCKER_HUB_USER/smarttask-api-gateway:$DOCKER_TAG $DOCKER_HUB_USER/smarttask-api-gateway:latest
                    docker tag $DOCKER_HUB_USER/smarttask-auth-service:$DOCKER_TAG $DOCKER_HUB_USER/smarttask-auth-service:latest
                    docker tag $DOCKER_HUB_USER/smarttask-task-service:$DOCKER_TAG $DOCKER_HUB_USER/smarttask-task-service:latest
                    docker tag $DOCKER_HUB_USER/smarttask-notification-service:$DOCKER_TAG $DOCKER_HUB_USER/smarttask-notification-service:latest
                    
                    docker tag $DOCKER_HUB_USER/smarttask-report-service:$DOCKER_TAG $DOCKER_HUB_USER/smarttask-report-service:latest
                    '''
                }
            }
        }

        stage('Push to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh '''
                    echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                    
                    docker push $DOCKER_HUB_USER/smarttask-api-gateway:$DOCKER_TAG
                    docker push $DOCKER_HUB_USER/smarttask-auth-service:$DOCKER_TAG
                    docker push $DOCKER_HUB_USER/smarttask-task-service:$DOCKER_TAG
                    docker push $DOCKER_HUB_USER/smarttask-notification-service:$DOCKER_TAG
                    docker push $DOCKER_HUB_USER/smarttask-report-service:$DOCKER_TAG
                    
                    docker push $DOCKER_HUB_USER/smarttask-api-gateway:latest
                    docker push $DOCKER_HUB_USER/smarttask-auth-service:latest
                    docker push $DOCKER_HUB_USER/smarttask-task-service:latest
                    docker push $DOCKER_HUB_USER/smarttask-notification-service:latest
                    docker push $DOCKER_HUB_USER/smarttask-report-service:latest
                    '''
                }
            }
        }

        stage('Deploy to EC2') {
          steps {
                sshagent(['ec2-ssh-key']) {
                  sh '''
                    ssh -o StrictHostKeyChecking=no $EC2_USER@$EC2_HOST "
                      cd devops-flow
                      git pull origin main
                      docker compose pull
                      docker compose down
                      docker compose up -d --remove-orphans
                      docker ps
                    "
                  '''
                    
                }
            }

            }
    

    }
}
 

