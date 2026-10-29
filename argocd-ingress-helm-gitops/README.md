# Argo CD + Helm + Kubernetes Ingress | GitOps Project

## Project Overview

This project demonstrates how to deploy and manage Kubernetes applications using **GitOps with Argo CD**, install an NGINX Ingress Controller using a Helm chart, and expose three different static websites through path-based routing.

The Kubernetes configuration files are maintained in GitHub. Argo CD monitors the repository and synchronizes the desired state with the Kubernetes cluster.

## Objectives

- Deploy an NGINX Ingress Controller using Helm through Argo CD.
- Deploy three separate website applications using Kubernetes Deployments.
- Create ConfigMaps to provide unique HTML content for each website.
- Expose the applications through ClusterIP Services.
- Configure path-based routing using a Kubernetes Ingress resource.
- Demonstrate GitOps by changing website content in GitHub and verifying the update in the running application.

## Architecture

```text
                 GitHub Repository
                         |
                         v
                   Argo CD Root App
                         |
              +----------+----------+
              |                     |
              v                     v
       ingress-nginx          three-websites
              |                     |
              v                     v
       Helm Chart Install     Kubernetes Resources
              |                     |
              v                     |
       NGINX Ingress Controller    |
              |                     |
              +----------+----------+
                         |
                         v
                AWS Load Balancer
                         |
             +-----------+-----------+
             |           |           |
           /app1       /app2       /app3
             |           |           |
             v           v           v
        website1-svc website2-svc website3-svc
             |           |           |
             v           v           v
         Website 1   Website 2   Website 3
```

## Environment

| Component | Configuration |
|---|---|
| Cloud platform | AWS |
| Kubernetes distribution | KOPS |
| Kubernetes version | v1.36.5 |
| Cluster nodes | One control-plane node and one worker node |
| Git repository | k8s-handon-repo |
| GitOps tool | Argo CD |
| Package manager | Helm |
| Ingress controller | NGINX Ingress Controller |
| Website container image | nginx:stable |
| Website namespace | demo-web |
| Argo CD namespace | argocdtask |

## Repository Structure

```text
argocd-ingress-helm-gitops/
├── README.md
├── argocd/
│   ├── root-app.yaml
│   └── apps/
│       ├── ingress-nginx.yaml
│       └── websites.yaml
└── k8s/
    └── manifests/
        ├── configmaps.yaml
        ├── deployments.yaml
        ├── services.yaml
        └── ingress.yaml
```

## Implementation

### 1. Kubernetes Cluster Preparation

Created a KOPS cluster with one control-plane node and one worker node.

Added a worker-node label to control application placement:

```bash
kubectl label node <worker-node-name> workload=worker
kubectl get nodes -L workload
```

The website Deployments and Ingress Controller use the `workload=worker` node selector.

### 2. Argo CD Configuration

Installed Argo CD in the `argocdtask` namespace and configured the root Application to monitor the GitHub repository.

The root Application discovers two child Applications:

- `ingress-nginx`: Installs the Ingress Controller through a Helm chart.
- `three-websites`: Deploys the website resources from the repository.

### 3. Helm-Based Ingress Controller

Configured `ingress-nginx.yaml` as an Argo CD Application that references the official NGINX Ingress Helm chart.

The controller uses a LoadBalancer Service to provide external access through AWS, subject to the cluster's cloud-provider and networking configuration.

### 4. Website Deployments

Created three Kubernetes Deployments:

- `website1`
- `website2`
- `website3`

Each Deployment runs one NGINX Pod and mounts its own ConfigMap containing a unique `index.html` page.

### 5. ConfigMaps and Services

Created three ConfigMaps:

- `website1-content`
- `website2-content`
- `website3-content`

Created three ClusterIP Services:

- `website1-svc`
- `website2-svc`
- `website3-svc`

Each Service selects the corresponding application's Pods and forwards traffic to container port 80.

