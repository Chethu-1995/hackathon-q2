pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "chethangowdahm/hackathon-q2"
        DOCKERHUB_CRED_ID = 'dockerhub'   // Jenkins credential ID for Docker Hub
        K8S_CRED_ID = 'k8s-token' // Jenkins credential ID for Kubernetes
        GCHAT_WEBHOOK_URL = 'https://chat.googleapis.com/v1/spaces/AAAAvJhslpE/messages?key=AIzaSyDdI0hCZtE6vySjMm-WEfRq3CPzqKqqsHI&token=XORFOGjWMMWYge0exFNYZEcdEEFopo-cR6PiLBnLJ-w' // Google Chat webhook URL
    }

    stages {
        stage('Clone Repository') {
            steps {
                git 'https://github.com/Chethu-1995/hackathon-q2.git'
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
                    withCredentials([string(credentialsId: K8S_CRED_ID, variable: 'K8S_TOKEN')]) {
                        sh """
                        kubectl apply -f k8s-deployment.yml
                        kubectl set image deployment/sample-app sample-app=${DOCKER_IMAGE}:${env.BUILD_ID}
                        kubectl rollout status deployment/sample-app
                        """
                    }
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
