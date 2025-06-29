pipeline {
    agent any

    environment {
        BACKEND_REPO = 'https://github.com/tanujbhatia24/backend_kube.git'
        FRONTEND_REPO = 'https://github.com/tanujbhatia24/frontend_kube.git'
    }

    stages {
        stage('Clone Backend') {
            steps {
                dir('backend') {
                    checkout([
                        $class: 'GitSCM',
                        branches: [[name: '*/main']],
                        userRemoteConfigs: [[url: "${BACKEND_REPO}"]],
                        extensions: [[$class: 'CloneOption', depth: 5, noTags: false, shallow: false]]
                    ])
                }
            }
        }

        stage('Clone Frontend') {
            steps {
                dir('frontend') {
                    checkout([
                        $class: 'GitSCM',
                        branches: [[name: '*/main']],
                        userRemoteConfigs: [[url: "${FRONTEND_REPO}"]],
                        extensions: [[$class: 'CloneOption', depth: 5, noTags: false, shallow: false]]
                    ])
                }
            }
        }

        stage('Build & Push Backend Image (if changed)') {
            steps {
                dir('backend') {
                    script {
                        def commitHash = sh(script: "git rev-parse --short HEAD", returnStdout: true).trim()
                        env.BACKEND_TAG = "tanujbhatia24/backend_kube:${commitHash}"

                        def backendChanged = sh(
                            script: "[ \$(git rev-list --count HEAD) -gt 1 ] && git diff --quiet HEAD~1 HEAD . || echo changed",
                            returnStdout: true
                        ).trim()

                        if (backendChanged == "changed") {
                            withCredentials([usernamePassword(credentialsId: 'tanuj-dockerhub', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                                sh """
                                    docker build -t $BACKEND_TAG .
                                    echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                                    docker push $BACKEND_TAG
                                """
                            }
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

                        def frontendChanged = sh(
                            script: "[ \$(git rev-list --count HEAD) -gt 1 ] && git diff --quiet HEAD~1 HEAD . || echo changed",
                            returnStdout: true
                        ).trim()

                        if (frontendChanged == "changed") {
                            withCredentials([usernamePassword(credentialsId: 'tanuj-dockerhub', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                                sh """
                                    docker build -t $FRONTEND_TAG .
                                    echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                                    docker push $FRONTEND_TAG
                                """
                            }
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

    post {
        always {
            echo "Pipeline execution completed."
        }
    }
}
