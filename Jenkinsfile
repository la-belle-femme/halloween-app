pipeline {
    agent any

    parameters {
        string(name: 'BRANCH_NAME', defaultValue: 'main', description: 'Enter the branch name to clone')
    }

    stages {
        stage('Clone Repository') {
            steps {
                script {
                    // Clone the repository using credentials and the specified branch
                    git branch: "${params.BRANCH_NAME}", 
                    credentialsId: 'catcup-token', 
                    url: 'https://github.com/chrisdylan237/halloween-app.git'
                }
            }
        }
        // Add more stages for your pipeline as needed
    }
    // Add post-build actions if required
}
