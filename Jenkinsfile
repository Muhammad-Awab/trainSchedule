pipeline {
    agent any

    stages {
        stage('GetCode') {
            steps {
                git 'https://github.com/Muhammad-Awab/trainSchedule.git'
            }
        }
        
        stage('BuildDockerImage'){
            steps{
                sh 'docker image build -t awab82002/maven:latest .'
            }
        }
        
        stage('Push Image'){
            
                withDockerRegistry(credentialsId: 'docker_hub_login', url: ' https://index.docker.io/v2/') {
                   sh """
                   docker push awab82002/maven:latest
                   """
                 }
               
            
            }
        
        
        stage('CanaryDeploy') {
            when {
                branch 'master'
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
            steps {
                script {
                    input 'Deploy to Production?'
                    milestone(1)
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
