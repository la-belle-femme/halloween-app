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
                    credentialsId: 'github-dylan', 
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
                script {
                    docker.image("${env.dockerhubusername}/scanner:latest").inside('-u 0:0') {
                        sh 'sonar-scanner'
                    }
                }
            }
        }
    }
}
