pipeline {
    agent any

    environment {
        REGISTRY = "tanujbhatia24"
        FRONTEND_REPO = "https://github.com/tanujbhatia24/frontend_kube.git"
        BACKEND_REPO = "https://github.com/tanujbhatia24/backend_kube.git"
        FRONTEND_IMAGE = "${REGISTRY}/frontend"
        BACKEND_IMAGE = "${REGISTRY}/backend"
    }

    stages {
        stage('Clone and Check for Changes') {
            steps {
                script {
                    def frontendChanged = false
                    def backendChanged = false

                    dir('frontend') {
                        git url: "${env.FRONTEND_REPO}"
                        frontendChanged = sh(script: "git rev-parse HEAD > ../.frontend_commit && git diff --quiet HEAD || echo changed", returnStdout: true).contains("changed")
                    }
                    dir('backend') {
                        git url: "${env.BACKEND_REPO}"
                        backendChanged = sh(script: "git rev-parse HEAD > ../.backend_commit && git diff --quiet HEAD || echo changed", returnStdout: true).contains("changed")
                    }

                    if (!frontendChanged && !backendChanged) {
                        echo "No new revisions to build."
                        currentBuild.result = 'SUCCESS'
                        // Exit early
                        return
                    }
                }
            }
        }

        stage('Build Docker Images') {
            when {
                expression {
                    fileExists('.frontend_commit') || fileExists('.backend_commit')
                }
            }
            steps {
                script {
                    if (fileExists('.frontend_commit')) {
                        sh "docker build -t ${FRONTEND_IMAGE}:latest ./frontend"
                    }
                    if (fileExists('.backend_commit')) {
                        sh "docker build -t ${BACKEND_IMAGE}:latest ./backend"
                    }
                }
            }
        }

        stage('Push Images to Registry') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                    sh "echo $PASSWORD | docker login -u $USERNAME --password-stdin"
                    if (fileExists('.frontend_commit')) {
                        sh "docker push ${FRONTEND_IMAGE}:latest"
                    }
                    if (fileExists('.backend_commit')) {
                        sh "docker push ${BACKEND_IMAGE}:latest"
                    }
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
