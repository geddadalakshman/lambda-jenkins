def call(body) {
    // Evaluate the body block and collect configuration into the object
    def config = [:]
    body.resolveStrategy = Closure.DELEGATE_FIRST
    body.delegate = config
    body()

    def branch = env.BRANCH_NAME
    def mergebranch = "master"
    def qabranch = "QA"
    def yes = "yes"
    def no = "no"
    def SUCCESS = "SUCCESS"

    if (body != null && body['mergebranch'] != null) {
        mergebranch = body['mergebranch']
    }

    config['mergebranch'] = mergebranch

    pipeline {
        agent {
            label 'linux'
        }
        options {
            buildDiscarder(logRotator(numToKeepStr: '10'))
        }
        environment {
            CURRENT_TIME = sh(script: 'date +%Y%m%d%H%M%S', returnStdout: true).trim()
            CURRENT_DIR = pwd()
            REPO_DIR = "${CURRENT_DIR}"
        }

        stages {
            stage('checkout') {
                steps {
                    script {
                        checkout scm
                        env.BUILD_AWS_ACCOUNT = sh(script: "curl -s -S 'http://169.254.169.254/latest/dynamic/instance-identity/document/' | jq -r '.account'", returnStdout: true)
                        env.JENKINS_FQDN = sh(script: 'echo ${BUILD_URL/https:\\/\\/} | cut -d "/" -f1', returnStdout: true).trim()
                        env.CODE_AUTHOR = sh(script: "git log -1 --no-merges --format='%ae' ${GIT_COMMIT}", returnStdout: true).trim()
                        env.CODE_MERGED = sh(script: "git log -1 --format='%ae' ${GIT_COMMIT}", returnStdout: true).trim()
                    }
                }
            }

            // stage('validate parameters') {
            //     steps {
            //         script {
            //             dir('app') {
            //                 // Read the YAML file
            //                 def yamlConfig = readYaml file: '${ENV}-deployvariables.yml'
            //                 // Iterate over each key-value pair and print them
            //                 yamlConfig.each { key, value ->
            //                     // Set each variable as a Jenkins env variable
            //                     env."${key.toUpperCase()}" = value
            //                 }
            //             }
            //             sh """
            //             #!bin/bash
            //             # Validation of parameter variables.
            //             if [ -z ${ENV} ]; then
            //                 echo '${ENV} is NULL. Please validate deploy-variables.sh'
            //                 exit 1
            //             fi
            //             if [ -z ${ECRREPO_NAME} ]; then
            //                 echo '${ECRREPO_NAME} is NULL. Please validate deploy-variables.sh'
            //                 exit 1
            //             fi
            //             if [ -z ${SERVICE_NAME} ]; then
            //                 echo '${SERVICE_NAME} is NULL. Please validate deploy-variables.sh'
            //                 exit 1
            //             fi
            //             if [ -z ${CLUSTER_NAME} ]; then
            //                 echo '${CLUSTER_NAME} is NULL. Please validate deploy-variables.sh'
            //                 exit 1
            //             fi
            //             """
            //         }
            //     }
            // }

            stage('build images') {
                steps {
                    script {
                        sh """
                        #!/bin/bash
                        echo "aws login"
                        aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 144538309574.dkr.ecr.us-east-1.amazonaws.com
                        echo "aws login end"
                        echo "${REPO_DIR}"
                        ls -lah
                        ##########################
                        ### 1. Build images
                        ########################## 
                        """   
                                                                    
                        def images = ['grpc', 'rest', 'web']

                        images.each { imageName ->
                            def buildCommand

                            if (!env.GH_TOKEN) {
                                buildCommand = "docker build --no-cache -t ${ECRREPO_NAME}-${imageName} " +
                                               "--build-arg GH_TOKEN=${GH_TOKEN} " +
                                               "--build-arg http_proxy=${HTTP_PROXY} " +
                                               "--build-arg https_proxy=${HTTPS_PROXY} " +
                                               "--build-arg HTTP_PROXY=${HTTP_PROXY} " +
                                               "--build-arg HTTPS_PROXY=${HTTPS_PROXY} " +
                                               "${REPO_DIR}/${imageName}"
                            } else {
                                buildCommand = "docker build --no-cache -t ${ECRREPO_NAME}-${imageName} " +
                                               "--build-arg http_proxy=${HTTP_PROXY} " +
                                               "--build-arg https_proxy=${HTTPS_PROXY} " +
                                               "--build-arg HTTP_PROXY=${HTTP_PROXY} " +
                                               "--build-arg HTTPS_PROXY=${HTTPS_PROXY} " +
                                               "${REPO_DIR}/${imageName}"
                            }

                            sh """
                            echo "Building ${imageName} image"
                            ${buildCommand}
                            """
                        }
                    }
                }
            }
        }
    }
}
