pipeline {
    agent any
    environment {
        IMAGE_NAME = "vishnuops-game"
        IMAGE_TAG = "v1"
        GIT_REPO_URL = "https://github.com/vishnu-rv/argo.git"
        GIT_BRANCH = "main"
        GIT_CREDENTIALS_ID = "cbc59cf9-574a-4d91-985e-a5bfab43c77a"
        DOCKER_CREDENTIALS_ID = "7e7a9991-7b74-4877-bdb2-541ef7e31273"
        NAMESPACE = "game"
    }
    stages {
        stage('Clone Git Repository') {
            steps {
                git branch: "${GIT_BRANCH}", url: "${GIT_REPO_URL}", credentialsId: "${GIT_CREDENTIALS_ID}"
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    sh """
                    docker build -t ${IMAGE_NAME}:${IMAGE_TAG} .
                    """
                }
            }
        }
        stage('Push Docker Image to Registry') {
            steps {
                withDockerRegistry([credentialsId: "${DOCKER_CREDENTIALS_ID}", url: '']) {
                    sh """
                    docker tag ${IMAGE_NAME}:${IMAGE_TAG} vishnuops-game:${IMAGE_TAG}
                    docker push vishnuops-game:${IMAGE_TAG}
                    """
                }
            }
        }
        stage('Update Kubernetes Manifest') {
            steps {
                script {
                    // Update deployment.yaml with the new image version
                    sh """
                    sed -i 's|image: .*|image: ${IMAGE_NAME}:${IMAGE_TAG}|' game/deployment.yaml
                    """
                }
            }
        }
        stage('Commit and Push Updated Manifest to Git') {
            steps {
                script {
                    sh """
                    git config --global user.name "Your Name"
                    git config --global user.email "your.email@example.com"
                    git add game/deployment.yaml
                    git commit -m "Updated image to ${IMAGE_NAME}:${IMAGE_TAG}"
                    git push origin ${GIT_BRANCH}
                    """
                }
            }
        }
        stage('Deploy with ArgoCD') {
            steps {
                script {
                    sh """
                    argocd app sync game-app
                    """
                }
            }
        }
    }
    post {
        success {
            echo "Pipeline completed successfully!"
        }
        failure {
            echo "Pipeline failed. Check logs for details."
        }
    }
}

