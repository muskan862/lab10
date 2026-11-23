pipeline {
    agent any

    environment {
        DOCKERHUB_USER = 'your-dockerhub-username'
        IMAGE_NAME     = 'exp9-webapp'
        IMAGE_TAG      = "${BUILD_NUMBER}"
        CONTAINER_NAME = 'exp9-webapp-live'
        // Use 'local' for local Docker deployment or 'cloud' for remote SSH deployment
        DEPLOY_MODE    = 'local'
        CLOUD_HOST     = 'your-cloud-server-ip'
        CLOUD_USER     = 'ubuntu'
        HOST_PORT      = '8082'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/your-username/your-repo.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh """
                    docker build -t ${DOCKERHUB_USER}/${IMAGE_NAME}:${IMAGE_TAG} .
                    docker tag ${DOCKERHUB_USER}/${IMAGE_NAME}:${IMAGE_TAG} ${DOCKERHUB_USER}/${IMAGE_NAME}:latest
                """
            }
        }

        stage('Push to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DH_USER', passwordVariable: 'DH_PASS')]) {
                    sh """
                        echo "$DH_PASS" | docker login -u "$DH_USER" --password-stdin
                        docker push ${DOCKERHUB_USER}/${IMAGE_NAME}:${IMAGE_TAG}
                        docker push ${DOCKERHUB_USER}/${IMAGE_NAME}:latest
                    """
                }
            }
        }

        stage('Deploy Container') {
            steps {
                script {
                    if (env.DEPLOY_MODE == 'cloud') {
                        sshagent(credentials: ['deploy-ssh-key']) {
                            sh """
                                ssh -o StrictHostKeyChecking=no ${CLOUD_USER}@${CLOUD_HOST} '
                                    docker pull ${DOCKERHUB_USER}/${IMAGE_NAME}:latest &&
                                    docker stop ${CONTAINER_NAME} || true &&
                                    docker rm ${CONTAINER_NAME} || true &&
                                    docker run -d --name ${CONTAINER_NAME} -p ${HOST_PORT}:80 ${DOCKERHUB_USER}/${IMAGE_NAME}:latest
                                '
                            """
                        }
                    } else {
                        sh """
                            docker pull ${DOCKERHUB_USER}/${IMAGE_NAME}:latest
                            docker stop ${CONTAINER_NAME} || true
                            docker rm ${CONTAINER_NAME} || true
                            docker run -d --name ${CONTAINER_NAME} -p ${HOST_PORT}:80 ${DOCKERHUB_USER}/${IMAGE_NAME}:latest
                        """
                    }
                }
            }
        }
    }

    post {
        success {
            echo "Deployment completed successfully on port ${HOST_PORT}."
        }
        failure {
            echo "Pipeline failed. Inspect the Jenkins console log for details."
        }
    }
}