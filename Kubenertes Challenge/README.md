# Kubernetes Challenge
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

6. You can stop the AKS cluster which might cost the free credits
```
az aks stop --name myAKSCluster --resource-group myResourceGroup
az aks show --name myAKSCluster --resource-group myResourceGroup #to verify
```

![image](https://github.com/daus89/cloud-resume-challenge/assets/129677949/10c0eb41-38c2-4720-ba0b-d045e2748def)

# Create and use a volume with Azure Blob Storage in Azure Kubernetes Service (AKS)

# Statically vs Dynamically privision storage fo AKS


| Feature                       | Static Provisioning                                           | Dynamic Provisioning                                              |
|-------------------------------|---------------------------------------------------------------|-------------------------------------------------------------------|
| **Definition**                | Manually created Persistent Volumes (PVs)                     | Persistent Volumes (PVs) are automatically provisioned            |
| **Setup**                     | Admin creates PVs manually before they are bound to PVCs      | StorageClass defines the provisioning method, and PVCs trigger PV creation |
| **Flexibility**               | Less flexible; requires pre-configured storage resources      | More flexible; PVs are created on demand based on PVC specifications |
| **Use Case**                  | Useful when specific storage configurations are needed        | Ideal for dynamic and scalable environments                       |
| **Administrative Effort**     | Higher; manual management of storage resources                | Lower; automated management of storage resources                  |
| **StorageClass**              | Optional; not required for PV creation                        | Required; defines the type of storage to be provisioned           |
| **Resource Management**       | Manual adjustment of storage allocation                       | Automatic adjustment and scaling of storage resources             |
| **Example Scenario**          | Dedicated storage for specific applications                   | General-purpose storage for applications needing on-demand volumes |



Create Azure Blob to use as persistent volume(PV) in the AKS cluster.
Reference: https://learn.microsoft.com/en-us/azure/aks/azure-csi-blob-storage-provision?tabs=mount-nfs%2Csecret

1. Enable Blob Storage CSI Driver in AKS Cluster
```
az aks update --enable-blob-driver --name myAKSCluster --resource-group myResourceGroup
```

2. Create Storage Class using NFS Protocol
Create a manifest file blob-nfs-sc.yaml
```
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: azureblob-nfs-premium
provisioner: blob.csi.azure.com
parameters:
  protocol: nfs
  tags: environment=Development
volumeBindingMode: Immediate
allowVolumeExpansion: true
mountOptions:
  - nconnect=4
```
Apply the manifest
```
kubectl apply -f blob-nfs-sc.yaml
```
# Dynamically provision a volume
1. Create Persistent Volume Claim manifest file
```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
    name: azure-managed-disk
spec:
  accessModes:
  - ReadWriteOnce
  storageClassName: azureblob-nfs-premium
  resources:
    requests:
      storage: 5Gi
```
# Statically provision a volume
1. In order to use Azure CLI to create storage, register Microsoft storage provider.
```
az provider register -n Microsoft.Storage --subscription <subscriptiion ID>
```
2. Get the resource group name
```
az aks show --resource-group myResourceGroup --name myAKSCluster --query nodeResourceGroup -o tsv
```
Output
```
MC_myResourceGroup_myAKSCluster_southeastasia
```



# Step 3
# Deploy Your Website to Kubernetes
Kubernetes Deployment: Create a website-deployment.yaml defining a Deployment that uses the Docker image created in Step 1A. Ensure the Deployment specifies the necessary environment variables and mounts for the database connection.
Outcome: The e-commerce web application is running on Kubernetes, with pods managed by the Deployment.


