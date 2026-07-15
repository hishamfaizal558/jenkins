pipeline {
    agent any

    triggers {
        githubPush()
    }

    environment {
        REMOTE_USER = "jekpro"              // Change to your VM2 username
        REMOTE_HOST = "192.168.122.243"     // Change to your VM2 IP
        APP_DIR = "/home/jekpro/jenkins"    // Path where the repo is cloned on VM2
        IMAGE_NAME = "appletlogic-website"
        CONTAINER_NAME = "appletlogic-website"
        HOST_PORT = "8080"
    }

    stages {

        stage('Checkout Source') {
            steps {
                checkout scm
            }
        }

        stage('Deploy to Docker Server') {
            steps {
                sh """
                ssh -o StrictHostKeyChecking=no ${REMOTE_USER}@${REMOTE_HOST} '
                    cd ${APP_DIR}

                    echo "Pulling latest code..."
                    git pull origin main

                    echo "Stopping old container..."
                    docker rm -f ${CONTAINER_NAME} 2>/dev/null || true

                    echo "Removing old image..."
                    docker rmi ${IMAGE_NAME}:latest 2>/dev/null || true

                    echo "Building new image..."
                    docker build --pull -t ${IMAGE_NAME}:latest .

                    echo "Starting new container..."
                    docker run -d \
                        --restart unless-stopped \
                        --name ${CONTAINER_NAME} \
                        -p ${HOST_PORT}:80 \
                        ${IMAGE_NAME}:latest

                    echo "Deployment completed."
                '
                """
            }
        }

        stage('Verify Deployment') {
            steps {
                sh """
                ssh ${REMOTE_USER}@${REMOTE_HOST} '
                    docker ps --filter name=${CONTAINER_NAME}
                '
                """
            }
        }
    }

    post {
        success {
            echo 'Deployment completed successfully.'
        }

        failure {
            echo 'Deployment failed.'
        }

        always {
            cleanWs()
        }
    }
}