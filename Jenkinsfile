pipeline {
    agent any
    
    tools {

    }
    
    stages {
        stage('GetCode') {
            steps {
                git branch: 'master', url: 'https://github.com/Muhammad-Awab/java_maven.git'
            }
        }
        stage('BuildDockerImage'){
            steps{
                sh 'docker image build -t awab82002/maven:latest'
            }
        }
         stage('Push Image'){
            steps{
                withCredentials([string(credentialsId: 'HubPWd', variable: 'HubPWd')]) {
                sh "docker login -u 82002 -p ${HubPWd}"
                }
                sh 'docker push awab82002/sonar-image:latest'
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
