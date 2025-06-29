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
                    def frontendCommit = ''
                    def backendCommit = ''
                    def previousFrontendCommit = readFile(file: '.last_frontend_commit', encoding: 'UTF-8')?.trim() ?: ''
                    def previousBackendCommit = readFile(file: '.last_backend_commit', encoding: 'UTF-8')?.trim() ?: ''

                    dir('frontend') {
                        git url: "${env.FRONTEND_REPO}"
                        frontendCommit = sh(script: "git rev-parse HEAD", returnStdout: true).trim()
                    }

                    dir('backend') {
                        git url: "${env.BACKEND_REPO}"
                        backendCommit = sh(script: "git rev-parse HEAD", returnStdout: true).trim()
                    }

                    def frontendChanged = (frontendCommit != previousFrontendCommit)
                    def backendChanged = (backendCommit != previousBackendCommit)

                    if (!frontendChanged && !backendChanged) {
                        echo "✅ No new revisions to build."
                        currentBuild.result = 'SUCCESS'
                        // Exit early
                        return
                    }

                    // Save new commit hashes for next run
                    writeFile file: '.last_frontend_commit', text: frontendCommit
                    writeFile file: '.last_backend_commit', text: backendCommit

                    // Set env vars for later stages
                    env.FRONTEND_CHANGED = "${frontendChanged}"
                    env.BACKEND_CHANGED = "${backendChanged}"
                }
            }
        }

        stage('Build Docker Images') {
            steps {
                script {
                    if (env.FRONTEND_CHANGED == 'true') {
                        echo "🔧 Building Frontend Docker image"
                        sh "docker build -t ${FRONTEND_IMAGE}:latest ./frontend"
                    }
                    if (env.BACKEND_CHANGED == 'true') {
                        echo "🔧 Building Backend Docker image"
                        sh "docker build -t ${BACKEND_IMAGE}:latest ./backend"
                    }
                }
            }
        }

        stage('Push Images to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                    script {
                        sh "echo $PASSWORD | docker login -u $USERNAME --password-stdin"
                        if (env.FRONTEND_CHANGED == 'true') {
                            sh "docker push ${FRONTEND_IMAGE}:latest"
                        }
                        if (env.BACKEND_CHANGED == 'true') {
                            sh "docker push ${BACKEND_IMAGE}:latest"
                        }
                    }
                }
            }
        }

        stage('Deploy with Helm') {
            steps {
                script {
                    echo "🚀 Deploying using Helm"
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
