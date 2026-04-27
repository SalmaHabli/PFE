pipeline {
    agent any

    environment {
        DOCKERHUB_USER = "salma217"
        IMAGE_BACKEND = "smartrh-backend"
        IMAGE_FRONTEND = "smartrh-frontend"
        VERSION = "${BUILD_NUMBER}"
    }

    stages {

        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('Build Backend') {
            steps {
                dir('backend') {
                    sh 'npm install'
                    sh 'npm run build'
                }
            }
        }

        stage('Build Frontend') {
            steps {
                dir('frontend') {
                    sh 'npm install'
                    sh 'npm run build'
                }
            }
        }

        stage('Test Backend') {
            steps {
                dir('backend') {
                    sh 'npm test || true'
                }
            }
        }

        stage('Build Docker Images') {
            steps {
                sh """
                docker build -t $DOCKERHUB_USER/$IMAGE_BACKEND:$VERSION ./backend
                docker build -t $DOCKERHUB_USER/$IMAGE_FRONTEND:$VERSION ./frontend
                """
            }
        }

        stage('Docker Login') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-credentials-id',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh '''
                    echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                    '''
                }
            }
        }

        stage('Push Images') {
            steps {
                sh """
                docker push $DOCKERHUB_USER/$IMAGE_BACKEND:$VERSION
                docker push $DOCKERHUB_USER/$IMAGE_FRONTEND:$VERSION
                """
            }
        }

        stage('Deploy Kubernetes') {
            steps {
                sh """
                kubectl apply -f k8s/
                kubectl set image deployment/backend backend=$DOCKERHUB_USER/$IMAGE_BACKEND:$VERSION || true
                kubectl set image deployment/frontend frontend=$DOCKERHUB_USER/$IMAGE_FRONTEND:$VERSION || true
                """
            }
        }
    }

    post {
        success {
            echo "🎉 SmartRH CI/CD SUCCESS"
        }
        failure {
            echo "❌ Pipeline failed"
        }
        always {
            cleanWs()
        }
    }
}
