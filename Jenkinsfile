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
                // Checks out the code from the Git repo you configured in the Jenkins job
                checkout scm 
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    // Uses the checked-out Dockerfile to build the image
                    docker.build("${DOCKER_IMAGE_NAME}:${env.BUILD_NUMBER}")
                }
            }
        }
        
        stage('Tag & Push to ACR') {
            steps {
                script {
                    def acrImage = "${ACR_LOGIN_SERVER}/${DOCKER_IMAGE_NAME}:${env.BUILD_NUMBER}"
                    docker.withRegistry("https://${ACR_LOGIN_SERVER}", AZURE_CREDENTIALS_ID) {
                        docker.image(DOCKER_IMAGE_NAME + ":${env.BUILD_NUMBER}").push()
                    }
                }
            }
        }

        stage('Deploy to AKS') {
            steps {
                script {
                    def imageTag = env.BUILD_NUMBER
                    def acrImage = "${ACR_LOGIN_SERVER}/${DOCKER_IMAGE_NAME}:${imageTag}"
                    
                    // This assumes your deployment.yaml is in your Git repo
                    sh "sed -i 's|image: .*|image: ${acrImage}|' deployment.yaml"
                    
                    sh "kubectl apply -f deployment.yaml"
                    sh "kubectl apply -f service.yaml"
                }
            }
        }
    }
}