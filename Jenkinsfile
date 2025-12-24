pipeline {
    agent any
    
    environment {
        DOCKER_REGISTRY = 'srshaikh1661'
        IMAGE_NAME = 'ci-cd-app'
        DOCKER_CREDENTIALS_ID = 'docker-hub-creds'
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        
        stage('Setup Python') {
            steps {
                script {
                    sh 'python --version'
                    sh 'pip --version'
                }
            }
        }
        
        stage('Install Dependencies') {
            steps {
                sh 'pip install -r requirements.txt'
            }
        }
        
        stage('Run Tests') {
            steps {
                sh 'pytest -v tests/ || true'
            }
            post {
                always {
                    junit '**/test-results/*.xml'
                }
            }
        }
        
        stage('Build Docker Image') {
            steps {
                script {
                    docker.build("${IMAGE_NAME}:${BUILD_NUMBER}")
                }
            }
        }
        
        stage('Test Docker Image') {
            steps {
                script {
                    docker.image("${IMAGE_NAME}:${BUILD_NUMBER}").inside('-p 5000:5000') {
                        sh '''
                            sleep 5
                            curl -f http://localhost:5000 || exit 1
                        '''
                    }
                }
            }
        }
        
        stage('Push to Docker Hub') {
            when {
                branch 'main'
            }
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', DOCKER_CREDENTIALS_ID) {
                        docker.image("${IMAGE_NAME}:${BUILD_NUMBER}").push()
                        docker.image("${IMAGE_NAME}:${BUILD_NUMBER}").push('latest')
                    }
                }
            }
        }
        
        stage('Deploy to Test Environment') {
            when {
                branch 'main'
            }
            steps {
                script {
                    sh '''
                        docker-compose down || true
                        docker-compose up -d --build
                    '''
                }
            }
        }
    }
    
    post {
        always {
            echo 'Pipeline completed. Cleaning up...'
            cleanWs()
        }
        success {
            echo 'Pipeline succeeded!'
            // Add email notification here
        }
        failure {
            echo 'Pipeline failed!'
            // Add email notification here
        }
    }
}