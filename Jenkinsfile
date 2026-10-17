
pipeline {
    agent any

    environment {
        IMAGE_NAME = 'YOUR_DOCKERHUB_USERNAME/jenkins-static-website'
    }

    stages {
        stage('Clone') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                sh 'docker build -t $IMAGE_NAME:$BUILD_NUMBER .'
                sh 'docker tag $IMAGE_NAME:$BUILD_NUMBER $IMAGE_NAME:latest'
            }
        }

        stage('Test') {
            steps {
                sh '''
                    docker rm -f website-ci-test || true
                    docker run -d --name website-ci-test \
                        -p 18081:80 $IMAGE_NAME:$BUILD_NUMBER

                    for i in $(seq 1 15); do
                        if curl -fsS http://localhost:18081/ -o /dev/null; then
                            echo "Website test passed"
                            docker rm -f website-ci-test
                            exit 0
                        fi
                        sleep 2
                    done

                    docker logs website-ci-test || true
                    docker rm -f website-ci-test || true
                    exit 1
                '''
            }
        }

        stage('Push to Docker Hub') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-creds',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )
                ]) {
                    sh '''
                        set +x
                        echo "$DOCKER_PASS" | docker login \
                            -u "$DOCKER_USER" --password-stdin

                        docker push $IMAGE_NAME:$BUILD_NUMBER
                        docker push $IMAGE_NAME:latest

                        docker logout
                    '''
                }
            }
        }

        stage('Deploy') {
            steps {
                sh '''
                    docker rm -f static-website || true
                    docker run -d \
                        --restart unless-stopped \
                        --name static-website \
                        -p 8081:80 $IMAGE_NAME:$BUILD_NUMBER

                    curl -fsS http://localhost:8081/ -o /dev/null
                    echo "Deployment successful"
                '''
            }
        }
    }

    post {
        always {
            sh 'docker rm -f website-ci-test || true'
        }
        success {
            echo 'CI/CD pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed. Check the Jenkins console output.'
        }
    }
}

