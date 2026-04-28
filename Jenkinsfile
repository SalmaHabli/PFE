pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-credentials-id')
        IMAGE_BACKEND = "smartrh-backend"
        IMAGE_FRONTEND = "smartrh-frontend"
        DOCKERHUB_USER = "salma217"
        VERSION = "${BUILD_NUMBER}"
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/SalmaHabli/PFE.git'
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

        stage('Docker Build & Push') {
            steps {
                sh '''
                docker build -t salma217/smartrh-backend:${BUILD_NUMBER} ./backend
                docker build -t salma217/smartrh-frontend:${BUILD_NUMBER} ./frontend

                echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin

                docker push salma217/smartrh-backend:${BUILD_NUMBER}
                docker push salma217/smartrh-frontend:${BUILD_NUMBER}
                '''
            }
        }

        stage('Deploy K8s') {
            steps {
                sh '''
                kubectl apply -f k8s/
                '''
            }
        }
    }
}
