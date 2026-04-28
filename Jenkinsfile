pipeline {
    agent any

    environment {
        DOCKERHUB_USER = "salma217"
        IMAGE_BACKEND = "smartrh-backend"
        IMAGE_FRONTEND = "smartrh-frontend"
        VERSION = "${BUILD_NUMBER}"
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main',
                url: 'https://github.com/SalmaHabli/PFE.git'
            }
        }

        stage('Build Backend') {
            steps {
                dir('backend') {
                    sh 'npm install'
                }
            }
        }

        stage('Build Frontend') {
            steps {
                dir('frontend') {
                    sh 'npm install'
                }
            }
        }

        stage('Docker Login') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-credentials-id', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
                    sh '''
                    echo $PASS | docker login -u $USER --password-stdin
                    '''
                }
            }
        }

        stage('Build Images') {
            steps {
                sh '''
                docker build -t $DOCKERHUB_USER/$IMAGE_BACKEND:$VERSION ./backend
                docker build -t $DOCKERHUB_USER/$IMAGE_FRONTEND:$VERSION ./frontend
                '''
            }
        }

        stage('Push Images') {
            steps {
                sh '''
                docker push $DOCKERHUB_USER/$IMAGE_BACKEND:$VERSION
                docker push $DOCKERHUB_USER/$IMAGE_FRONTEND:$VERSION
                '''
            }
        }

        stage('Deploy Kubernetes') {
            steps {
                sh '''
                kubectl apply -f k8s/
                kubectl set image deployment/backend backend=$DOCKERHUB_USER/$IMAGE_BACKEND:$VERSION || true
                kubectl set image deployment/frontend frontend=$DOCKERHUB_USER/$IMAGE_FRONTEND:$VERSION || true
                '''
            }
        }
    }

    post {
        success {
            echo "✅ Deployment SUCCESS"
        }
        failure {
            echo "❌ Pipeline FAILED"
        }
    }
}
