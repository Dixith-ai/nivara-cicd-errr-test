pipeline {
    agent any
    
    environment {
        IMAGE_NAME = 'nivara-app'
        IMAGE_TAG = "${env.BUILD_NUMBER ?: 'latest'}"
        CONTAINER_NAME = 'nivara-container'
        HOST_PORT = '80'
        CONTAINER_PORT = '3000'
    }
    
    stages {
        stage('Checkout') {
            steps {
                echo 'Checking out code...'
                checkout scm
            }
        }
        
        stage('Build Docker Image') {
            steps {
                echo "Building Docker image: ${IMAGE_NAME}:${IMAGE_TAG}"
                sh "docker build -t ${IMAGE_NAME}:${IMAGE_TAG} -f Dockerfile.dockerfile ."
                sh "docker tag ${IMAGE_NAME}:${IMAGE_TAG} ${IMAGE_NAME}:latest"
            }
        }
        
        stage('Stop Old Container') {
            steps {
                echo 'Stopping and removing old container...'
                sh """
                    docker stop ${CONTAINER_NAME} || true
                    docker rm ${CONTAINER_NAME} || true
                """
            }
        }
        
        stage('Deploy Container') {
            steps {
                echo "Deploying new container: ${CONTAINER_NAME}"
                sh """
                    docker run -d \
                        --name ${CONTAINER_NAME} \
                        -p ${HOST_PORT}:${CONTAINER_PORT} \
                        --restart unless-stopped \
                        ${IMAGE_NAME}:${IMAGE_TAG}
                """
            }
        }
        
        stage('Verify Deployment') {
            steps {
                echo 'Verifying deployment...'
                sh """
                    sleep 5
                    docker ps | grep ${CONTAINER_NAME}
                """
            }
        }
        
        stage('Cleanup Old Images') {
            steps {
                echo 'Cleaning up old Docker images...'
                sh """
                    docker image prune -f --filter "label=stage=builder"
                    docker images ${IMAGE_NAME} --format "{{.ID}} {{.Tag}}" | grep -v "${IMAGE_TAG}" | grep -v "latest" | awk '{print \$1}' | xargs -r docker rmi || true
                """
            }
        }
    }
    
    post {
        success {
            echo 'Deployment successful!'
            echo "Application is running at http://localhost:${HOST_PORT}"
        }
        failure {
            echo 'Deployment failed!'
            sh "docker logs ${CONTAINER_NAME} || true"
        }
        always {
            echo 'Pipeline execution completed.'
        }
    }
}
