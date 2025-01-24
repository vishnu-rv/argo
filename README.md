
---

# ArgoCD with GitOps: Deploying Applications 🚀

## What is GitOps? 🌐

GitOps is a modern approach to continuous delivery that uses Git as the source of truth for both the desired state of the application and its infrastructure. In GitOps, all changes to the system (whether application code, configurations, or infrastructure) are made via pull requests to a Git repository. These changes are automatically applied to the system, ensuring that the live environment is always in sync with the repository.

### Key Benefits of GitOps:

Single Source of Truth: Everything, including infrastructure and applications, is defined in Git.
Automated and Continuous Deployment: Changes pushed to Git are automatically applied to the system without manual intervention.
Auditability: Git provides an audit trail for every change made to the system.
Security: Access control is managed through Git's permission system, ensuring a secure way to manage changes.

## GitOps Workflow:

Developers make changes to code or anything in a Git repository.
A tool like ArgoCD or Flux monitors the repository.
Whenever a change is detected in the Git repository, it is automatically deployed to the Kubernetes cluster, ensuring the system is up to date.

## What is ArgoCD? 🔧

**ArgoCD** is a declarative, GitOps continuous delivery tool for Kubernetes. It allows you to manage and deploy Kubernetes applications using Git repositories as the source of truth. This enables continuous delivery and automated synchronization of Kubernetes resources.

### Key Features of ArgoCD:
- **Declarative**: It ensures Kubernetes resources are in sync with the Git repository.
- **Self-healing**: ArgoCD automatically syncs and corrects the state if the deployed resources deviate from the desired state in Git.
- **Easy Rollbacks**: Changes can be rolled back easily to a previous state.
- **Real-time Feedback**: Provides real-time status of deployments and alerts for any issues.

---

## Why Use ArgoCD in GitOps? 🖥️

ArgoCD works seamlessly in a **GitOps** workflow by integrating with Git repositories. By utilizing Git as the source of truth, we ensure that our Kubernetes deployments are versioned, auditable, and easily traceable.

### Benefits of GitOps with ArgoCD:
- **Single Source of Truth**: The configuration files (like `deployment.yaml`, `service.yaml`) reside in a Git repository.
- **Version Control**: Changes to infrastructure or applications are tracked and can be rolled back using Git's version control.
- **Improved Collaboration**: Team members collaborate by modifying the same configuration files.
- **Security**: All changes are auditable, and access control is tied to the Git repository.

---

## Prerequisites ✅

Before starting, ensure you have the following tools:
- **ArgoCD** installed in your Kubernetes cluster.
- A **Git repository** with your Kubernetes manifests (deployment and service YAML files).
- **Jenkins (Optional)** for CI/CD automation.

---

## Step 1: Install ArgoCD ⚙️

To install **ArgoCD** on your Kubernetes cluster, follow these steps:

1. **Install ArgoCD** via the official method or Helm:
   - For installation via Helm, you can run:

   ```bash
   kubectl create namespace argocd
   helm repo add argo https://argoproj.github.io/argo-helm
   helm install argocd argo/argo-cd -n argocd
   ```

2. **Access ArgoCD UI**:
   To access ArgoCD's UI, update the service type to **LoadBalancer** and fetch the external IP:

   ```bash
   kubectl expose svc argocd-server --type=LoadBalancer --name=argocd-server -n argocd
   kubectl get svc -n argocd
   ```

   You can now access the UI using the external IP and port from the output.

---

## Step 2: Link Git Repository to ArgoCD 🔗

ArgoCD will pull the application manifests from your Git repository. You need to provide it access to the Git repository using a username and token for authentication.

1. **Add Git Repository to ArgoCD**:
   - Go to the ArgoCD UI.
   - Navigate to **Settings** > **Repositories** > **Connect Repo**.
   - Enter the **repository URL** (e.g., `https://github.com/vishnu-rv/argo.git`) and the **username** and **token** for authentication.
   - Save the changes.

ArgoCD will now have access to your Git repository, and it will automatically sync your Kubernetes manifests.

---

## Step 3: Create Application in ArgoCD 🛠️

To create an application in ArgoCD:

