pipeline {
    agent {
        kubernetes {
            yaml '''
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: node
    image: node:18
    command: ['cat']
    tty: true
'''
        }
    }

    stages {
        stage('Build') {
            steps {
                container('node') {
                    sh 'node -v'
                    sh 'npm -v'
                }
            }
        }
    }
}
