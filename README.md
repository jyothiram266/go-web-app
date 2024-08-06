# 
### Overview
This project demonstrates the deployment of a scalable Go application using Docker, Kubernetes, Helm, ArgoCD, and GitHub Actions.

### Features
- Optimized Containerization: Utilizes Docker multi-stage builds to reduce image size and enhance deployment efficiency.
- Kubernetes Deployment: Manages the application on AWS EKS for high availability and scalability.
- Helm Charts: Simplifies deployment and configuration management within the Kubernetes cluster.
- CI/CD Automation: Implements continuous deployment with ArgoCD and robust CI pipelines using GitHub Actions.
- Traffic Management: Uses Nginx Ingress Controller for efficient traffic routing and secure access to services.

### Prerequisites
- Docker
- Kubernetes
- AWS EKS
- Helm
- ArgoCD
- GitHub Actions
- Installation

#### Clone the Repository:
```
git clone https://github.com/jyothiram266/go-web-app
cd go-web-app
```
Build the Docker Image:

```bash
docker build -t jyothiram266/ .
```

Deploy to Kubernetes:

Helm Installation:

helm install your-release-name ./helm-chart-directory
Apply Kubernetes Manifests:
bash
Copy code
kubectl apply -f ./k8s
Set Up ArgoCD for Continuous Deployment:

Install ArgoCD:
bash
Copy code
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
Connect Your Repository:
Follow ArgoCD documentation to connect your GitHub repository.
Configure GitHub Actions:

Ensure your repository has the necessary GitHub Actions workflows in .github/workflows.
Usage
Access the Application:

Find the Nginx Ingress Controller IP:
bash
Copy code
kubectl get ingress
Open the IP address in your browser.
Monitor Deployment:

Use ArgoCD UI or CLI to monitor and manage deployments.
Contributing
Feel free to open issues or submit pull requests for any enhancements or bug fixes.