1. **Go to the ArgoCD UI** and select **+ New App**.
2. Enter the following details:
   - **Application Name**: `game`
   - **Project**: `default`
   - **Repository URL**: `https://github.com/vishnu-rv/argo.git`
   - **Revision**: `main` (or the branch you want to deploy from)
   - **Path**: `/` (path to the directory where your Kubernetes YAML files are located)
   - **Cluster**: Select your cluster (or the default cluster)
   - **Namespace**: `game` (the namespace where you want to deploy the application)

Once you have configured this, click **Create**. ArgoCD will pull the configuration files from the Git repository and deploy the application in the specified Kubernetes cluster.

(Here, you can add a screenshot of the creation process)

---

## Step 4: GitOps Workflow with ArgoCD 🔄

Once ArgoCD is set up and your application is created, any changes made to the Git repository will automatically trigger deployments.

1. **Push Changes to Git**:
   Modify your Kubernetes manifest files (like `deployment.yaml` and `service.yaml`) in the Git repository. For example, to update the Docker image version, change:

   ```yaml
   image: vishnu2117/vishnuops-game:v2
   ```

2. **Automatic Sync with ArgoCD**:
   ArgoCD will automatically detect changes in the Git repository and sync them to the Kubernetes cluster, deploying the updated version of the application.

   ![image](https://github.com/user-attachments/assets/aceb2f64-2e66-4361-804c-94a55a4a90cc)

   Output:

   ![Screencastfrom2025-01-2419-15-40-ezgif com-speed](https://github.com/user-attachments/assets/2ebd51d8-c19f-48ad-8b47-f3f7d9890b31)



---

## Step 5: Example Deployment and Service Files 📄

In this example, we created the following files:

1. **deployment.yaml**:
   This file describes the deployment, including the Docker image and replicas.

   ```yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: game-deployment
     namespace: game
   spec:
     replicas: 1
     selector:
       matchLabels:
         app: game
     template:
       metadata:
         labels:
           app: game
       spec:
         containers:
           - name: game
             image: vishnu2117/vishnuops-game:v1
             ports:
               - containerPort: 80
   ```

2. **service.yaml**:
   This file defines the LoadBalancer service to expose the app to the internet.

   ```yaml
   apiVersion: v1
   kind: Service
   metadata:
     name: game-service
     namespace: game
   spec:
     selector:
       app: game
     ports:
       - protocol: TCP
         port: 80
         targetPort: 80
     type: LoadBalancer
   ```

---

## Optional: Automating with Jenkins 🧑‍💻

You can also automate this deployment using **Jenkins**. If you have Jenkins set up in your pipeline, you can use the provided sample pipeline in the same repository to automate the build and deployment process.

Here is a sample Jenkins pipeline:

```groovy
pipeline {
    agent any
    environment {
        DOCKER_IMAGE = "vishnu2117/vishnuops-game"
        VERSION = "v1"
    }
    stages {
        stage('Build Docker Image') {
            steps {
                script {
                    sh 'docker build -t $DOCKER_IMAGE:$VERSION .'
                }
            }
        }
        stage('Push Docker Image') {
            steps {
                script {
                    sh 'docker push $DOCKER_IMAGE:$VERSION'
                }
            }
        }
        stage('Deploy to Kubernetes') {
            steps {
                script {
                    kubernetesDeploy(
                        configs: 'k8s/*.yaml',
                        kubeconfigId: 'k8s-config',
                        enableConfigSubstitution: true
                    )
                }
            }
        }
    }
}
```

This Jenkins pipeline builds the Docker image, pushes it to Docker Hub, and then deploys the updated application to Kubernetes using the Kubernetes plugin.

---

## Conclusion 🎉

By following the steps above, you have successfully set up **ArgoCD** with a **GitOps** workflow to deploy your application on Kubernetes. ArgoCD continuously monitors your Git repository and automatically applies changes to your Kubernetes cluster whenever changes are detected. Additionally, if you choose to integrate Jenkins, you can automate the entire process from building to deploying your application.

With this setup, your deployment process is now automated and version-controlled through Git!

---

Keep learning! 😊