### 6. Path-Based Ingress Routing

Configured the `websites-ingress` resource with three paths:

| Path | Backend Service | Website |
|---|---|---|
| `/app1` | `website1-svc` | Website 1 |
| `/app2` | `website2-svc` | Website 2 |
| `/app3` | `website3-svc` | Website 3 |

The NGINX rewrite annotation removes the matching path prefix before forwarding requests to the website container.

## Deployment Workflow

1. Created the Kubernetes and Argo CD YAML files locally.
2. Committed and pushed the project folder to GitHub.
3. Applied the root Application once to bootstrap the GitOps workflow.
4. Argo CD discovered the child Applications.
5. The Ingress Controller Helm chart and website resources were synchronized.
6. AWS provisioned an external LoadBalancer endpoint.
7. Validated the three website paths using HTTP requests.
8. Edited Website 2's HTML content directly in GitHub and verified the updated page.

## Useful Commands

### Verify cluster nodes

```bash
kubectl get nodes -o wide
kubectl get nodes -L workload
```

### Check Argo CD Applications

```bash
kubectl get applications -n argocdtask
```

### Inspect the Ingress Controller

```bash
kubectl get pods -n ingress-nginx -o wide
kubectl get svc -n ingress-nginx
```

### Inspect website resources

```bash
kubectl get deployments,pods,svc,ingress -n demo-web -o wide
```

### Bootstrap the root Application

Run once after confirming that the project folder is available in the configured GitHub repository:

```bash
kubectl apply -f argocd/root-app.yaml
```

### Test the website endpoints

Replace `<INGRESS_HOST>` with the external hostname shown by your Ingress resource:

```bash
curl -i http://<INGRESS_HOST>/app1
curl -i http://<INGRESS_HOST>/app2
curl -i http://<INGRESS_HOST>/app3
```

## Validation Results

The following results were observed during testing:

- All three Argo CD Applications reached `Synced` and `Healthy` status.
- The NGINX Ingress Controller Pod was running on the worker node.
- All three website Deployments had one available replica each.
- All three website Pods were running on the worker node.
- The Ingress resource contained all three routing paths.
- HTTP requests to `/app1`, `/app2`, and `/app3` returned `200 OK`.
- A direct GitHub edit to Website 2 was synchronized and the updated content was displayed.

## Key Learnings

- GitOps principles and declarative Kubernetes management.
- Argo CD root and child Application configuration.
- Deploying Helm charts through Argo CD.
- Kubernetes Deployments, Services, ConfigMaps, and Ingress.
- Path-based routing and URL rewriting.
- Worker-node scheduling using node selectors.
- Troubleshooting Kubernetes RBAC and Argo CD synchronization.
- Validating application behavior through HTTP requests.
- Testing configuration changes through a GitHub-driven workflow.

## Screenshots

Add your screenshots to a folder such as `screenshots/` and reference them here.

Suggested evidence:

1. Argo CD Applications showing `Synced` and `Healthy`.
2. Kubernetes Pods and Services running on the worker node.
3. Website 1 response through `/app1`.
4. Website 2 response through `/app2`.
5. Website 3 response through `/app3`.
6. Updated Website 2 after the GitHub change.

Example Markdown:

```markdown
![Argo CD Applications](screenshots/argocd-applications.png)
![Website 1](screenshots/website1.png)
![Website 2 GitOps Update](screenshots/website2-gitops-update.png)
![Website 3](screenshots/website3.png)
```

## Repository

GitHub: https://github.com/pranitha-cloud-portfolio/k8s-handon-repo

Project directory: `argocd-ingress-helm-gitops`

## Notes

This is a hands-on learning project. The NGINX Ingress Controller project has been retired, so this implementation should not be considered a production-ready deployment. For new production environments, evaluate a maintained ingress solution and verify compatibility with the Kubernetes version in use.

TLS, authentication, and production security hardening are outside the scope of this lab.
