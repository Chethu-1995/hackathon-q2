pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "yourdockerhubusername/sample-app"
        DOCKERHUB_CRED_ID = 'dockerhub'   // Jenkins credential ID for Docker Hub
        GCHAT_WEBHOOK_URL = 'https://chat.googleapis.com/v1/spaces/xxxx/messages?key=yyyy&token=zzzz' // Google Chat webhook URL
    }

    stages {
        stage('Clone Repository') {
            steps {
                git 'https://github.com/YOUR_USERNAME/sample-app.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    dockerImage = docker.build("${DOCKER_IMAGE}:${env.BUILD_ID}")
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    docker.withRegistry('', DOCKERHUB_CRED_ID) {
                        dockerImage.push()
                        dockerImage.push('latest')
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    sh """
                    kubectl apply -f k8s-deployment.yml
                    kubectl set image deployment/sample-app sample-app=${DOCKER_IMAGE}:${env.BUILD_ID}
                    kubectl rollout status deployment/sample-app
                    """
                }
            }
        }

        stage('Send Notification') {
            steps {
                script {
                    def message = """
                    {
                        "text": "Build ${env.BUILD_ID} completed successfully!"
                    }
                    """
                    sh """
                    curl -X POST -H 'Content-Type: application/json' \
                    -d '${message}' ${GCHAT_WEBHOOK_URL}
                    """
                }
            }
        }
    }

    post {
        failure {
            script {
                def message = """
                {
                    "text": "Build ${env.BUILD_ID} failed!"
                }
                """
                sh """
                curl -X POST -H 'Content-Type: application/json' \
                -d '${message}' ${GCHAT_WEBHOOK_URL}
                """
            }
        }
    }
}
