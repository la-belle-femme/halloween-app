pipeline {
    agent any
    
    environment {
        dockerhubusername = 'chrisdylan'
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
                script {
                    docker.withRegistry('https://index.docker.io/v1/', 'dockerhub-dylan') {
                        docker.login(credentialsId: 'dockerhub-dylan')
                    }
                }
            }
        }
    }
}
