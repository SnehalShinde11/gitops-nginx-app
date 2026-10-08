# Project 3.4 - GitOps Application Deployment using ArgoCD on Kubernetes

## 1. Project Overview

This project demonstrates the deployment and management of a containerized Nginx application on Kubernetes using the **GitOps methodology** with **ArgoCD**.

The Kubernetes application configuration is maintained in a GitHub repository. ArgoCD continuously monitors the Git repository and automatically synchronizes changes to the Kubernetes cluster.

The project uses:

- Google Kubernetes Engine (GKE)
- Kubernetes
- ArgoCD
- GitHub
- Nginx
- Kubernetes Deployment
- Kubernetes Service
- GitOps-based synchronization
- Automatic synchronization and self-healing

---

## 2. Problem Statement

Manually deploying applications to Kubernetes using `kubectl apply` can become difficult to manage as application complexity and the number of environments increase.

Manual deployments can lead to:

- Configuration drift
- Inconsistent deployments
- Difficult rollback processes
- Lack of centralized deployment history
- Manual intervention for application updates
- Kubernetes state differing from the desired configuration

The objective of this project is to implement a **GitOps-based deployment model** where Git acts as the source of truth for the Kubernetes application.

---

## 3. Solution Approach

The project follows this GitOps workflow:

```text
Developer
    |
    | Push Kubernetes manifests
    v
GitHub Repository
    |
    | ArgoCD monitors repository
    v
ArgoCD
    |
    | Automatic Sync
    v
Kubernetes / GKE
    |
    v
Nginx Application
```

Whenever the Kubernetes manifests are modified and pushed to GitHub:

1. ArgoCD detects the change.
2. ArgoCD compares the Git state with the Kubernetes cluster.
3. ArgoCD automatically synchronizes the changes.
4. Kubernetes resources are updated.
5. Application health can be monitored from the ArgoCD dashboard.

---

## 4. Application Details

The application used in this project is an Nginx web application.

| Configuration | Value |
|---|---|
| Application | Nginx |
| Docker Image | `nginx:1.27-alpine` |
| Initial Replicas | 2 |
| Kubernetes Deployment | `gitops-nginx` |
| Kubernetes Service | `gitops-nginx-service` |
| Application Namespace | `gitops-nginx` |
| ArgoCD Application | `gitops-nginx` |
| Git Branch | `master` |
| Manifest Directory | `k8s` |

The existing Kubernetes manifests are maintained inside the `k8s` directory of the GitHub repository.

---

## 5. Repository Structure

The project repository already contains the required Kubernetes manifests.

```text
gitops-nginx-app/
└── k8s/
    ├── deployment.yaml
    └── service.yaml
```

The Kubernetes manifests are consumed directly by ArgoCD.

> **Note:** The project directory and Kubernetes manifests already exist. Do not create another project directory or duplicate the existing files.

---

## 6. Dependencies

The following components are required:

- Google Cloud Platform account
- Google Kubernetes Engine (GKE)
- Google Cloud Shell or local machine with `gcloud`
- `kubectl`
- Git
- GitHub repository
- ArgoCD
- Nginx container image

### Verify Required Tools

```bash
gcloud version
kubectl version --client
git --version
```

---

# 7. Execution Steps

# Task 1 - Create Kubernetes Cluster

Create a Kubernetes cluster using **Google Kubernetes Engine (GKE)**.

The cluster can be created from the Google Cloud Console or using the `gcloud` CLI.

## Subtask 1.1 - Create GKE Cluster

### General Syntax

```bash
gcloud container clusters create <CLUSTER_NAME> \
  --zone <ZONE> \
  --machine-type <MACHINE_TYPE> \
  --num-nodes <NUMBER_OF_NODES>
```

### Project Configuration

Create the GKE cluster according to the assignment requirements.

---

## Subtask 1.2 - Configure kubectl

After creating the cluster, obtain its credentials:

```bash
gcloud container clusters get-credentials <CLUSTER_NAME> \
  --zone <ZONE> \
  --project <PROJECT_ID>
```

This configures `kubectl` to communicate with the GKE cluster.

---

## Subtask 1.3 - Verify Kubernetes Cluster

```bash
kubectl cluster-info
kubectl get nodes
```

