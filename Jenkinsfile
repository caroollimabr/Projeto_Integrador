pipeline {

    agent none

    environment {

        NODE_ENV="homologacao"
        AWS_ACCESS_KEY=""
        AWS_SECRET_ACCESS_KEY=""
        AWS_SDK_LOAD_CONFIG="0"
        BUCKET_NAME="digitalhouse-grupolovelace-homologacao"
        REGION="us-east-1" 
        PERMISSION=""
        ACCEPTED_FILE_FORMATS_ARRAY=""
        VERSION="1.0.0"
    }


    options {
        buildDiscarder(logRotator(numToKeepStr: '5'))
    }
    triggers {
        cron('@daily')
    }

    stages{
        stage("Build, Test and Push Docker Image") {
            agent {  
                node {
                    label 'master'
                }
            }
            stages {

                stage('Clone repository') {
                    steps {
                        script {
                            if(env.GIT_BRANCH=='origin/Jenkins-pipeline'){
                                checkout scm
                            }
                            sh('printenv | sort')
                            echo "My branch is: ${env.GIT_BRANCH}"
                        }
                    }
                }
                stage('Build image'){       
                    steps {
                        script {
                            print "Environment will be : ${env.NODE_ENV}"
                            docker.build("grupolovelace:latest")
                        }
                    }
                }

                stage('Test image') {
                    steps {
                        script {
                            
                            docker.image("grupolovelace:latest").withRun('-p 8030:3000') { c ->
                                sh 'docker ps'
                                sh 'sleep 10'
                                sh 'curl http://127.0.0.1:8030/api/v1/healthcheck'
                                
                            }
                    
                        }
                    }
                }

                stage('Docker push') {
                    steps {
                        echo 'Push latest para AWS ECR'
                        script {
                            docker.withRegistry('https://185721683284.dkr.ecr.us-east-1.amazonaws.com', 'ecr:us-east-1:awscredentials') {
                                docker.image('grupolovelace').push()
                            }
                        }
                    }
                }
            }
        }

        stage('Deploy to homologacao') {
            agent {  
                node {
                    label 'homologacao'
                }
            }

            steps { 
                script {
                    if(env.GIT_BRANCH=='origin/Jenkins-pipeline'){
 
                        docker.withRegistry('https://185721683284.dkr.ecr.us-east-1.amazonaws.com', 'ecr:us-east-1:awscredentials') {
                            docker.image('grupolovelace').pull()
                        }

                        echo 'Deploy para Homologacao'
                        sh "hostname"
                        sh "docker stop grupolovelace"
                        sh "docker rm grupolovelace"
                        //sh "docker run -d --name app1 -p 8030:80 185721683284.dkr.ecr.us-east-1.amazonaws.com/grupolovelace:latest"
                         withCredentials([[$class:'AmazonWebServicesCredentialsBinding' 
                            , credentialsId 'homologacaos3']]) {
                        sh "docker run -d --name app1 -p 8030:80 -e NODE_ENV=homologacao -e AWS_ACCESS_KEY=$AWS_ACCESS_KEY_ID -e AWS_SECRET_ACCESS_KEY=$AWS_SECRET_ACCESS_KEY -e BUCKET_NAME=digitalhouse-grupolovelace-homologacao 185721683284.dkr.ecr.us-east-1.amazonaws.com/grupolovelace:latest"
                        }
                        
                        sh "docker ps"
                        sh 'sleep 10'
                        sh 'curl http://127.0.0.1:8030/api/v1/healthcheck'

                    }
                }
            }

        }

        stage('Deploy to Producao') {
            agent {  
                node {
                    label 'Producao'
                }
            }

            steps { 
                script {
                    if(env.GIT_BRANCH=='origin/Jenkins-pipeline'){
 
                        environment {

                            NODE_ENV="Producao"
                            AWS_ACCESS_KEY="123456"
                            AWS_SECRET_ACCESS_KEY="asdfghjkkll"
                            AWS_SDK_LOAD_CONFIG="0"
                            BUCKET_NAME="digitalhouse-grupolovelace-producao"
                            REGION="us-east-1" 
                            PERMISSION=""
                            ACCEPTED_FILE_FORMATS_ARRAY=""
                        }


                        docker.withRegistry('https://185721683284.dkr.ecr.us-east-1.amazonaws.com', 'ecr:us-east-1:awscredentials') {
                            docker.image('grupolovelace').pull()
                        }

                        echo 'Deploy para Produção'
                        sh "hostname"
                        sh "docker stop grupolovelace"
                        sh "docker rm grupolovelace"
                        //sh "docker run -d --name app1 -p 8030:3000 185721683284.dkr.ecr.us-east-1.amazonaws.com/grupolovelace:latest"
                         withCredentials([[$class:'AmazonWebServicesCredentialsBinding' 
                            ,  credentialsId 'producaos3']]) {
                           sh "docker run -d --name app1 -p 8030:3000 -e NODE_ENV=Producao -e AWS_ACCESS_KEY=$AWS_ACCESS_KEY_ID -e AWS_SECRET_ACCESS_KEY=$AWS_SECRET_ACCESS_KEY -e BUCKET_NAME=digitalhouse-grupolovelace-producao 185721683284.dkr.ecr.us-east-1.amazonaws.com/grupolovelace:latest"
                        }                                   
                        sh "docker ps"
                        sh 'sleep 10'
                        sh 'curl http://127.0.0.1:8030/api/v1/healthcheck'

                    }
                }
            }

        }

    }
}
