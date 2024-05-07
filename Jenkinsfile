pipeline {
    agent any

    parameters {
        string(name: 'BRANCH_NAME', defaultValue: 'master', description: 'Enter the branch name to clone')
        string(name: 'DOCKER_HUB_USERNAME', defaultValue: 'chrisdylan', description: 'Enter your Docker Hub username')
        string(name: 'DOCKER_IMAGE_NAME', defaultValue: 'sonarcli', description: 'Enter the name for the Sonar CLI Docker image')
        booleanParam(name: 'RUN_STAGE', defaultValue: true, description: 'Set to true to run the stage')
        booleanParam(name: 'DELETE_APPLICATION', defaultValue: false, description: 'Set to true to delete the application container')
        string(name: 'HOST_PORT', defaultValue: '8080', description: 'Enter the host port to bind with container port')
        string(name: 'CONTAINER_NAME', defaultValue: 'halloween_container', description: 'Enter the name for the container')
    }

    stages {
        stage('Clone Repository') {
            when {
                expression { params.RUN_STAGE }
            }
            steps {
                script {
                    git branch: "${params.BRANCH_NAME}", credentialsId: 'catcup-token', url: 'https://github.com/chrisdylan237/halloween-app.git'
                }
            }
        }
        
        stage('Build Sonar CLI Docker Image') {
            when {
                expression { params.RUN_STAGE }
            }
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
            when {
                expression { params.RUN_STAGE }
            }
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
            when {
                expression { params.RUN_STAGE }
            }
            steps {
                script {
                    sh "docker build -t ${params.DOCKER_HUB_USERNAME}/halloween:${BUILD_NUMBER} ."
                }
            }
        }
        
        stage('Login to DockerHub') {
            when {
                expression { params.RUN_STAGE }
            }
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: "dockerhub-dylan", 
                    usernameVariable: 'username', passwordVariable: 'password')]) {
                        sh "docker login -u $username -p $password"
                    }
                }
            }
        }
        
        stage('Push to DockerHub') {
            when {
                expression { params.RUN_STAGE }
            }
            steps {
                script {
                    sh "docker push ${params.DOCKER_HUB_USERNAME}/halloween:${BUILD_NUMBER}"
                }
            }
        }
        
        stage('Deploy Image') {
            when {
                expression { params.RUN_STAGE }
            }
            steps {
                script {
                    sh "docker run -itd -p ${params.HOST_PORT}:80 --name ${params.CONTAINER_NAME} ${params.DOCKER_HUB_USERNAME}/halloween:${BUILD_NUMBER}"
                }
            }
        }
        
        stage('Delete Application') {
            when {
                expression { params.DELETE_APPLICATION }
            }
            steps {
                script {
                    sh "docker rm -f ${params.CONTAINER_NAME}"
                    // Verify if the container is deleted
                    sh "docker ps | grep ${params.CONTAINER_NAME}"
                }
            }
        }
        
        // Add more stages for your pipeline as needed
    }
    // Add post-build actions if required
}
