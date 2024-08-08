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
# Go App Deployment on Kubernetes

This document provides the steps to deploy the Go App on a Kubernetes cluster.

## Deployment Manifest

The following YAML manifest creates a Kubernetes Deployment for the Go App:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: go-app
  labels:
    app: go-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: go-app
  template:
    metadata:
      labels:
        app: go-app
    spec:
      containers:
      - name: go-app
        image: jyothiram266/go-app
        ports:
        - containerPort: 8080
```
# Service Manifest
The following YAML manifest creates a Kubernetes Service to expose the Go App:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: go-app-service
spec:
  type: NodePort
  selector:
    app: go-app
  ports:
    - protocol: TCP
      port: 8080         # Port that the service will expose
      targetPort: 8080   # Port on the container that the service should forward traffic to
      nodePort: 30007
```
# Ingress Manifest
The following YAML manifest creates a Kubernetes Ingress to route traffic to the Go App:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: go-app
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx
  rules:
  - host: go-web.local
    http:
      paths: 
      - path: /
        pathType: Prefix
        backend:
          service:
            name: go-app-service
            port:
              number: 8080
```
Applying the Manifests
To apply the above manifests, save each to a separate file (deployment.yaml, service.yaml, ingress.yaml) and run the following commands:

```sh
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
kubectl apply -f ingress.yaml
```
Verifying the Deployment
After applying the manifests, you can verify the deployment and services with the following commands:

```sh
kubectl get deployments
kubectl get services
kubectl get ingress
```
This should show the go-app deployment running, the go-app-service service, and the go-app ingress.

Accessing the Application
To access the application, you need to configure your local DNS to resolve go-web.local to the Ingress controller's public IP. You can find the public IP address of the Ingress controller with:

```sh
kubectl get services -o wide -w -n ingress-nginx
```
Find the external IP address for the ingress-nginx-controller service and add an entry to your /etc/hosts file (or the equivalent on your operating system):

```sh
<external-ip-address> go-web.local
```
Then, you should be able to access your Go App by navigating to http://go-web.local in your web browser.

Sure, let's start from the very beginning and include the command to create a Helm chart using `helm create`.

### Steps to Create and Use the Helm Chart

1\. **Create the Helm chart:**

   Use the `helm create` command to scaffold a new Helm chart:
   ```sh
   helm create go-app
   ```

   This command creates a directory named `go-app` with a default Helm chart structure.

2\. **Update the Helm chart:**

   Replace the default files with your specific configuration.

   Directory Structure

   Ensure your directory structure looks like this:

   
   
  

3\. **Modify `Chart.yaml`:**

   Edit the `Chart.yaml` file with the following content:

    apiVersion: v2
    name: go-app-chart
    description: A Helm chart for Kubernetes
    
    # A chart can be either an 'application' or a 'library' chart.
    #
    # Application charts are a collection of templates that can be packaged into versioned archives
    # to be deployed.
    #
    # Library charts provide useful utilities or functions for the chart developer. They're included as
    # a dependency of application charts to inject those utilities and functions into the rendering
    # pipeline. Library charts do not define any templates and therefore cannot be deployed.
    type: application
    
    # This is the chart version. This version number should be incremented each time you make changes
    # to the chart and its templates, including the app version.
    # Versions are expected to follow Semantic Versioning (https://semver.org/)
    version: 0.1.0
    
    # This is the version number of the application being deployed. This version number should be
    # incremented each time you make changes to the application. Versions are not expected to
    # follow Semantic Versioning. They should reflect the version the application is using.
    # It is recommended to use it with quotes.
    appVersion: "1.16.0"
     
4\. **Modify `values.yaml`:**

   Update the `values.yaml` file with the following content:

   ```yaml 
       
      replicaCount: 1
      
      image:
        repository: jyothiram266/go-app
        pullPolicy: IfNotPresent
        # Overrides the image tag whose default is the chart appVersion.
        tag: "v1"
      
      ingress:
        enabled: false
        className: ""
        annotations: {}
          # kubernetes.io/ingress.class: nginx
          # kubernetes.io/tls-acme: "true"
        hosts:
          - host: chart-example.local
            paths:
              - path: /
                pathType: ImplementationSpecific
   ```

5\. **Modify `templates/deployment.yaml`:**

   Update the `templates/deployment.yaml` file with the following content:

   ```yaml

      # This is a sample deployment manifest file for a simple web application.
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: go-app
      labels:
        app: go-app
    spec:
      replicas: 1
      selector:
        matchLabels:
          app: go-app
      template:
        metadata:
          labels:
            app: go-app
        spec:
          containers:
          - name: go-app
            image: jyothiram266/go-app:{{ .Values.image.tag }}
            ports:
            - containerPort: 8080
   ```

6\. **Modify `templates/service.yaml`:**

   Update the `templates/service.yaml` file with the following content:

   ```yaml
    apiVersion: v1
    kind: Service
    metadata:
      name: go-app-service
    spec:
      type: NodePort
      selector:
        app: go-app
      ports:
        - protocol: TCP
          port: 8080         # Port that the service will expose
          targetPort: 8080   # Port on the container that the service should forward traffic to
          nodePort: 30007  
       ```

7\. **Modify `templates/ingress.yaml`:**

   Update the `templates/ingress.yaml` file with the following content:

   ```yaml
  apiVersion: networking.k8s.io/v1
  kind: Ingress
  metadata:
    name: go-app
    annotations:
      nginx.ingress.kubernetes.io/rewrite-target: /
  spec:
    ingressClassName: nginx
    rules:
    - host: go-web.local
      http:
        paths: 
        - path: /
          pathType: Prefix
          backend:
            service:
              name: go-app-service
              port:
                number: 8080
   ```

8\. **Package the Helm chart:**

   Package your Helm chart into a tar.gz file:

   ```sh
   helm package go-app
   ```

9\. **Install the Helm chart:**

   Install the chart to your Kubernetes cluster:

   helm install my-go-app go-app-0.1.0.tgz
   ```

   Replace `my-go-app` with your desired release name.

10\. **Verify the deployment:**

    Verify that the deployment, service, and ingress have been created successfully:

    ```sh
     kubectl get deployments
     kubectl get services
     kubectl get ingress
    ```

This process will allow you to create and deploy your application using Helm, with configurable parameters in the `values.yaml` file. Adjust any configurations as needed for your specific use case.

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
