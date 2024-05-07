pipeline {
    agent any
    
    environment {
        dockerhubusername = 'chrisdylan'
        container_name = 'hallo'
    }
    
    parameters {
        string(name: 'branch_name', defaultValue: 'master', description: 'Branch name to clone')
    }
    
    stages {
        stage('Clone Repository') {
            steps {
                script {
                    git branch: "${params.branch_name}", 
                    credentialsId: 'github-dylans', 
                    url: 'https://github.com/chrisdylan237/halloween-app.git'
                }
            }
        }
        
        stage('Build Sonar Scanner Image') {
            steps {
                script {
                    dir("${WORKSPACE}/sonar") {
                        sh "docker build -t ${env.dockerhubusername}/scanner:latest ."
                    }
                }
            }
        }
        
        stage('SonarQube Analysis') {
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
            steps {
                script {
                    sh "docker build -t ${env.dockerhubusername}/halloween:${BUILD_NUMBER} ."
                }
            }
        }
        
        stage('Login to DockerHub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-dylan', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                    sh "docker login -u ${USERNAME} -p ${PASSWORD}"
                }
            }
        }
        
        stage('Push Docker Image to DockerHub') {
            steps {
                script {
                    sh "docker push ${env.dockerhubusername}/halloween:${BUILD_NUMBER}"
                }
            }
        }
        
        stage('Deploy Docker Image') {
            steps {
                script {
                    sh "docker run -itd -p 8087:80 --name ${container_name} ${env.dockerhubusername}/halloween:${BUILD_NUMBER}"
                    sh "docker ps"
                }
            }
        }
    }
}
