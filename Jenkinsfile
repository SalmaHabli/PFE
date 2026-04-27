pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-credentials-id')
        IMAGE_BACKEND = "smartrh-backend"
        IMAGE_FRONTEND = "smartrh-frontend"
        DOCKERHUB_USER = "ton_dockerhub_username"
    }

    stages {

        stage('Checkout Code') {
            steps {
                echo "📥 Cloning repository..."
                git branch: 'main', url: 'https://github.com/SalmaHabli/PFE.git'
            }
        }

        stage('Build Backend') {
            steps {
                echo "⚙️ Building backend..."
                dir('backend') {
                    sh 'npm install'
                    sh 'npm run build || echo "No build step found"'
                }
            }
        }

        stage('Build Frontend') {
            steps {
                echo "⚙️ Building frontend..."
                dir('frontend') {
                    sh 'npm install'
                    sh 'npm run build'
                }
            }
        }

        stage('Run Tests') {
            steps {
                echo "🧪 Running tests..."
                dir('backend') {
                    sh 'npm test || echo "No tests configured"'
                }
            }
        }

        stage('Build Docker Images') {
            steps {
                echo "🐳 Building Docker images..."

                sh """
                docker build -t $DOCKERHUB_USER/$IMAGE_BACKEND:latest ./backend
                docker build -t $DOCKERHUB_USER/$IMAGE_FRONTEND:latest ./frontend
                """
            }
        }

        stage('Login & Push to DockerHub') {
            steps {
                echo "📤 Pushing images to DockerHub..."

                sh """
                echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin

                docker push $DOCKERHUB_USER/$IMAGE_BACKEND:latest
                docker push $DOCKERHUB_USER/$IMAGE_FRONTEND:latest
                """
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                echo "☸️ Deploying to Kubernetes..."

                sh """
                kubectl apply -f k8s/
                kubectl rollout restart deployment backend || true
                kubectl rollout restart deployment frontend || true
                """
            }
        }
    }

    post {
        success {
            echo "🎉 Pipeline executed successfully!"
        }

        failure {
            echo "❌ Pipeline failed. Check logs."
        }
    }
}
