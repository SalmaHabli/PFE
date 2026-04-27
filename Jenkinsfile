pipeline {
    agent {
        docker {
            image 'node:18'
        }
    }

    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-credentials-id')
        IMAGE_BACKEND = "smartrh-backend"
        IMAGE_FRONTEND = "smartrh-frontend"
        DOCKERHUB_USER = "salma217"
        VERSION = "${BUILD_NUMBER}"
    }

    stages {

        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/SalmaHabli/PFE.git'
            }
        }

        stage('Build Backend') {
            steps {
                dir('backend') {
                    sh 'npm install'
                    sh 'npm run build || true'
                }
            }
        }

        stage('Build Frontend') {
            steps {
                dir('frontend') {
                    sh 'npm install'
                    sh 'npm run build || true'
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

        stage('Login DockerHub') {
            steps {
                sh """
                echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin
                """
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
            echo "🎉 SUCCESS"
        }
        failure {
            echo "❌ FAILED"
        }
    }
}
