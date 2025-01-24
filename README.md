---

# **ArgoCD with GitOps: Deploying Applications 🚀**

## **What is ArgoCD?**

**ArgoCD** is a declarative, GitOps continuous delivery tool for Kubernetes. It helps automate the deployment and management of applications in Kubernetes by syncing the cluster state with a Git repository. In this setup, Git serves as the single source of truth, enabling teams to control application deployments through version-controlled Git repositories.

### **Key Features of ArgoCD:**
- **Declarative Configuration**: It deploys Kubernetes applications based on configurations in Git.
- **Real-time Feedback**: Provides live updates on application status and health.
- **Automated Rollback**: Easily revert applications to a previous state using Git commit history.

---

## **Why Use ArgoCD in a GitOps Workflow?**

In a **GitOps** workflow, **ArgoCD** enables automated Continuous Deployment (CD) through Git repositories. By linking the repository to ArgoCD, every change made in the Git repository triggers an automatic deployment in your Kubernetes cluster.

### **Benefits of Using ArgoCD with GitOps:**
- **Single Source of Truth**: Kubernetes configuration and manifests are stored in Git.
- **Version Control**: All deployment changes are version-controlled and auditable.
- **Collaboration**: Teams collaborate by working on the same configuration files.
- **Security and Auditability**: Changes are auditable, and deployments are tied to Git access control policies.

---

## **Steps to Deploy an Application using ArgoCD with GitOps**

In this guide, we walk through the process of deploying an application on Kubernetes using ArgoCD and GitOps.

---

### **Step 1: Install ArgoCD in Your Kubernetes Cluster 🛠️**

ArgoCD must be installed within the Kubernetes cluster to manage deployments. Here’s how we can install ArgoCD:

1. **Install ArgoCD via kubectl**:

   Run the following command to install ArgoCD into the `argocd` namespace.

   ```bash
   kubectl create namespace argocd
   kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
   ```

2. **Expose ArgoCD Server with LoadBalancer**:

   To access the ArgoCD UI, we exposed the ArgoCD service using a LoadBalancer.

   ```bash
   kubectl expose svc argocd-server -n argocd --type=LoadBalancer --name=argocd-server
   ```

   Once the service is exposed, you can access the ArgoCD UI using the external IP of the LoadBalancer.

---

### **Step 2: Access ArgoCD UI 🌐**

Once the service is exposed as a LoadBalancer, you can access ArgoCD using the external IP provided by the LoadBalancer.

1. Get the external IP:

   ```bash
   kubectl get svc -n argocd
   ```

2. Access the ArgoCD UI through the browser by navigating to `http://<External-IP>`.

3. **Login to ArgoCD UI**:
   - Username: `admin`
   - Password: The initial password is the name of the ArgoCD server pod, which can be retrieved using:
     ```bash
     kubectl get pods -n argocd
     ```

---

### **Step 3: Add Git Repository to ArgoCD 🚀**

ArgoCD communicates with Git repositories to sync application states. In our case, we have connected the repository using HTTPS and provided the necessary authentication details.

1. **Add Git Repository**:

   In the ArgoCD UI, navigate to **Settings** > **Repositories** > **Connect Repo**.

2. **Provide Repository Information**:
   - **Repository URL**: `https://github.com/vishnu-rv/argo.git` (Your Git repository URL)
   - **Username**: `Your GitHub Username`
   - **Password/Token**: Provide your GitHub **Personal Access Token (PAT)**.

This step allows ArgoCD to access and pull changes from the Git repository.

---

### **Step 4: Create an Application in ArgoCD 📦**

Now, we will create an ArgoCD application that will sync with our Git repository and deploy the application in Kubernetes.

1. **Go to ArgoCD UI** and click on **New App**.
2. **Fill in the Application Details**:
   - **Application Name**: `game`
   - **Project**: `default`
   - **Repository URL**: `https://github.com/vishnu-rv/argo.git`
   - **Revision**: `main` (or the branch you want to deploy from)
   - **Path**: `/` (or the path where `deployment.yaml` and `service.yaml` are located)
   - **Cluster**: Select your cluster.
   - **Namespace**: `game` (the namespace where you want to deploy).

After creating the application, ArgoCD will automatically sync the configuration from the Git repository and deploy the application.

---

### **Step 5: GitOps Workflow - Sync Changes Automatically 🔄**

ArgoCD will automatically deploy any changes made to the Git repository. Whenever changes are pushed to the Git repository (e.g., updating the Docker image version), ArgoCD will detect the change and deploy it to Kubernetes.

For example, to update the Docker image version:

1. Modify the `deployment.yaml` file in your Git repository:
   ```yaml
   image: vishnu2117/vishnuops-game:v2
   ```

2. Push the changes to the Git repository:
   ```bash
   git add .
   git commit -m "Update Docker image version to v2"
   git push
   ```

3. ArgoCD will detect the change and deploy the updated configuration to Kubernetes.

![image](https://github.com/user-attachments/assets/894176ce-a878-4053-9780-1d1d5f3105c5)


---

### **Step 6: Continuous Deployment with Jenkins 🤖**

To automate the deployment further, you can integrate **Jenkins** for Continuous Integration (CI). Jenkins can monitor changes in your Git repository and trigger builds and deployments automatically.

1. Create a **Jenkins Pipeline** in your repository. Here's an example `Jenkinsfile`:
   
   ```groovy
   pipeline {
       agent any
       stages {
           stage('Checkout') {
               steps {
                   git 'https://github.com/vishnu-rv/argo.git'
               }
           }
           stage('Build Docker Image') {
               steps {
                   script {
                       docker.build('vishnu2117/vishnuops-game:v1')
                   }
               }
           }
           stage('Push Image to Docker Hub') {
               steps {
                   script {
                       docker.withRegistry('', 'dockerhub-credentials') {
                           docker.image('vishnu2117/vishnuops-game:v1').push()
                       }
                   }
               }
           }
           stage('Deploy to Kubernetes') {
               steps {
                   script {
                       sh 'kubectl apply -f deployment.yaml -n game'
                   }
               }
           }
       }
   }
   ```

2. **Jenkins will automatically trigger the pipeline** on any code changes, build the Docker image, push it to Docker Hub, and deploy it to Kubernetes using the updated manifest.

---

## **Conclusion 🎉**

In this guide, we set up a full GitOps workflow using **ArgoCD** to deploy applications to Kubernetes, with a Git repository as the source of truth. We also showed how Jenkins can automate the deployment process through Continuous Integration.

### **Steps Covered**:
- Install ArgoCD in the Kubernetes cluster.
- Expose ArgoCD UI via a LoadBalancer.
- Add Git repository in ArgoCD and link it with the application.
- Sync the application automatically using Git commits.
- Automate deployment with Jenkins pipelines.

This setup ensures a streamlined and automated deployment pipeline that enhances collaboration, auditing, and security, all while being highly scalable and repeatable.

---

### **Future Enhancements**:
You can further enhance the pipeline by adding automated tests or additional steps like linting and security checks before deployment.

---

You can now add the screenshots wherever needed in this file! The guide should now be comprehensive and easy to understand for anyone using ArgoCD with GitOps.