The nodes should show the `Ready` status.

---

# Task 2 - Install ArgoCD

## Subtask 2.1 - Create ArgoCD Namespace

```bash
kubectl create namespace argocd
```

Verify:

```bash
kubectl get namespace argocd
```

Expected:

```text
NAME      STATUS
argocd    Active
```

---

## Subtask 2.2 - Install ArgoCD

Install ArgoCD using the official ArgoCD installation manifest:

```bash
kubectl apply -n argocd \
  -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

This creates the required ArgoCD components inside the `argocd` namespace.

---

## Subtask 2.3 - Verify ArgoCD Pods

```bash
kubectl get pods -n argocd
```

Wait until the ArgoCD components reach their healthy state.

You can also check all ArgoCD resources:

```bash
kubectl get all -n argocd
```

---

## Subtask 2.4 - Retrieve ArgoCD Initial Admin Password

Check the ArgoCD initial admin secret:

```bash
kubectl get secret argocd-initial-admin-secret \
  -n argocd \
  -o yaml
```

The password is stored as a Base64-encoded value.

Decode it using:

```bash
echo '<base64-encoded-value>' | base64 -d
```

Alternatively:

```bash
kubectl -n argocd get secret argocd-initial-admin-secret \
  -o jsonpath='{.data.password}' | base64 -d
```

ArgoCD login credentials:

```text
Username: admin
Password: <ArgoCD initial admin password>
```

> **Security:** Never commit the ArgoCD admin password or Kubernetes secrets to the Git repository.

---

# Task 2.1 - Access ArgoCD Dashboard

## Subtask 2.1.1 - Check ArgoCD Server Service

```bash
kubectl get svc argocd-server -n argocd
```

The ArgoCD server service exposes the required HTTP/HTTPS ports.

---

## Subtask 2.1.2 - Port Forward ArgoCD Server

```bash
kubectl port-forward svc/argocd-server \
  -n argocd \
  8080:80
```

The local port `8080` is forwarded to the ArgoCD server.

---

## Subtask 2.1.3 - Open ArgoCD Using Cloud Shell Web Preview

In Google Cloud Shell:

1. Start the port-forward command.
2. Open **Web Preview**.
3. Select **Change port**.
4. Enter `8080`.
5. Open the preview.

Log in using:

```text
Username: admin
Password: <ArgoCD initial admin password>
```

---

# Task 3 - Prepare Git Repository

The GitHub repository for this project has already been created and contains the required Kubernetes manifests.

Repository:

```text
gitops-nginx-app
```

The Kubernetes manifests are maintained under:

```text
k8s/
```

The repository uses:

```text
master
```

> **Important:** The project directory and Kubernetes manifests already exist. Do not create another project directory or duplicate the existing files.

---

## Subtask 3.1 - Navigate to the Existing Project

```bash
cd ~/kubernetes-web-app/project-3.4-gitops-nginx

pwd
ls
ls k8s/
```

Verify that the existing project and `k8s` directory are present.

---

## Subtask 3.2 - Verify Git Repository

```bash
git status
git remote -v
git branch
```

Verify that the repository points to the GitHub repository and that the current branch is `master`.

---

## Subtask 3.3 - Verify Existing Kubernetes Manifests

```bash
ls k8s/
```

The existing application uses:

- Nginx image `nginx:1.27-alpine`
- Initial replica count of 2
- Kubernetes Deployment
- Kubernetes Service

---

## Subtask 3.4 - Review Existing Manifests

Review the existing Deployment:

```bash
cat k8s/deployment.yaml
```

Review the existing Service:

```bash
cat k8s/service.yaml
```

Do not recreate these files if they already exist.

---

## Subtask 3.5 - Commit and Push Changes Only When Required

If changes are required in the existing manifests:

```bash
git status
git diff

