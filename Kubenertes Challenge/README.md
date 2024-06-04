# Kubenertes Challenge
Reference: https://cloudresumechallenge.dev/docs/extensions/kubernetes-challenge/

# Intro
Imagine you are going to deploy an e-commerce website. It’s crucial to consider the challenges of modern web application deployment and how containerization and Kubernetes (K8s) offer compelling solutions:

Scalability: How can your application automatically adjust to fluctuating traffic?
Consistency: How do you ensure your application runs the same across all environments?
Availability: How can you update your application with zero downtime?
Containerization, using Docker, encapsulates your application and its environment, ensuring it runs consistently everywhere. Kubernetes, a container orchestration platform, automates deployment, scaling, and management, offering:

Dynamic Scaling: Adjusts application resources based on demand.
Self-healing: Restarts failed containers and reschedules them on healthy nodes.
Seamless Updates & Rollbacks: Enables zero-downtime updates and easy rollbacks.
By leveraging Kubernetes and containerization for your challenge, you embrace a scalable, consistent, and resilient deployment strategy. This not only demonstrates your technical acumen but aligns with modern DevOps practices.

# Step 1
# Containerize Your E-Commerce Website and Database
A. Web Application Containerization
Create a Dockerfile: Navigate to the root of the e-commerce application and create a Dockerfile. 
Create Dockerfile. Reference: https://docs.docker.com/reference/dockerfile/#here-documents

This file should instruct Docker to:
Use php:7.4-apache as the base image.
Install mysqli extension for PHP.
Copy the application source code to /var/www/html/.
Update database connection strings to point to a Kubernetes service named mysql-service.
Expose port 80 to allow traffic to the web server.
Build and Push the Docker Image:

Execute docker build -t yourdockerhubusername/ecom-web:v1 . to build your image.
```
$docker build -t my-ecommerce-app .
```
Push it to Docker Hub with docker push yourdockerhubusername/ecom-web:v1.

```
$docker login --username username
$docker tag my-ecommerce-app daus89/myecommerce-app:v1
$docker push daus89/myecommerce-app:v1
```

Outcome: Your web application Docker image is now available on Docker Hub.
B. Database Containerization
Database Preparation: Instead of containerizing the database yourself, you’ll use the official MariaDB image. Prepare the database initialization script (db-load-script.sql) to be used with Kubernetes ConfigMaps or as an entrypoint script.

# Step 2
# Set Up Kubernetes on a Public Cloud Provider

Cluster Creation: Choose AWS (EKS), Azure (AKS), or GCP (GKE) and follow their documentation to create a Kubernetes cluster. Ensure you have kubectl configured to interact with your cluster.
Outcome: A fully operational Kubernetes cluster ready for deployment.

Create new azure ID with new email to get free 200USD credit.
Make sure to run CMD/Powershell as Administrator.

1. Install azure CLI: https://learn.microsoft.com/en-us/cli/azure/install-azure-cli
2. Login into azure account using azure cli
```
az login --tenant <microsoft entra tenant ID>
```
3. Create New Azure Resource Group
```
az group create --name myResourceGroup --location southeastasia
```
4. Create new Azure Container Registry (ACR)
```

az acr create --name <Propose any acrName> --resource-group myResourceGroup --sku basic
```
5. Create AKS resources
```
az aks create --resource-group myResourceGroup --name myAKSCluster --node-count 1 --generate-ssh-keys --attach-acr <acrName>
```
![image](https://github.com/daus89/cloud-resume-challenge/assets/129677949/ab7680e5-aa2f-46d0-91a2-2bb694589c5d)

