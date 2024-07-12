pipeline {
    agent any

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
            when {
                branch 'master'
            }
            steps {
                script {
                    echo 'Performing canary deployment...'
                    // Load Kubernetes configuration from secrets
                    withCredentials([file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG')]) {
                        sh '''
                            echo "Deploying to EKS with canary configuration..."
                            kubectl --kubeconfig=$KUBECONFIG apply -f train-schedule-kube-canary.yml
                        '''
                    }
                }
            }
        }

        stage('Deploy to Production') {
            when {
                branch 'master'
            }
            steps {
                script {
                    input 'Deploy to Production?'
                    milestone(1)
                    echo 'Performing production deployment...'
                    // Load Kubernetes configuration from secrets
                    withCredentials([file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG')]) {
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

    post {
        always {
            echo 'Cleaning up...'
            cleanWs()
        }
    }
}
