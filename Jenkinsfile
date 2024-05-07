pipeline {
    agent any
    
    parameters {
        booleanParam(name: 'run_stages', defaultValue: true, description: 'Set to true to run all stages')
        booleanParam(name: 'delete_application', defaultValue: false, description: 'Set to true to delete the application')
        string(name: 'branch_name', defaultValue: 'main', description: 'Branch name to clone')
    }
    
    environment {
        dockerhubusername = 'chrisdylan'
        container_name = 'hallo'
    }
    
    stages {
        stage('Clone Repository') {
            when {
                expression { params.run_stages }
            }
            steps {
                script {
                    git branch: "${params.branch_name}", 
                    credentialsId: 'github-dylans', 
                    url: 'https://github.com/chrisdylan237/halloween-app.git'
                }
            }
        }
        
        stage('Build Sonar Scanner Image') {
            when {
                expression { params.run_stages }
            }
            steps {
                script {
                    dir("${WORKSPACE}/sonar") {
                        sh "docker build -t ${env.dockerhubusername}/scanner:latest ."
                    }
                }
            }
        }
        
        stage('SonarQube Analysis') {
            when {
                expression { params.run_stages }
            }
            steps {
                withSonarQubeEnv('SonarScanner') {
                    script {
                        docker.image("${env.dockerhubusername}/scanner:latest").inside('-u 0:0') {
                            sh 'sonar-scanner'
                        }
                    }
                }
            }
        }
        
        stage('Build Docker Image') {
            when {
                expression { params.run_stages }
            }
            steps {
                script {
                    sh "docker build -t ${env.dockerhubusername}/halloween:${BUILD_NUMBER} ."
                }
            }
        }
        
        stage('Login to DockerHub') {
            when {
                expression { params.run_stages }
            }
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-dylan', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                    sh "docker login -u ${USERNAME} -p ${PASSWORD}"
                }
            }
        }
        
        stage('Push Docker Image to DockerHub') {
            when {
                expression { params.run_stages }
            }
            steps {
                script {
                    sh "docker push ${env.dockerhubusername}/halloween:${BUILD_NUMBER}"
                }
            }
        }
        
        stage('Deploy Docker Image') {
            when {
                expression { params.run_stages }
            }
            steps {
                script {
                    sh "docker run -itd -p 8087:80 --name ${container_name} ${env.dockerhubusername}/halloween:${BUILD_NUMBER}"
                    sh "docker ps | grep ${container_name}"
                }
            }
        }
        
        stage('Delete Application') {
            when {
                expression { params.delete_application }
            }
            steps {
                script {
                    sh "docker rm -f ${container_name}"
                    sh "docker ps | grep ${container_name}"
                }
            }
        }
    }
    
    post {
        success {
            slackSend color: '#2EB67D',
            channel: 'dev-project', 
            message: "halloween build status" +
            "\n Project Name: halloween" +
            "\n Job Name: ${env.JOB_NAME}" +
            "\n Build number: ${currentBuild.displayName}" +
            "\n Build Status : SUCCESS" +
            "\n Build url : ${env.BUILD_URL}"
        }
        failure {
            slackSend color: '#E01E5A',
            channel: 'dev-project',  
            message: "halloween build status" +
            "\n Project Name: halloween" +
            "\n Job Name: ${env.JOB_NAME}" +
            "\n Build number: ${currentBuild.displayName}" +
            "\n Build Status : FAILED" +
            "\n Build User : dylan" +
            "\n Action : Please check the console output to fix this job IMMEDIATELY" +
            "\n Build url : ${env.BUILD_URL}"
        }
        unstable {
            slackSend color: '#ECB22E',
            channel: 'del-uk', 
            message: "halloween build status" +
            "\n Project Name: halloween" +
            "\n Job Name: ${env.JOB_NAME}" +
            "\n Build number: ${currentBuild.displayName}" +
            "\n Build Status : UNSTABLE" +
            "\n Action : Please check the console output to fix this job IMMEDIATELY" +
            "\n Build url : ${env.BUILD_URL}"
        }
    }
}
