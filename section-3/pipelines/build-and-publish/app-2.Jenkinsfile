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
                 app = docker.build("app-2")
                }
            }
        }
        stage('Test'){
            steps {
                 echo 'Testing microservice app 2'
            }
        }
        stage('Deploy') {
            steps {
                script{
                        docker.withRegistry('https://[ECR-Repository-id].dkr.ecr.ap-southeast-1.amazonaws.com', 'ecr:ap-southeast-1:aws-credentials') {
                    app.push("${env.BUILD_NUMBER}")
                    app.push("latest")
                    }
                }
            }
        }
    }
}