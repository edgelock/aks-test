pipeline {
    agent any

    environment {
        ACR_LOGIN_SERVER  = 'containerregistry1alskdmalksdma.azurecr.io'
        AZURE_CREDENTIALS_ID = 'azure-service-principal' // The ID you created in Jenkins
        DOCKER_IMAGE_NAME = 'my-simple-app'
    }
    
    stages {
        stage('Checkout Source') {
            steps {
                checkout scm 
            }
        }

        stage('Build Docker Image') {
            steps {
                // Change into the directory with the Dockerfile
                dir('simple-web-app') {
                    script {
                        docker.build("${DOCKER_IMAGE_NAME}:${env.BUILD_NUMBER}")
                    }
                }
            }
        }
        
        stage('Tag & Push to ACR') {
            steps {
                script {
                    docker.withRegistry("https://${ACR_LOGIN_SERVER}", AZURE_CREDENTIALS_ID) {
                        def acrImage = "${ACR_LOGIN_SERVER}/${DOCKER_IMAGE_NAME}:${env.BUILD_NUMBER}"
                        docker.image("${DOCKER_IMAGE_NAME}:${env.BUILD_NUMBER}").push(acrImage)
                    }
                }
            }
        }

        stage('Deploy to AKS') {
            steps {
                // Change into the directory with the yaml files
                dir('simple-web-app') {
                    script {
                        def imageTag = env.BUILD_NUMBER
                        def acrImage = "${ACR_LOGIN_SERVER}/${DOCKER_IMAGE_NAME}:${imageTag}"
                        
                        // Use your actual filenames: my-webapp.yaml and my-webapp-service.yaml
                        sh "sed -i 's|image: .*|image: ${acrImage}|' my-webapp.yaml"
                        
                        sh "kubectl apply -f my-webapp.yaml"
                        sh "kubectl apply -f my-webapp-service.yaml"
                    }
                }
            }
        }
    }
}