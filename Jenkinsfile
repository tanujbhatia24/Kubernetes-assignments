pipeline {
    agent any

    environment {
        REGISTRY = "tanujbhatia24"
        FRONTEND_REPO = "https://github.com/tanujbhatia24/frontend_kube.git"
        BACKEND_REPO = "https://github.com/tanujbhatia24/backend_kube.git"
        FRONTEND_IMAGE = "${REGISTRY}/mern-frontend"
        BACKEND_IMAGE = "${REGISTRY}/mern-backend"
    }

    stages {

        stage('Clone Repos') {
            steps {
                dir('frontend') {
                    git url: "${env.FRONTEND_REPO}"
                }
                dir('backend') {
                    git url: "${env.BACKEND_REPO}"
                }
            }
        }

        stage('Build Docker Images') {
            steps {
                script {
                    sh "docker build -t ${FRONTEND_IMAGE}:latest ./frontend"
                    sh "docker build -t ${BACKEND_IMAGE}:latest ./backend"
                }
            }
        }

        stage('Push Images to Registry') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                    sh "echo $PASSWORD | docker login -u $USERNAME --password-stdin"
                    sh "docker push ${FRONTEND_IMAGE}:latest"
                    sh "docker push ${BACKEND_IMAGE}:latest"
                }
            }
        }

        stage('Deploy with Helm') {
            steps {
                script {
                    sh """
                        helm upgrade --install mern-app ./helm-chart \
                            --set frontend.image.repository=${FRONTEND_IMAGE} \
                            --set backend.image.repository=${BACKEND_IMAGE}
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
