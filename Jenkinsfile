pipeline {
    agent any

    environment {
        DOCKER_CREDENTIALS = credentials('tanuj-dockerhub')
        BACKEND_REPO = 'https://github.com/tanujbhatia24/backend_kube.git'
        FRONTEND_REPO = 'https://github.com/tanujbhatia24/frontend_kube.git'
    }
    stages {
        stage('Clone Backend') {
            steps {
                dir('backend') {
                    git url: "${BACKEND_REPO}", branch: 'main'
                }
            }
        }

        stage('Clone Frontend') {
            steps {
                dir('frontend') {
                    git url: "${FRONTEND_REPO}", branch: 'main'
                }
            }
        }

        stage('Build & Push Backend Image (if changed)') {
            steps {
                dir('backend') {
                    script {
                        def commitHash = sh(script: "git rev-parse --short HEAD", returnStdout: true).trim()
                        env.BACKEND_TAG = "tanujbhatia24/backend_kube:${commitHash}"

                        def backendChanged = sh(script: "git diff --quiet HEAD~1 HEAD . || echo changed", returnStdout: true).trim()
                        if (backendChanged == "changed") {
                            sh """
                                docker build -t ${env.BACKEND_TAG} .
                                echo "$DOCKER_CREDENTIALS_PSW" | docker login -u "$DOCKER_CREDENTIALS_USR" --password-stdin
                                docker push ${env.BACKEND_TAG}
                            """
                        } else {
                            echo "No backend changes detected. Skipping backend image build."
                        }
                    }
                }
            }
        }

        stage('Build & Push Frontend Image (if changed)') {
            steps {
                dir('frontend') {
                    script {
                        def commitHash = sh(script: "git rev-parse --short HEAD", returnStdout: true).trim()
                        env.FRONTEND_TAG = "tanujbhatia24/frontend_kube:${commitHash}"

                        def frontendChanged = sh(script: "git diff --quiet HEAD~1 HEAD . || echo changed", returnStdout: true).trim()
                        if (frontendChanged == "changed") {
                            sh """
                                docker build -t ${env.FRONTEND_TAG} .
                                echo "$DOCKER_CREDENTIALS_PSW" | docker login -u "$DOCKER_CREDENTIALS_USR" --password-stdin
                                docker push ${env.FRONTEND_TAG}
                            """
                        } else {
                            echo "No frontend changes detected. Skipping frontend image build."
                        }
                    }
                }
            }
        }

        stage('Deploy via Helm') {
            steps {
                script {
                    // Inject the image tags into Helm values dynamically
                    sh """
                        helm upgrade --install mern ./mern-chart \
                          --set image.backend=${env.BACKEND_TAG} \
                          --set image.frontend=${env.FRONTEND_TAG} \
                          --namespace mern --create-namespace
                    """
                }
            }
        }
    }
}
