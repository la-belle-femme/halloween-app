pipeline {
    agent any

    parameters {
        string(name: 'BRANCH_NAME', defaultValue: 'master', description: 'Enter the branch name to clone')
        string(name: 'DOCKER_HUB_USERNAME', defaultValue: 'chrisdylan', description: 'Enter your Docker Hub username')
        string(name: 'DOCKER_IMAGE_NAME', defaultValue: 'sonarcli', description: 'Enter the name for the Sonar CLI Docker image')
        
    }

    stages {
        stage('Clone Repository') {
            steps {
                script {
                    git branch: "${params.BRANCH_NAME}", credentialsId: 'catcup-token', url: 'https://github.com/chrisdylan237/halloween-app.git'
                }
            }
        }
        
        stage('Build Sonar CLI Docker Image') {
            steps {
                script {
                    dir("${workspace}/sonar") {
                        sh "docker build -t ${params.DOCKER_HUB_USERNAME}/${params.DOCKER_IMAGE_NAME} ."
                        sh "docker images | grep ${params.DOCKER_IMAGE_NAME}"
                    }
                }
            }
        }
        
        stage('Run Sonar Analysis with Docker') {
            steps {
                script {
                    withSonarQubeEnv('SonarScanner') {
                        docker.image("${params.DOCKER_HUB_USERNAME}/${params.DOCKER_IMAGE_NAME}").inside {
                            sh 'sonar-scanner'
                        }
                    }
                }
            }
        }
        
        stage('Build Application') {
            steps {
                script {
                    sh "docker build -t ${params.DOCKER_HUB_USERNAME}/halloween:${BUILD_NUMBER} ."
                }
            }
        }
        
        stage('Login to DockerHub') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: "dockerhub-dylan", 
                    usernameVariable: 'username', passwordVariable: 'password')]) {
                        sh "docker login -u $username -p $password"
                    }
                }
            }
        }
        
        // Add more stages for your pipeline as needed
    }
    // Add post-build actions if required
}
