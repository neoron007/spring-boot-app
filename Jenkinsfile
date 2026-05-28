pipeline {
    agent any

    environment {
        DOCKER_USER = 'roshanurkude01'
        DOCKER_IMAGE = 'spring-boot-app'
        IMAGE_TAG = '${BUILD_NUMBER}'
        PORT_HOST = 3000
        PORT_CON = 3000
        DOCKER_CREDS = credentials('dockerhub-credentials')
    }

    stages {
        stage('1. Checkout Code') {
            steps {
                // Clone Source Code into the local Jenkins workspace folder from scm
                checkout scm
            }
        }

        stage('2. Build Image') {
            steps {
                // Compile the app and create docker image locally
                sh "docker build -t ${DOCKER_USER}/${DOCKER_IMAGE}:${IMAGE_TAG} ."
                sh "docker tag ${DOCKER_USER}/${DOCKER_IMAGE}:${IMAGE_TAG} ${DOCKER_USER}/${DOCKER_IMAGE}:latest"
            }
        }

        stage('3. Push to Docker Hub') {
            steps {
                // Log into docker hub and push the image to it
                sh "echo \$DOCKER_CREDS_PSW | docker login -u \$DOCKER_CREDS_USR --password-stdin"
                sh "docker push ${DOCKER_USER}/${DOCKER_IMAGE}:${IMAGE_TAG}"
                sh "docker push ${DOCKER_USER}/${DOCKER_IMAGE}:latest"
                sh "docker logout"
            }
        }

        stage('4. Deployment') {
            steps {
                echo "Deploying"

                // Stop and remove old container versions
                sh "docker stop ${DOCKER_IMAGE} || true"
                sh "docker rm ${DOCKER_IMAGE} || true"

                // Launch the new container instantly using the local image copy
                sh "docker run -d -p 3000:3000 --name ${DOCKER_IMAGE} ${DOCKER_USER}/${DOCKER_IMAGE}:${IMAGE_TAG}"
            }
        }

        stage('5. Health Check') {
             steps {
                // Wait 10 seconds for the Spring Boot application to finish starting up
                sleep time: 10, unit: 'SECONDS'
                
                // Test the local endpoint directly
                sh "curl -I http://localhost:3000/index.html"
            }
        }

        post {
            always {
                echo "Successfully deployed."
            }
        }
    }
}