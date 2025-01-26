pipeline {
    agent any
    environment {
        IMAGE_NAME = "vishnuops-game"    // Docker image name
        IMAGE_TAG = "v1"                // Image tag
        GIT_REPO_URL = "https://github.com/vishnu-rv/argo.git"
        GIT_BRANCH = "main"
        GIT_CREDENTIALS_ID = "cbc59cf9-574a-4d91-985e-a5bfab43c77a"
        DOCKER_CREDENTIALS_ID = "7e7a9991-7b74-4877-bdb2-541ef7e31273"
        NAMESPACE = "game"              // Kubernetes namespace
    }
    stages {
        stage('Clone Git Repository') {
            steps {
                git branch: "${GIT_BRANCH}", url: "${GIT_REPO_URL}", credentialsId: "${GIT_CREDENTIALS_ID}"
            }
        }
        stage('Push Docker Image to Registry') {
            steps {
                script {
                    withDockerRegistry([credentialsId: "${DOCKER_CREDENTIALS_ID}", url: '']) {
                        sh """
                        docker pull ${IMAGE_NAME}:${IMAGE_TAG}
                        docker tag ${IMAGE_NAME}:${IMAGE_TAG} <your-registry-url>/${IMAGE_NAME}:${IMAGE_TAG}
                        docker push <your-registry-url>/${IMAGE_NAME}:${IMAGE_TAG}
                        """
                    }
                }
            }
        }
        stage('Update Kubernetes Manifest') {
            steps {
                script {
                    // Update Kubernetes deployment.yaml with the new Docker image
                    sh """
                    sed -i 's|image: .*|image: <your-registry-url>/${IMAGE_NAME}:${IMAGE_TAG}|' game/deployment.yaml
                    """
                }
            }
        }
        stage('Commit and Push Updated Manifest to Git') {
            steps {
                script {
                    sh """
                    git config --global user.name "vishnu-rv"
                    git config --global user.email "vishnuofficial2117@gmail.com"
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
