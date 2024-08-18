pipeline {
    agent any

    environment {
        AWS_REGION = 'us-east-1'
        AWS_CREDENTIALS_ID = 'aws-credentials'
        S3_BUCKET_NAME = ''
        GITHUB_REPO = ''
        FILE_NAME = 'main.py'
    }

    stages {
        stage('Determine Environment and GitHub Repo') {
            steps {
                withAWS(credentials: "${env.AWS_CREDENTIALS_ID}", region: "${env.AWS_REGION}") {
                    script {
                        // Determine the environment based on the branch name
                        def environment = env.BRANCH_NAME // Assuming BRANCH_NAME is used to indicate the environment

                        // Example logic to determine GitHub repository based on environment
                        if (environment == 'dev') {
                            env.GITHUB_REPO = 'https://github.com/geddadalakshman/lambda-jenkins.git'
                        } else if (environment == 'QA') {
                            env.GITHUB_REPO = 'https://github.com/your-org/qa-repo.git'
                        } else if (environment == 'prod') {
                            env.GITHUB_REPO = 'https://github.com/your-org/prod-repo.git'
                        } else {
                            error("Unsupported environment: ${environment}")
                        }

                        // Fetch and filter S3 buckets based on environment
                        def buckets = sh(script: "aws s3api list-buckets --query 'Buckets[].Name' --output text", returnStdout: true).trim().split()

                        // Filter buckets based on naming convention
                        def matchingBucket = buckets.find { it.contains(environment) }

                        if (matchingBucket) {
                            env.S3_BUCKET_NAME = matchingBucket
                            echo "Using S3 bucket: ${env.S3_BUCKET_NAME}"
                        } else {
                            error "No matching S3 bucket found for environment: ${environment}"
                        }
                    }
                }
            }
        }

        stage('Checkout Code') {
            steps {
                // Checkout the code from the determined GitHub repository
                git branch: 'main', url: "${env.GITHUB_REPO}"
            }
        }

        stage('Zip Folder') {
            steps {
                script {
                    // def folderName = 'your-folder-name'
                    sh "zip -r ${env.FILE_NAME}.zip ${env.FILE_NAME}"
                }
            }
        }

        stage('Upload to S3') {
            steps {
                withAWS(credentials: "${env.AWS_CREDENTIALS_ID}", region: "${env.AWS_REGION}") {
                    sh """
                    aws s3 cp ${env.FILE_NAME}.zip s3://${env.S3_BUCKET_NAME}/${env.FILE_NAME}.zip
                    """
                }
            }
        }
    }

    post {
        always {
            cleanWs()
        }
    }
}
