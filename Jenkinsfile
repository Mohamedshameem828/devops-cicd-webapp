pipeline {
    agent any

    options {
        timestamps()
        disableConcurrentBuilds()
    }

    environment {
        IMAGE_REPO = 'shameemdoc/devops-cicd-webapp'
        DOCKERHUB_CREDENTIALS = 'dockerhub-creds'
        K8S_NAMESPACE = 'devops-demo'
        K8S_DEPLOYMENT = 'webapp'
        K8S_CONTAINER = 'nginx'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
                script {
                    env.IMAGE_TAG = sh(
                        script: 'git rev-parse --short=12 HEAD',
                        returnStdout: true
                    ).trim()
                    env.FULL_IMAGE = "${env.IMAGE_REPO}:${env.IMAGE_TAG}"
                }
            }
        }

        stage('Validate Source') {
            steps {
                sh '''
                    set -eu
                    test -f index.html
                    test -f style.css
                    test -f script.js
                    test -f Dockerfile
                    echo "Required files are present."
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                    set -eu
                    docker build --pull -t "$FULL_IMAGE" .
                '''
            }
        }

        stage('Push Image to Docker Hub') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: "${DOCKERHUB_CREDENTIALS}",
                        usernameVariable: 'DOCKERHUB_USER',
                        passwordVariable: 'DOCKERHUB_TOKEN'
                    )
                ]) {
                    sh '''
                        set +x
                        echo "$DOCKERHUB_TOKEN" | docker login \
                          --username "$DOCKERHUB_USER" --password-stdin
                        docker push "$FULL_IMAGE"
                        docker logout
                    '''
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh '''
                    set -eu
                    kubectl set image \
                      deployment/"$K8S_DEPLOYMENT" \
                      "$K8S_CONTAINER"="$FULL_IMAGE" \
                      -n "$K8S_NAMESPACE"

                    kubectl rollout status \
                      deployment/"$K8S_DEPLOYMENT" \
                      -n "$K8S_NAMESPACE" \
                      --timeout=180s

                    kubectl get deployment "$K8S_DEPLOYMENT" \
                      -n "$K8S_NAMESPACE" -o wide
                '''
            }
        }
    }

    post {
        success {
            echo 'CI/CD pipeline completed successfully.'
        }
        failure {
            echo 'Pipeline failed. Inspect the failed stage and console output.'
        }
    }
}