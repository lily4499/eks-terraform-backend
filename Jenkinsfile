pipeline {
    agent any
    parameters {
        choice(
            name: 'ACTION',
            choices: ['apply', 'destroy'],
            description: 'Choose whether to apply or destroy the infrastructure'
        )
    }
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
                dir('eks') {
                    script {
                        if (params.ACTION == 'apply') {
                            sh 'terraform plan'
                        } else if (params.ACTION == 'destroy') {
                            sh 'terraform plan -destroy'
                        }
                    }
                }
            }
        }

        stage('Apply or Destroy Infrastructure') {
            steps {
                dir('eks') {
                    script {
                        if (params.ACTION == 'apply') {
                            sh 'terraform apply -auto-approve'
                        } else if (params.ACTION == 'destroy') {
                            dir('app') {
                                echo 'Deleting application from Kubernetes...'
                                sh 'kubectl delete all --all'
                            }
                            sh 'terraform destroy -auto-approve'
                        }
                    }
                }
            }
        }

        stage('Deploy Application') {
            when {
                expression { params.ACTION == 'apply' } // Only deploy when the action is 'apply'
            }
            steps {
                dir('app') {
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





















// pipeline {
//     agent any
//     environment {
//          AWS_ACCESS_KEY_ID = credentials('aws-credentials') // ID of AWS Credentials
//     }

//     stages {
//         stage('Initialize Terraform') {
//             steps {
//                 dir('eks') { // Navigate to the eks directory
//                     sh 'terraform init'
//                 }
//             }
//         }

//         stage('Plan Infrastructure') {
//             steps {
//                 dir('eks') { // Ensure Terraform commands are run from the eks directory
//                     sh 'terraform plan'
//                 }
//             }
//         }

//         stage('Apply Infrastructure') {
//             steps {
//                 dir('eks') {
//                     sh 'terraform apply -auto-approve'
//                 }
//             }
//         }

//         stage('Deploy Application') {
//             steps {
//                 dir('app') { // Navigate to the app directory
//                     sh 'kubectl apply -f deployment.yaml'
//                     sh 'kubectl apply -f service.yaml'
//                 }
//             }
//         }
//     }

//     post {
//         always {
//             echo 'Cleaning up workspace...'
//             cleanWs()
//         }
//         failure {
//             echo 'Build failed. Check logs for details.'
//         }
//     }
// }

























