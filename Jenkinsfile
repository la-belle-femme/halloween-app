pipeline {
    agent any

    parameters {
        string(name: 'GIT_BRANCH', defaultValue: 'master', description: 'Git branch name')
        string(name: 'DOCKERHUB_USERNAME', defaultValue: '', description: 'Docker Hub Username')
        string(name: 'HOST_PORT', defaultValue: '8080', description: 'Host Port')
        string(name: 'CONTAINER_NAME', defaultValue: 'my-container', description: 'Container Name')
        booleanParam(name: 'RUN_STAGES', defaultValue: false, description: 'Run stages')
        booleanParam(name: 'DELETE_APPLICATION', defaultValue: false, description: 'Delete Application')
        booleanParam(name: 'SKIP_STAGES', defaultValue: false, description: 'Skip stages')
    }

    stages {
        stage('Clone Repository') {
            when {
                expression { params.RUN_STAGES }
            }
            steps {
                script {
                    git branch: "${params.GIT_BRANCH}", credentialsId: 'github-catchup', 
                    url: 'https://github.com/chrisdylan237/halloween-app.git'
                }
            }
        }

        stage('Build SonarQube Scanner CLI Image') {
            when {
                expression { params.RUN_STAGES }
            }
            steps {
                script {
                    dir("${WORKSPACE}/sonar") {
                        sh "docker build -t ${params.DOCKERHUB_USERNAME}/sonar-scanner-cli:${BUILD_NUMBER} ."
                    }
                }
            }
        }

        stage('SonarQube Analysis') {
            when {
                expression { params.RUN_STAGES }
            }
            steps {
                script {
                    withSonarQubeEnv("SonarScanner") {
                        docker.image("${params.DOCKERHUB_USERNAME}/sonar-scanner-cli:${BUILD_NUMBER}").inside {
                            sh "sonar-scanner"
                        }
                    }
                }
            }
        }

        stage('Build Application') {
            when {
                expression { params.RUN_STAGES && !params.SKIP_STAGES }
            }
            steps {
                script {
                    sh "docker build -t ${params.DOCKERHUB_USERNAME}/catchup:${BUILD_NUMBER} ."
                }
            }
        }

        stage('Push Image to DockerHub') {
            when {
                expression { params.RUN_STAGES && !params.SKIP_STAGES }
            }
            steps {
                script {
                    sh "docker push ${params.DOCKERHUB_USERNAME}/catchup:${BUILD_NUMBER}"
                }
            }
        }

        stage('Login to DockerHub') {
            when {
                expression { params.RUN_STAGES }
            }
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'dockerhub-dylan', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                        sh "docker login -u ${USERNAME} -p ${PASSWORD}"
                    }
                }
            }
        }

        stage('Deploy Application') {
            steps {
                script {
                    sh "docker run -itd -p ${params.HOST_PORT}:80 --name ${params.CONTAINER_NAME} ${params.DOCKERHUB_USERNAME}/catchup:${BUILD_NUMBER}"
                    sh "docker ps | grep ${params.CONTAINER_NAME}"
                }
            }
        }

        stage('Delete Application') {
            when {
                expression { params.DELETE_APPLICATION }
            }
            steps {
                script {
                    sh "docker stop ${params.CONTAINER_NAME}"
                    sh "docker rm ${params.CONTAINER_NAME}"
                }
            }
        }
    }

    post {
        success {
            slackSend(channel: '#catchup-notify', message: "Job '${env.JOB_NAME}' completed successfully!")
        }
        failure {
            slackSend(channel: '#catchup-notify', message: "Job '${env.JOB_NAME}' failed!")
        }
    }
}
