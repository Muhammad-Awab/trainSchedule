pipeline {
    agent any
    environment {
        DOCKER_IMAGE_NAME = "awab82002/maven"
        DOCKER_REGISTRY = "https://registry.hub.docker.com"
    }
    stages {
        stage('Checkout') {
            steps {
                git 'https://github.com/Muhammad-Awab/trainSchedule.git'
            }
        }
        stage('Build Docker Image') {
            when {
                branch 'master'
            }
            steps {
                script {
                    def app = docker.build("${DOCKER_IMAGE_NAME}:${env.BUILD_NUMBER}")
                }
            }
        }
        stage('Push Docker Image') {
            when {
                branch 'master'
            }
            steps {
                script {
                    docker.withRegistry(DOCKER_REGISTRY, 'docker_hub_login') {
                        docker.image("${DOCKER_IMAGE_NAME}:${env.BUILD_NUMBER}").push()
                        docker.image("${DOCKER_IMAGE_NAME}:latest").push()
                    }
                }
            }
        }
        stage('CanaryDeploy') {
            when {
                branch 'master'
            }
            environment {
                CANARY_REPLICAS = 1
            }
            steps {
                script {
                    kubernetesDeploy(
                        kubeconfigId: 'kubeconfig',
                        configs: 'train-schedule-kube-canary.yml',
                        enableConfigSubstitution: true
                    )
                }
            }
        }
        stage('DeployToProduction') {
            when {
                branch 'master'
            }
            environment {
                CANARY_REPLICAS = 0
            }
            steps {
                script {
                    input 'Deploy to Production?'
                    milestone(1)
                    kubernetesDeploy(
                        kubeconfigId: 'kubeconfig',
                        configs: 'train-schedule-kube-canary.yml',
                        enableConfigSubstitution: true
                    )
                    kubernetesDeploy(
                        kubeconfigId: 'kubeconfig',
                        configs: 'train-schedule-kube.yml',
                        enableConfigSubstitution: true
                    )
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
