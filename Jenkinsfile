pipeline {
    agent any

    parameters {
        string(name: 'GIT_BRANCH', defaultValue: 'master', description: 'Git branch name')
        string(name: 'DOCKERHUB_USERNAME', defaultValue: '', description: 'Docker Hub Username')
    }

    stages {
        stage('Clone Repository') {
            steps {
                script {
                    git branch: "${params.GIT_BRANCH}", credentialsId: 'github-catchup', 
                    url: 'https://github.com/chrisdylan237/halloween-app.git'
                }
            }
        }

        stage('Build SonarQube Scanner CLI Image') {
            steps {
                script {
                    dir("${WORKSPACE}/sonar") {
                        sh "docker build -t ${params.DOCKERHUB_USERNAME}/sonar-scanner-cli:${BUILD_NUMBER} ."
                    }
                }
            }
        }

        stage('SonarQube Analysis') {
            steps {
                script {
                    withSonarQube("SonarScanner") {
                        docker.image("${params.DOCKERHUB_USERNAME}/sonar-scanner-cli:${BUILD_NUMBER}").inside {
                            sh "sonar-scanner"
                        }
                    }
                }
            }
        }
    }
}
