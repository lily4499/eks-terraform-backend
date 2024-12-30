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
        AWS_ACCESS_KEY_ID = credentials('aws-credentials') // AWS Credentials ID
        AWS_REGION = 'us-east-1'
        CLUSTER_NAME = 'eks_cluster'
        CLUSTER_EXISTS = 'false' // Initialize to false
    }

    stages {
        stage('Initialize Terraform') {
            steps {
                dir('eks') {
                    sh 'terraform init'
                }
            }
        }

        stage('Check Existing Resources') {
            steps {
                dir('eks') {
                    script {
                        // Check if the EKS cluster already exists
                        def clusterCheck = sh(
                            script: "aws eks describe-cluster --name $CLUSTER_NAME --region $AWS_REGION > /dev/null 2>&1 && echo 'true' || echo 'false'",
                            returnStdout: true
                        ).trim()

                        if (clusterCheck == 'true') {
                            env.CLUSTER_EXISTS = 'true'
                            echo "Cluster exists: true"
                        } else {
                            env.CLUSTER_EXISTS = 'false'
                            echo "Cluster exists: false"
                        }
                    }
                }
            }
        }

        stage('Plan Infrastructure') {
            steps {
                dir('eks') {
                    script {
                        if (params.ACTION == 'apply' && env.CLUSTER_EXISTS == 'false') {
                            echo 'Cluster does not exist. Planning creation.'
                            sh 'terraform plan'
                        } else if (params.ACTION == 'apply' && env.CLUSTER_EXISTS == 'true') {
                            echo 'Cluster already exists. Skipping Terraform plan.'
                        } else if (params.ACTION == 'destroy') {
                            echo 'Planning destruction of resources.'
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
                        if (params.ACTION == 'apply' && env.CLUSTER_EXISTS == 'false') {
                            echo 'Applying Terraform to create resources.'
                            sh 'terraform apply -auto-approve'
                        } else if (params.ACTION == 'apply') {
                            echo 'Skipping Terraform apply as the cluster already exists.'
                        } else if (params.ACTION == 'destroy') {
                            dir('app') {
                                echo 'Deleting application from Kubernetes...'
                                sh 'aws eks update-kubeconfig --region $AWS_REGION --name $CLUSTER_NAME'
                                sh 'kubectl delete all --all'
                            }
                            sh 'terraform destroy -auto-approve'
                        }
                    }
                }
            }
        }

        stage('Update Kubeconfig') {
            when {
                expression { params.ACTION == 'apply' && env.CLUSTER_EXISTS == 'false' }
            }
            steps {
                echo 'Updating kubeconfig for EKS cluster...'
                sh 'aws eks update-kubeconfig --region $AWS_REGION --name $CLUSTER_NAME'
            }
        }

        stage('Deploy Application') {
            when {
                expression { params.ACTION == 'apply' }
            }
            steps {
                dir('app') {
                    echo 'Deploying application to Kubernetes...'
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

























