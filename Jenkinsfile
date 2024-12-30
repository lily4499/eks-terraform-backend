pipeline {
    agent any
    environment {
         AWS_ACCESS_KEY_ID = credentials('aws-credentials') // ID of AWS Credentials
    }

    stages {
        stage('Initialize Terraform') {
            steps {
                dir('eks') { // Navigate to the eks directory
                    sh 'terraform init'
                }
            }
        }

        stage('Plan Infrastructure') {
            steps {
                dir('eks') { // Ensure Terraform commands are run from the eks directory
                    sh 'terraform plan'
                }
            }
        }

        stage('Apply Infrastructure') {
            steps {
                dir('eks') {
                    sh 'terraform apply -auto-approve'
                }
            }
        }

        stage('Deploy Application') {
            steps {
                dir('app') { // Navigate to the app directory
                    sh 'kubectl apply -f deployment.yaml'
                    sh 'kubectl apply -f service.yaml'
                }
            }
        }
    }

    post {
        always {
            echo 'Cleaning up workspace...'
            cleanWs()
        }
        failure {
            echo 'Build failed. Check logs for details.'
        }
    }
}

