git add k8s/
git commit -m "<commit-message>"
git push origin master
```

If no changes are required, no additional commit is necessary.

---

# Task 4 - Configure ArgoCD Application

The Nginx application will be configured from the **ArgoCD web interface**.

Do not manually deploy the application using `kubectl apply`. ArgoCD should manage the Kubernetes resources from Git.

## Subtask 4.1 - Create a New ArgoCD Application

From the ArgoCD dashboard:

```text
Applications → + NEW APP
```

Configure the application as follows.

### General Configuration

| Field | Value |
|---|---|
| Application Name | `gitops-nginx` |
| Project Name | `default` |
| Sync Policy | `Automatic` |

Enable:

- Prune Resources
- Self Heal

---

## Subtask 4.2 - Configure Application Source

Configure the Git repository as the application source.

| Field | Value |
|---|---|
| Repository | `gitops-nginx-app` |
| Revision | `master` |
| Path | `k8s` |

The `k8s` directory contains the Kubernetes manifests that ArgoCD will deploy.

---

## Subtask 4.3 - Configure Application Destination

Configure:

| Field | Value |
|---|---|
| Cluster URL | `https://kubernetes.default.svc` |
| Namespace | `gitops-nginx` |

If available, enable:

```text
Create Namespace
```

or the corresponding:

```text
Auto Create Namespace
```

sync option.

---

## Subtask 4.4 - Create the ArgoCD Application

Click:

```text
Create
```

ArgoCD will begin synchronizing the Kubernetes resources from Git.

Do **not** run:

```bash
kubectl apply -f k8s/
```

because ArgoCD is responsible for deploying the application.

---

# Task 4.1 - Verify ArgoCD Application

## Subtask 4.1.1 - Verify ArgoCD Application Resource

```bash
kubectl get application gitops-nginx -n argocd
```

The application should eventually show a healthy and synchronized state.

---

## Subtask 4.1.2 - Verify Kubernetes Resources

```bash
kubectl get all -n gitops-nginx
```

Verify pods:

```bash
kubectl get pods -n gitops-nginx
```

Verify Deployment:

```bash
kubectl get deployment gitops-nginx -n gitops-nginx
```

Verify Service:

```bash
kubectl get service -n gitops-nginx
```

---

# Task 4.2 - Verify Nginx Application

## Subtask 4.2.1 - Retrieve LoadBalancer IP

If the Service is configured as a `LoadBalancer`:

```bash
kubectl get service -n gitops-nginx
```

Wait until an `EXTERNAL-IP` is assigned.

---

## Subtask 4.2.2 - Test Nginx Application

Use the assigned LoadBalancer IP:

```bash
curl http://<LOAD_BALANCER_IP>
```

The response should contain the Nginx web page.

> The LoadBalancer IP is environment-specific and should not be hard-coded in the README.

---

# Task 5 - Deploy Application Using GitOps

The application deployment is controlled through Git.

The Git repository acts as the **source of truth**.

The deployment flow is:

```text
Change Kubernetes Manifest
          |
          v
      Git Commit
          |
          v
      Git Push
          |
          v
      GitHub
          |
          v
       ArgoCD
          |
          v
   Automatic Sync
          |
          v
    Kubernetes
```

## Subtask 5.1 - Modify Application Configuration

Make the required changes to the existing Kubernetes manifest in the Git repository.

Do not create duplicate Kubernetes manifests.

---

## Subtask 5.2 - Commit and Push Changes

Commit and push the changes to the `master` branch.

ArgoCD monitors the configured repository and detects changes pushed to the configured revision.

---

## Subtask 5.3 - Verify Automatic Deployment

```bash
kubectl get application gitops-nginx -n argocd
```

ArgoCD should detect the change and synchronize the application automatically.

---

# Task 6 - Verify Deployment

## Subtask 6.1 - Verify ArgoCD Dashboard

Open the ArgoCD dashboard and verify:

```text
Application: gitops-nginx
Sync Status: Synced
Health Status: Healthy
```

The resource tree should display the Kubernetes resources managed by ArgoCD.

---

## Subtask 6.2 - Verify Kubernetes Resources

```bash
kubectl get all -n gitops-nginx
```

Verify pods:

```bash
kubectl get pods -n gitops-nginx
```

Verify Deployment:

```bash
kubectl get deployment gitops-nginx -n gitops-nginx
```

Verify Service:

```bash
kubectl get service gitops-nginx-service -n gitops-nginx
```

---

# Task 7 - Test GitOps Synchronization

This task demonstrates that changes made in Git are automatically synchronized to Kubernetes by ArgoCD.

