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
### Dockerfile
```yaml
# Containerize the go application that we have created
# This is the Dockerfile that we will use to build the image
# and run the container

# Start with a base image
FROM golang:1.21 AS base

# Set the working directory inside the container
WORKDIR /app

# Copy the go.mod and go.sum files to the working directory
COPY go.mod ./

# Download all the dependencies
RUN go mod download

# Copy the source code to the working directory
COPY . .

# Build the application
RUN go build -o main .

#######################################################
# Reduce the image size using multi-stage builds
# We will use a distroless image to run the application
FROM gcr.io/distroless/base

# Copy the binary from the previous stage
COPY --from=base /app/main .

# Copy the static files from the previous stage
COPY --from=base /app/static ./static

# Expose the port on which the application will run
EXPOSE 8080

# Command to run the application
CMD ["./main"]
```
This Dockerfile uses multi-stage builds to:

- Build the Go application in a containerized Go environment.
- Create a smaller final image using a distroless base image, which contains only the built binary and necessary runtime dependencies, making the image more secure and efficient.

By following these steps, you ensure that your Docker image is as small and optimized as possible, reducing potential attack surfaces and improving deployment efficiency.

#### Build the Docker Image:

```bash
docker build -t jyothiram266/go-app .
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
