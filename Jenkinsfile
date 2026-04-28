pipeline {
    agent any

    environment {
        DOCKERHUB_USER = "salma217"
        IMAGE_BACKEND = "smartrh-backend"
        IMAGE_FRONTEND = "smartrh-frontend"
        VERSION = "${BUILD_NUMBER}"
        K8S_NAMESPACE = "smartrh"
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

        stage('Docker Build') {
            steps {
                sh '''
                docker build -t $DOCKERHUB_USER/$IMAGE_BACKEND:$VERSION ./backend
                docker build -t $DOCKERHUB_USER/$IMAGE_FRONTEND:$VERSION ./frontend
                '''
            }
        }

        stage('Docker Push') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-credentials-id', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
                    sh '''
                    echo $PASS | docker login -u $USER --password-stdin

                    docker push $DOCKERHUB_USER/$IMAGE_BACKEND:$VERSION
                    docker push $DOCKERHUB_USER/$IMAGE_FRONTEND:$VERSION
                    '''
                }
            }
        }

        stage('Update Kubernetes (Rolling Update)') {
            steps {
                sh '''
                kubectl set image deployment/smartrh-backend \
                smartrh-backend=$DOCKERHUB_USER/$IMAGE_BACKEND:$VERSION \
                -n $K8S_NAMESPACE

                kubectl set image deployment/smartrh-frontend \
                smartrh-frontend=$DOCKERHUB_USER/$IMAGE_FRONTEND:$VERSION \
                -n $K8S_NAMESPACE
                '''
            }
        }

        stage('Verify') {
            steps {
                sh '''
                kubectl get pods -n smartrh
                kubectl rollout status deployment/smartrh-backend -n smartrh
                kubectl rollout status deployment/smartrh-frontend -n smartrh
                '''
            }
        }
    }

    post {
        success {
            echo "✅ SmartRH deployed successfully"
        }
        failure {
            echo "❌ Deployment failed"
        }
    }
}
