
pipeline {
    agent any

    environment {
        // Replace with your Docker Hub username and repository name
        DOCKER_IMAGE_NAME = 'YOUR_DOCKERHUB_USERNAME/static-website'
        TAG = 'latest'
        
        // The ID of the credential you created in Jenkins for Docker Hub
        DOCKER_CREDENTIALS_ID = 'dockerhub-creds'
    }

    stages {
        stage('Clone') {
            steps {
                // Automatically pulls the latest source code from your connected GitHub repository
                checkout scm
            }
        }

        stage('Build') {
            steps {
                script {
                    echo "Building Docker image from Dockerfile..."
                    app = docker.build("${DOCKER_IMAGE_NAME}:${TAG}")
                }
            }
        }

        stage('Test') {
            steps {
                script {
                    echo "Validating that the container runs locally and the website is accessible on port 8081..."
                    
                    // Run container in background mapping port 8081
                    sh "docker run -d --name test-website-container -p 8081:80 ${DOCKER_IMAGE_NAME}:${TAG}"
                    
                    // Perform a curl request to verify the web server is responding on port 8081
                    sh "curl -I http://localhost:8081"
                    
                    // Clean up test container
                    sh "docker rm -f test-website-container"
                }
            }
        }

        stage('Push') {
            steps {
                script {
                    echo "Securely authenticating and pushing final image to Docker Hub..."
                    docker.withRegistry('https://registry.hub.docker.com', "${DOCKER_CREDENTIALS_ID}") {
                        app.push("${TAG}")
                    }
                }
            }
        }
    }

    post {
        always {
            // Cleans up the Jenkins workspace
            cleanWs()
        }
        failure {
            // Ensures leftover test containers are purged if a test fails
            sh "docker rm -f test-website-container || true"
        }
    }
}
