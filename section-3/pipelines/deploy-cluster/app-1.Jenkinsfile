pipeline {
    agent any
    options {
        skipStagesAfterUnstable()
    }
    stages {
         stage('Clone repository') { 
            steps { 
                script{
                checkout scm
                }
            }
        }

        stage('Build') { 
            steps { 
                script{
                 app = docker.build("app-1")
                }
            }
        }
        stage('Test'){
            steps {
                 echo 'Testing app-1'
            }
        }
        stage('Push') {
            steps {
                script{
                        docker.withRegistry('https://[ECR-Repository-id].dkr.ecr.ap-southeast-2.amazonaws.com', 'ecr:ap-southeast-1:aws-credentials') {
                    app.push("${env.BUILD_NUMBER}")
                    app.push("latest")
                    }
                }
            }
        }
        stage('Deploy'){
            steps {
                 sh 'kubectl apply -f deployment-cluster-app-1.yml'
            }
        }

    }
}