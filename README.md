# 
### Overview
This project demonstrates the deployment of a scalable Go application using Docker, Kubernetes, Helm, ArgoCD, and GitHub Actions.
![1](https://raw.githubusercontent.com/jyothiram266/go-web-app/master/screenshots/Screenshot%20from%202024-08-08%2018-58-01.png)
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
# Install EKS

Please follow the prerequisites doc before this.

## Install a EKS cluster with EKSCTL

```
eksctl create cluster --name demo-cluster --region us-east-1 
```
![2](https://github.com/jyothiram266/go-web-app/blob/master/screenshots/Screenshot%20from%202024-08-04%2012-11-24.png?raw=true)
## Delete the cluster

```
eksctl delete cluster --name demo-cluster --region us-east-1
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
# Install Nginx Ingress Controller on AWS

## Step 1: Deploy the below manifest

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.11.1/deploy/static/provider/aws/deploy.yaml
```
![6](https://github.com/jyothiram266/go-web-app/blob/master/screenshots/Screenshot%20from%202024-08-04%2012-59-32.png?raw=true)
Applying the Manifests
To apply the above manifests, save each to a separate file (deployment.yaml, service.yaml, ingress.yaml) and run the following commands:

```sh
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
kubectl apply -f ingress.yaml
```
![3](https://github.com/jyothiram266/go-web-app/blob/master/screenshots/Screenshot%20from%202024-08-04%2012-32-29.png?raw=true)

After applying the manifests, you can verify the deployment and services with the following commands:

```sh
kubectl get all
kubectl get ingress
```
This should show the go-app deployment running, the go-app-service service, and the go-app ingress.
![5](https://github.com/jyothiram266/go-web-app/blob/master/screenshots/Screenshot%20from%202024-08-04%2014-29-49.png?raw=true)

Accessing the Application
To access the application, you need to configure your local DNS to resolve go-web.local to the Ingress controller's public IP. You can find the public IP address of the Ingress controller with:

```sh
kubectl get services -o wide -w -n ingress-nginx
```
![4](https://github.com/jyothiram266/go-web-app/blob/master/screenshots/Screenshot%20from%202024-08-04%2012-32-42.png?raw=true)

Find the external IP address for the ingress-nginx-controller service and add an entry to your /etc/hosts file (or the equivalent on your operating system):
```sh
<external-ip-address> go-web.local
```
![7](https://github.com/jyothiram266/go-web-app/blob/master/screenshots/Screenshot%20from%202024-08-04%2014-41-38.png?raw=true)

Then, you should be able to access your Go App by navigating to http://go-web.local in your web browser.

![8](https://github.com/jyothiram266/go-web-app/blob/master/screenshots/Screenshot%20from%202024-08-04%2013-17-12.png?raw=true)
![9](https://github.com/jyothiram266/go-web-app/blob/master/screenshots/Screenshot%20from%202024-08-04%2013-17-17.png)
![10](https://github.com/jyothiram266/go-web-app/blob/master/screenshots/Screenshot%20from%202024-08-04%2013-17-22.png)

Sure, let's start from the very beginning and include the command to create a Helm chart using `helm create`.

### Steps to Create and Use the Helm Chart

1\. **Create the Helm chart:**

   Use the `helm create` command to scaffold a new Helm chart:
   ```sh
   helm create go-app-chart
   ```

![11](https://github.com/jyothiram266/go-web-app/blob/master/screenshots/Screenshot%20from%202024-08-04%2014-18-17.png?raw=true)


This command creates a directory named `go-app-chart` with a default Helm chart structure.

2\. **Update the Helm chart:**

   Replace the default files with your specific configuration
   Directory Structure
   Ensure your directory structure looks like this:
   ![12](https://github.com/jyothiram266/go-web-app/blob/master/screenshots/Screenshot%20from%202024-08-08%2008-36-41.png?raw=true)
   

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

5\. **Modify `templates/deployment.yaml`:**

   Update the `templates/deployment.yaml` file with the following content:

```
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
   
7\. **Modify `templates/ingress.yaml`:**

   Update the `templates/ingress.yaml` file with the following content:

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
   

8\. **Package the Helm chart:**

   Package your Helm chart into a tar.gz file:
   
   ```helm package go-app ```

9\. **Install the Helm chart:**

   Install the chart to your Kubernetes cluster:
   ```
   helm install my-go-app go-app-0.1.0.tgz
   ```

   Replace `my-go-app` with your desired release name.

   ![13](https://github.com/jyothiram266/go-web-app/blob/master/screenshots/Screenshot%20from%202024-08-04%2014-28-29.png?raw=true)

10\. **Verify the deployment:**

    Verify that the deployment, service, and ingress have been created successfully:

    ```sh
     kubectl get deployments
     kubectl get services
     kubectl get ingress
    ```

This process will allow you to create and deploy your application using Helm, with configurable parameters in the `values.yaml` file. Adjust any configurations as needed for your specific use case.

CI/CD Pipeline
--------------

This project uses GitHub Actions to automate the build, test, and deployment processes. The workflow is defined in .github/workflows/deploy.yml and is triggered on every push to the main branch. Below is an explanation of each job in the workflow:

```yaml
on:
  push:
    branches:
      - main
    paths-ignore:
      - 'helm/**'
      - 'k8s/**'
      - 'README.md'

jobs:

  build:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout repository
      uses: actions/checkout@v4

    - name: Set up Go 1.22
      uses: actions/setup-go@v2
      with:
        go-version: 1.22

    - name: Build
      run: go build -o go-web-app

    - name: Test
      run: go test ./...
  
  code-quality:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout repository
      uses: actions/checkout@v4

    - name: Run golangci-lint
      uses: golangci/golangci-lint-action@v6
      with:
        version: v1.56.2
  
  push:
    runs-on: ubuntu-latest

    needs: build

    steps:
    - name: Checkout repository
      uses: actions/checkout@v4

    - name: Set up Docker Buildx
      uses: docker/setup-buildx-action@v1

    - name: Login to DockerHub
      uses: docker/login-action@v3
      with:
        username: ${{ secrets.DOCKERHUB_USERNAME }}
        password: ${{ secrets.DOCKERHUB_TOKEN }}

    - name: Build and Push action
      uses: docker/build-push-action@v6
      with:
        context: .
        file: ./Dockerfile
        push: true
        tags: ${{ secrets.DOCKERHUB_USERNAME }}/go-app:${{github.run_id}}

  update-newtag-in-helm-chart:
    runs-on: ubuntu-latest

    needs: push

    steps:
    - name: Checkout repository
      uses: actions/checkout@v4
      with:
        token: ${{ secrets.TOKEN }}

    - name: Update tag in Helm chart
      run: |
        sed -i 's/tag: .*/tag: "${{github.run_id}}"/' helm/go-web-chart/values.yaml

    - name: Commit and push changes
      run: |
        git config --global user.email "jyothiram266@gmail.com"
        git config --global user.name "jyothiram266"
        git add helm/go-app-chart/values.yaml
        git commit -m "Update tag in Helm chart"
        git push
```

### Workflow Overview

1.  **Build Job**
    
    *   **Purpose:** Build and test the Go application.
        
    *   **Steps:**
        
        *   **Checkout repository:** Fetches the code from the repository.
            
        *   **Set up Go:** Configures the environment with Go 1.22.
            
        *   **Build:** Compiles the Go application into a binary.
            
        *   **Test:** Runs tests on the codebase to ensure functionality.
            
2.  **Code Quality Job**
    
    *   **Purpose:** Analyze code quality using linting tools.
        
    *   **Steps:**
        
        *   **Checkout repository:** Fetches the code from the repository.
            
        *   **Run golangci-lint:** Executes golangci-lint to check for code quality issues.
            
3.  **Push Job**
    
    *   **Purpose:** Build and push the Docker image to DockerHub.
        
    *   **Steps:**
        
        *   **Checkout repository:** Fetches the code from the repository.
            
        *   **Set up Docker Buildx:** Configures Docker Buildx for multi-platform builds.
            
        *   **Login to DockerHub:** Authenticates with DockerHub using credentials stored in GitHub Secrets.
            
        *   **Build and Push:** Builds the Docker image using the Dockerfile and pushes it to DockerHub with a unique tag based on the GitHub run ID.
            
4.  **Update Tag in Helm Chart Job**
    
    *   **Purpose:** Update the image tag in the Helm chart and commit the changes.
        
    *   **Steps:**
        
        *   **Checkout repository:** Fetches the code from the repository.
            
        *   **Update tag in Helm chart:** Updates the Docker image tag in the values.yaml file of the Helm chart.
            
        *   **Commit and push changes:** Commits the updated Helm chart and pushes the changes to the repository.
            
5.  **Deploy Job**
    
    *   **Purpose:** Deploy the updated Helm chart to the EKS cluster.
        
    *   **Steps:**
        
        *   **Checkout repository:** Fetches the code from the repository.
            
        *   **Set up Helm:** Configures Helm for deploying the chart.
            
        *   **Set up Kubernetes:** Configures kubectl to interact with the Kubernetes cluster.
            
        *   **Configure kubectl:** Sets up kubectl with the Kubernetes credentials stored in GitHub Secrets.
            
        *   **Deploy Helm chart:** Uses Helm to upgrade or install the application in the Kubernetes cluster.
            

### Secrets

To ensure the workflow functions correctly, the following secrets must be set in the GitHub repository:

*   **DOCKERHUB\_USERNAME:** Your DockerHub username.
    
*   **DOCKERHUB\_TOKEN:** Your DockerHub access token.
    
*   **TOKEN:** GitHub token for committing changes.
    
*   **KUBECONFIG:** Kubernetes configuration file content or base64 encoded kubeconfig.
    

### How to Modify

To modify the workflow:

1.  **Update the Docker image or tag:** Change the Dockerfile or values.yaml in the Helm chart.
    
2.  **Update Helm chart parameters:** Modify the Helm chart's values.yaml to match your deployment needs.
    
3.  **Change Kubernetes or DockerHub credentials:** Update the GitHub repository secrets if credentials change.
    

This workflow ensures a continuous integration and deployment process, allowing for automated builds, tests, and deployments to the Kubernetes cluster.

### Argo CD Overview

**Argo CD** is a declarative, GitOps continuous delivery tool for Kubernetes. It continuously monitors the Git repository where your Kubernetes manifests (including Helm charts) are stored and automatically syncs them to your Kubernetes cluster, ensuring that the cluster state matches the desired state as defined in your repository.

### Workflow Overview

1.  **Build and Test Job (GitHub Actions)**
    
    *   **Purpose:** Build and test the Go application.
        
    *   **Steps:**
        
        *   Build the Go application.
            
        *   Run tests to ensure the application works as expected.
            
2.  **Push Docker Image Job (GitHub Actions)**
    
    *   **Purpose:** Build and push the Docker image to DockerHub.
        
    *   **Steps:**
        
        *   Build the Docker image using the Dockerfile.
            
        *   Push the image to DockerHub with a unique tag.
            
3.  **Update Helm Chart Job (GitHub Actions)**
    
    *   **Purpose:** Update the image tag in the Helm chart and commit the changes to the repository.
        
    *   **Steps:**
        
        *   Update the image tag in the values.yaml file.
            
        *   Commit and push the changes to the repository.
            
4.  **Continuous Deployment with Argo CD**
    
    *   **Purpose:** Automatically deploy the updated application to the Kubernetes cluster.
        
    *   **How It Works:**
        
        *   Argo CD is configured to monitor the Git repository where your Helm charts are stored.
            
        *   Whenever a new commit is detected in the main branch (containing updated Kubernetes manifests or Helm charts), Argo CD automatically syncs the changes to the Kubernetes cluster.
            
        *   Argo CD ensures that the cluster state always matches the desired state defined in the repository, making deployments predictable and repeatable.
            

### Setting Up Argo CD

To set up Argo CD for this project:

1.  **Install Argo CD:**
    
    *   Follow the [Argo CD installation guide](https://github.com/jyothiram266/go-web-app/blob/master/argocd/01-install.md) to install Argo CD in your Kubernetes cluster.
      ![17](https://github.com/jyothiram266/go-web-app/blob/master/screenshots/Screenshot%20from%202024-08-04%2014-44-22.png?raw=true)
        
2. **Deploy via Argo CD:**
    ![18](https://github.com/jyothiram266/go-web-app/blob/master/screenshots/Screenshot%20from%202024-08-04%2014-46-15.png?raw=true)
    *   Argo CD will automatically deploy the latest version of your application whenever changes are detected in the repository.
        

### Benefits of Using Argo CD

*   **GitOps:** The entire deployment process is driven by Git, providing a clear history of changes.
    
*   **Automation:** Deployment is fully automated, reducing manual intervention.
    
*   **Declarative Approach:** The desired state of your application is declared in Git, and Argo CD ensures the actual state matches it.
    
*   **Rollback:** Easily revert to previous versions by syncing to a previous Git commit.
    

By combining GitHub Actions for CI and Argo CD for CD, you achieve a fully automated and reliable pipeline for delivering your application to production.


### Contributing
Feel free to open issues or submit pull requests for any enhancements or bug fixes.