## Subtask 7.1 - Check Current Replica Count

```bash
kubectl get deployment gitops-nginx -n gitops-nginx
```

Initially, the Deployment should have:

```text
2/2
```

replicas available.

---

## Subtask 7.2 - Modify Replica Count in Git

Modify the existing:

```text
k8s/deployment.yaml
```

Change:

```yaml
replicas: 2
```

to:

```yaml
replicas: 3
```

Do not create another Deployment file.

---

## Subtask 7.3 - Commit and Push the Change

```bash
git add k8s/deployment.yaml

git status

git commit -m "Scale GitOps Nginx to 3 replicas"

git push origin master
```

---

## Subtask 7.4 - Check ArgoCD Synchronization

```bash
kubectl get application gitops-nginx -n argocd
```

ArgoCD should detect the new Git commit and synchronize the application.

---

## Subtask 7.5 - Verify Deployment Scaling

```bash
kubectl get deployment gitops-nginx -n gitops-nginx
```

The desired replica count should become:

```text
3
```

Verify the pods:

```bash
kubectl get pods -n gitops-nginx
```

Three Nginx pods should eventually be running.

---

## Subtask 7.6 - Verify Application Health

```bash
kubectl get application gitops-nginx -n argocd
```

The application should return to:

```text
Synced
Healthy
```

---

## Subtask 7.7 - Review Command History

```bash
history
```

This can be used to review the commands executed during the deployment and synchronization process.

---

## Subtask 7.8 - Verify Automatic Sync Configuration

```bash
kubectl get application gitops-nginx \
  -n argocd \
  -o yaml
```

Look for:

```yaml
syncPolicy:
  automated:
```

This confirms that ArgoCD is configured for automatic synchronization.

---

# Task 8 - GitOps Rollback

A key advantage of GitOps is that application changes can be rolled back through Git.

Instead of using:

```bash
kubectl rollout undo
```

the GitOps approach is to revert the Git commit and allow ArgoCD to synchronize the reverted configuration.

The rollback flow is:

```text
Git Commit
    |
    v
Git Revert
    |
    v
Git Push
    |
    v
ArgoCD Detects Change
    |
    v
Automatic Synchronization
    |
    v
Kubernetes Rolled Back
```

## Subtask 8.1 - Check Current Deployment

```bash
kubectl get deployment gitops-nginx -n gitops-nginx
```

---

## Subtask 8.2 - Review Git History

```bash
git log --oneline -2
```

Identify the commit that introduced the replica change.

---

## Subtask 8.3 - Navigate to Project

```bash
cd ~/kubernetes-web-app/project-3.4-gitops-nginx
```

---

## Subtask 8.4 - Revert the Git Commit

Use the commit ID that introduced the scaling change:

```bash
git revert <COMMIT_ID>
```

For example:

```bash
git revert 5f6c8ef
```

The revert restores the previous desired configuration in Git.

---

## Subtask 8.5 - Verify Manifest

```bash
grep replicas k8s/deployment.yaml
```

The replica count should return to the previous desired value.

---

## Subtask 8.6 - Push the Revert

```bash
git push origin master
```

ArgoCD detects the new Git commit and automatically synchronizes the cluster.

---

## Subtask 8.7 - Verify Rollback

Check the ArgoCD application:

```bash
kubectl get application gitops-nginx -n argocd
```

Check the Deployment:

```bash
kubectl get deployment gitops-nginx -n gitops-nginx
```

Check the pods:

```bash
kubectl get pods -n gitops-nginx
```

The Kubernetes state should now match the reverted Git configuration.

---

# Task 9 - Deploy Multiple Applications

GitOps can be extended to manage multiple Kubernetes applications.

The project includes the existing `k8s-demo/` path for demonstrating an additional application configuration.

## Subtask 9.1 - Verify Existing Configuration

```bash
git status
```

Verify that the existing `k8s-demo/` configuration is ready to be committed.

---

## Subtask 9.2 - Commit and Push the Second Application

If the second application's existing manifests are ready to be committed:

```bash
git add k8s-demo/

git commit -m "Add second GitOps Nginx application"

git push origin master
```

---

## Subtask 9.3 - Configure the Second Application in ArgoCD

