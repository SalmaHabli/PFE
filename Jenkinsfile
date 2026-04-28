pipeline {
    agent {
        kubernetes {
            yaml """
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: node
    image: node:18
    command: ['cat']
    tty: true
"""
        }
    }

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
                container('node') {
                    dir('backend') {
                        sh 'node -v'
                        sh 'npm install'
                    }
                }
            }
        }

        stage('Build Frontend') {
            steps {
                container('node') {
                    dir('frontend') {
                        sh 'npm install'
                    }
                }
            }
        }

        stage('Docker Build') {
            steps {
                sh '''
                docker build -t $DOCKERHUB_USER/$IMAGE_BACKEND:$VERSION ./backend
                docker build -t $DOCKERHUB_USER/$IMAGE_FRONTEND:$VERSION ./frontend
                '''
            }
        }

        stage('Deploy') {
            steps {
                sh '''
                kubectl set image deployment/smartrh-backend \
                smartrh-backend=$DOCKERHUB_USER/$IMAGE_BACKEND:$VERSION -n smartrh || true

                kubectl set image deployment/smartrh-frontend \
                smartrh-frontend=$DOCKERHUB_USER/$IMAGE_FRONTEND:$VERSION -n smartrh || true
                '''
            }
        }
    }
}
