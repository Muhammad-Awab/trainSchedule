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
                    kubeconfig(caCertificate: 'LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSURCVENDQWUyZ0F3SUJBZ0lJSzJUejgyMWV6Ujh3RFFZSktvWklodmNOQVFFTEJRQXdGVEVUTUJFR0ExVUUKQXhNS2EzVmlaWEp1WlhSbGN6QWVGdzB5TkRBM01USXhNelE1TkRGYUZ3MHpOREEzTVRBeE16VTBOREZhTUJVeApFekFSQmdOVkJBTVRDbXQxWW1WeWJtVjBaWE13Z2dFaU1BMEdDU3FHU0liM0RRRUJBUVVBQTRJQkR3QXdnZ0VLCkFvSUJBUURGVHA5MHE3YVNGRXN0UklObm9QbGg3TGNjU014eTRnTGZkTC9sYlZsVkczVnZWOW91NHFTYnRBdWQKY05SY05QRjdBSEJ3QUx5R2dIOW8xUjRSRGpjTFRjaVh4ZjByREZERzZUdlZsQmR6QUVkaWVobDFCbGZPQnhXRQphcktDb1lQVm1DOFJzdTVWdm1WSEtEZkUvLzMzUGgyNnltTTJHSTdnSU9Ua3gxTW1YMmp2UmpySE5oSHJZVEdlCmhDcUdsWFg1RHUva2NsNnRYV2tpc29xMHFRSFNvcnE2TncxdFdMSmdCejZxTWVMUU9ibmFHSzlKdGtMQzdGTkUKS3NZRU1GWExGK0VTbkZBY1d2ZWhQMEFmQ3Q2Q054bURVVmN6RDZpcWRBRDNKbENMdnlyZ0c0aFgxQlA5Y0cxQQorWGVqa3NBWmpmek0xME9nTmVqaGdFTzBnVC9QQWdNQkFBR2pXVEJYTUE0R0ExVWREd0VCL3dRRUF3SUNwREFQCkJnTlZIUk1CQWY4RUJUQURBUUgvTUIwR0ExVWREZ1FXQkJUS1p6YzBHTVQ3VHpEZUYyUFNtZGxoZlBuclVqQVYKQmdOVkhSRUVEakFNZ2dwcmRXSmxjbTVsZEdWek1BMEdDU3FHU0liM0RRRUJDd1VBQTRJQkFRQjZXdi9sZURrRgo0Smd4eGpqeUZaTkd1T3Vxc0NyQ29ZMXIycmplVTIrNXJxUG1SK3pJV3l3eEZWS3ZMM0pIRGlKdE9YZGJVVy9jCnpMeStzalE3Rnl4OVl0M3lzZXI5TGJzTU56M2JnMXVlTmRVTmJwcENCeVhPRWtZVnhsSFF0ZEY0aW1vQUU5R2UKa0JsVG02V0lHS043ZnNvTEZtbUtXY2d2NC96L2oxMVRxUzRHVXE1elVKc3ZlVlI4ZmN0QkZKRFdtaTN2Y0xFcgpDS1RmNERReDBSQkswRTdvSER0cVUwSGk0UkxnOHJqMTZpcXJoaWRwUE9XRmwxNHRLcTB5aGVoTGRJYXo4NE9wCkdKVEtHZUYvV3hkcWt6Q3pQRC9UVEdlWk1mT0xlVHU4bXQ0TG51dUp0aGVSK2R0NUtWZXZIdkFkWC9pVmU0R3kKbzl3c0JCOVpkQkxICi0tLS0tRU5EIENFUlRJRklDQVRFLS0tLS0K', credentialsId: 'kubecon', serverUrl: 'https://68E537C3125ADA37F25ABEE5B5831978.yl4.us-east-1.eks.amazonaws.com') {
                           sh 'kubectl get ns'
                    }
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
