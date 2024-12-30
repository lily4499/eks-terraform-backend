pipeline {
  agent any
  environment {
      AWS_ACCESS_KEY_ID = credentials('aws-credentials') // ID of AWS Credentials
  }

  stages {
    stage('Provision Infrastructure') {
      steps {
        sh 'terraform init'
        sh 'terraform plan'
        sh 'terraform apply -auto-approve'
      }
    }
    stage('Deploy Application') {
      steps {
        sh 'kubectl apply -f deployment.yaml'
        sh 'kubectl apply -f service.yaml'
      }
    }
  }
}
