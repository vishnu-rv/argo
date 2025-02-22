pipeline {
    agent any
    
    environment {
        DOCKER_CREDENTIALS_ID = // Docker credentials ID
        GIT_CREDENTIALS_ID =      // Git credentials ID
        DOCKER_REPO =   // Full image path for Docker
        GIT_REPO =   // Git repository URL
        GIT_BRANCH =   // Branch name (adjust if needed)
    }
    
    stages {
        // Checkout the Git repository
        stage('Checkout SCM') {
            steps {
                script {
                    // Checkout the code from the specified GitHub repository using credentials
                    checkout([$class: 'GitSCM', 
                        branches: [[name: "*/${GIT_BRANCH}"]], 
                        doGenerateSubmoduleConfigurations: false, 
                        extensions: [], 
                        userRemoteConfigs: [[url: "${GIT_REPO}", credentialsId: "${GIT_CREDENTIALS_ID}"]]
                    ])
                }
            }
        }
        
        // Build Docker image and push it to the registry
        stage('Push Docker Image to Registry') {
            steps {
                script {
                    // Login to Docker registry using Jenkins credentials
                    withDockerRegistry([credentialsId: "${DOCKER_CREDENTIALS_ID}"]) {
                        sh 'docker login -u $DOCKER_USERNAME -p $DOCKER_PASSWORD https://index.docker.io/v1/'
                    }
                }
            }
        }
        
        // Pull Docker image from the registry
        stage('Pull Docker Image') {
            steps {
                script {
                    // Pull the Docker image using full image path
                    sh "docker pull ${DOCKER_REPO}"
                }
            }
        }
        
        // Update Kubernetes manifest (you can customize this step)
        stage('Update Kubernetes Manifest') {
            steps {
                script {
                    // Assuming you are modifying a file or some part of the manifest
                    echo "Updating Kubernetes manifest with new Docker image"
                    // Example: sed or similar to update the image version in Kubernetes YAML file
                    sh "sed -i 's|image: .*|image: ${DOCKER_REPO}|' k8s/manifest.yaml"
                }
            }
        }
        
        // Commit and push the updated manifest back to Git
        stage('Commit and Push Updated Manifest to Git') {
            steps {
                script {
                    // Add, commit, and push the updated Kubernetes manifest
                    sh '''
                    git config --global user.email "your-email@example.com"
                    git config --global user.name "Your Name"
                    git add k8s/manifest.yaml
                    git commit -m "Updated Kubernetes manifest with new Docker image"
                    git push origin ${GIT_BRANCH}
                    '''
                }
            }
        }
        
        // Deploy with ArgoCD
        stage('Deploy with ArgoCD') {
            steps {
                script {
                    // Example: Use ArgoCD CLI to sync the app
                    echo "Deploying to Kube
