pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
                echo "Building branch: ${env.BRANCH_NAME}"
                // Add your build steps here, e.g., compile code, run tests, etc.
            }
        }

        stage('Deploy') {
            when {
                branch 'dev'
            }
            steps {
                echo "Deploying to dev environment"
                // Add your dev deployment steps here
            }
        }

        stage('Deploy to Prod') {
            when {
                branch 'prod'
            }
            steps {
                echo "Deploying to prod environment"
                // Add your prod deployment steps here
            }
        }
    }

    post {
        always {
            echo "Cleaning up"
            cleanWs() // Clean up the workspace
        }
    }
}
