pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t my-app .'
            }
        }

        stage('Deploy Container') {
            steps {
                sh '''
                    docker stop my-container || true
                    docker rm my-container || true
                    docker run -d --name my-container -p 80:3000 my-app
                '''
            }
        }

        stage('Cleanup') {
            steps {
                sh 'docker image prune -f'
            }
        }
    }
}