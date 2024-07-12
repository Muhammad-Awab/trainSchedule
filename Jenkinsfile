pipeline {
    agent any

    environment {
        AWS_DEFAULT_REGION = 'us-east-1' // Set your AWS region
        KUBECONFIG = '/home/ubuntu/.kube/' // Set the path to your kubeconfig file
    }

    stages {
        stage('Get Code') {
            steps {
                git 'https://github.com/Muhammad-Awab/trainSchedule.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker image build -t awab82002/train .'
            }
        }

        stage('Deploy Image') {
            steps {
                script {
                    // Use Jenkins credentials for Docker Hub login
                    withCredentials([usernamePassword(credentialsId: 'docker_hub_login', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                        sh "docker login -u $DOCKER_USERNAME -p $DOCKER_PASSWORD"

                        // Push the image
                        sh "docker push awab82002/train:latest"
                    }
                }
            }
        }

        stage('Canary Deploy') {
            steps {
                script {
                    echo 'Performing canary deployment...'
                    // Load Kubernetes configuration from secrets
                    withCredentials([file(credentialsId: 'kubecon', variable: 'KUBECONFIG')]) {
                        withAWS(credentials: 'awscred', region: 'us-east-1') {
                            sh 'env' // Print environment variables for debugging
                            sh 'cat $KUBECONFIG' // Print kubeconfig file content for debugging
                            sh 'kubectl get ns'
                        }
                    }
                }
            }
        }

        stage('Deploy to Production') {
            steps {
                script {
                    input 'Deploy to Production?'
                    milestone(1)
                    echo 'Performing production deployment...'
                    // Load Kubernetes configuration from secrets
                    withCredentials([file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG')]) {
                        withAWS(credentials: 'awscred', region: 'us-east-1') {
                            sh 'env' // Print environment variables for debugging
                            sh 'cat $KUBECONFIG' // Print kubeconfig file content for debugging
                            sh '''
                                echo "Deploying to EKS production environment..."
                                kubectl get ns
                                kubectl --kubeconfig=$KUBECONFIG apply -f train-schedule-kube-prod.yml
                            '''
                        }
                    }
                }
            }
        }
    }

    post {
        always {
            echo 'Cleaning up...'
            cleanWs()
        }
    }
}