Create another ArgoCD Application through the ArgoCD dashboard.

The second application can use:

- A separate Git path
- A separate Kubernetes namespace
- A separate ArgoCD Application
- Its own synchronization status
- Its own health status

> Do not invent or recreate files inside `k8s-demo/`. Use the existing repository configuration.

---

# Task 10 - Monitor Application Health

Application health can be monitored from the ArgoCD dashboard.

## Subtask 10.1 - Monitor ArgoCD Application

The ArgoCD dashboard provides visibility into:

- Application synchronization status
- Application health
- Kubernetes resources
- Deployment status
- Pod status
- Service status
- Git revision
- Synchronization history

The application should ideally show:

```text
Sync Status: Synced
Health Status: Healthy
```

---

## Subtask 10.2 - Monitor Kubernetes Resources

```bash
kubectl get all -n gitops-nginx
```

Pod monitoring:

```bash
kubectl get pods -n gitops-nginx
```

Deployment monitoring:

```bash
kubectl get deployment gitops-nginx -n gitops-nginx
```

Service monitoring:

```bash
kubectl get service -n gitops-nginx
```

---

# 11. Useful Kubernetes and ArgoCD Commands

### Check Cluster

```bash
kubectl cluster-info
kubectl get nodes
```

### Check ArgoCD

```bash
kubectl get pods -n argocd
kubectl get svc -n argocd
```

### Check ArgoCD Application

```bash
kubectl get application gitops-nginx -n argocd
```

### Check Application Resources

```bash
kubectl get all -n gitops-nginx
```

### Check Pods

```bash
kubectl get pods -n gitops-nginx
```

### Check Deployment

```bash
kubectl get deployment gitops-nginx -n gitops-nginx
```

### Check Service

```bash
kubectl get service -n gitops-nginx
```

### Check Detailed ArgoCD Application Configuration

```bash
kubectl get application gitops-nginx \
  -n argocd \
  -o yaml
```

---

# 12. GitOps Resource Flow

The complete resource flow implemented in this project is:

```text
                    GitHub
              gitops-nginx-app
                     |
                     |
              k8s/deployment.yaml
              k8s/service.yaml
                     |
                     v
                  ArgoCD
             gitops-nginx App
                     |
          Automatic Synchronization
                     |
                     v
              GKE Kubernetes
                     |
          +----------+----------+
          |                     |
          v                     v
     Deployment              Service
     gitops-nginx       gitops-nginx-service
          |
          v
       Nginx Pods
```

---

# 13. Key Concepts Demonstrated

### GitOps

Git is used as the source of truth for Kubernetes application configuration.

### ArgoCD

ArgoCD continuously monitors the Git repository and synchronizes the desired configuration with Kubernetes.

### Continuous Synchronization

Changes pushed to Git can automatically be deployed to the Kubernetes cluster.

### Automated Sync

ArgoCD automatically applies changes without requiring manual `kubectl apply`.

### Self-Healing

ArgoCD can detect differences between the desired Git state and the Kubernetes state and restore the desired configuration.

### Pruning

Resources removed from the Git-managed configuration can be removed from the Kubernetes application when pruning is enabled.

### Kubernetes Deployment

The Nginx application is deployed using a Kubernetes Deployment.

### Kubernetes Service

A Kubernetes Service exposes the Nginx application.

### Git-Based Rollback

Application configuration can be rolled back by reverting a Git commit and allowing ArgoCD to synchronize the reverted state.

### Multiple Application Management

ArgoCD can manage multiple Kubernetes applications from different Git paths and namespaces.

---

# 14. Project Details

| Detail | Information |
|---|---|
| Name | Snehal Shinde |
| Project | Project 3.4 |
| Assignment | GitOps Application Deployment using ArgoCD on Kubernetes |
| GitHub Repository | `gitops-nginx-app` |
| Kubernetes Platform | Google Kubernetes Engine (GKE) |
| GitOps Tool | ArgoCD |
| Application | Nginx |
| Docker Image | `nginx:1.27-alpine` |
| Git Branch | `master` |
| Manifest Directory | `k8s` |
| ArgoCD Application | `gitops-nginx` |
| Application Namespace | `gitops-nginx` |
