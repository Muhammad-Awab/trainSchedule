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
                    // Add your Kubernetes deployment script here for canary deployment
                    echo 'Performing canary deployment...'
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
                    // Add your Kubernetes deployment script here for production deployment
                    echo 'Performing production deployment...'
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
