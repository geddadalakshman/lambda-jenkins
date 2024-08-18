pipeline {
    agent any

    environment {
        AWS_REGION = 'us-east-1'
        S3_BUCKET_NAME = 'sagemaker-us-east-1-533267196238'
        FILE_NAME = 'main.py'
    }

    stages {
        stage('Checkout') {
            steps {
                // Checkout the code from GitHub
                git branch: 'main', url: 'https://github.com/geddadalakshman/lambda-jenkins.git'
            }
        }
        
        stage('Zip Folder') {
            steps {
                script {
                    // Zip the specified folder
                    sh "zip -r ${env.FILE_NAME}.zip ${env.FILE_NAME}"
                }
            }
        }

        stage('Upload to S3') {
            steps {
                withAWS(credentials: 'aws-credentials', region: "${env.AWS_REGION}") {
                    // Upload the zip file to the S3 bucket
                    sh """
                    aws s3 cp ${env.FILE_NAME}.zip s3://${env.S3_BUCKET_NAME}/
                    """
                }
            }
        }
    }
    
    post {
        always {
            // Clean up the workspace.
            cleanWs()
        }
    }
}
