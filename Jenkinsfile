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
                    // Clone the repository using credentials and the specified branch
                    git branch: "${params.BRANCH_NAME}", credentialsId: 'catcup-token', url: 'https://github.com/chrisdylan237/halloween-app.git'
                }
            }
        }
        
        stage('Build Sonar CLI Docker Image') {
            steps {
                script {
                    // Change directory to the Sonar directory
                    dir("${workspace}/sonar") {
                        // Run docker build command to build the Sonar CLI Docker image
                        sh "docker build -t ${params.DOCKER_HUB_USERNAME}/${params.DOCKER_IMAGE_NAME} ."
                        sh "docker images | grep ${params.DOCKER_IMAGE_NAME}"
                    }
                }
            }
        }
        stage('Run Sonar Analysis with Docker') {
            steps{
                withSonarQubeEnv('SonarScanner'){
                    script{
                        docker.image("${params.DOCKER_HUB_USERNAME}/${params.DOCKER_IMAGE_NAME}")
                        sh 'sonar-scanner'
                    }
                }
            }
        // Add more stages for your pipeline as needed
    }
    // Add post-build actions if required
}
