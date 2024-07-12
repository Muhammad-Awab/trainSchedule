pipeline {
    agent any

    environment {
        AWS_DEFAULT_REGION = 'us-east-1' // Set your AWS region
        KUBECONFIG = '/home/ubuntu/.kube/config' // Set the path to your kubeconfig file
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
                    withCredentials([usernamePassword(credentialsId: 'docker_hub_login', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                        sh "docker login -u $DOCKER_USERNAME -p $DOCKER_PASSWORD"
                        sh "docker push awab82002/train:latest"
                    }
                }
            }
        }

        stage('Canary Deploy') {
            steps {
                script {
                    echo 'Performing canary deployment...'
                    withCredentials([file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG_FILE')]) {
                        sh 'cp $KUBECONFIG_FILE $KUBECONFIG'
                        withAWS(credentials: 'awscred', region: 'us-east-1') {
                            sh 'aws eks update-kubeconfig --region us-east-1 --name web-quickstart'
                            sh 'kubectl get ns'
                            sh 'kubectl apply -f train-schedule-kube-canary.yml'
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
