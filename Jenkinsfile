pipeline {
    agent any
    environment {
        // Be sure to replace "awab82002" with your own Docker Hub username
        DOCKER_IMAGE_NAME = "awab82002/maven"
    }
    stages {
        stage('Build Docker Image') {
            when {
                branch 'master'
            }
            steps {
                script {
                    def app = docker.build(DOCKER_IMAGE_NAME)
                    app.inside {
                        sh 'echo Hello, World!'
                    }
                }
            }
        }
        stage('Push Docker Image') {
            when {
                branch 'master'
            }
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'docker_hub_login') {
                        app.push("${env.BUILD_NUMBER}")
                        app.push("latest")
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
}
