pipeline {
    agent any

    stages {

        stage('Clone Source') {
            steps {
                echo 'Downloading project from GitHub...'
                checkout scm
            }
        }

        stage('Check Files') {
            steps {
                sh 'ls -la'
            }
        }

        stage('Deploy Kubernetes') {
            steps {
                sh '''
                ansible-playbook -i /root/ansible-k8s/inventory /root/ansible-k8s/playbooks/deploy-k8s.yml
                '''
            }
        }

        stage('Success') {
            steps {
                echo 'SmartRH deployed successfully!'
            }
        }
    }
}
