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
1. Create Persistent Volume Claim manifest file pvc.yaml
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
      storage: 10Gi
```
2. Apply the pvc manifest
```
kubectl apply -f pvc.yaml
```
3. Check the newly created pvc
```
kubectl get pvc
```


# Step 3
# Deploy Your Website to Kubernetes and Expose to Public Via LoadBalance
Kubernetes Deployment: Create a website-deployment.yaml defining a Deployment that uses the Docker image created in Step 1A. Ensure the Deployment specifies the necessary environment variables and mounts for the database connection.
Outcome: The e-commerce web application is running on Kubernetes, with pods managed by the Deployment.

# Create Database manifest and necessarilly configuration
1. Create and apply configmap to configure the DB db-configmap.yaml
```
apiVersion: v1
kind: ConfigMap
metadata:
  name: mariadb-init
data:
  init.sql: |
   USE ecomdb;
   CREATE TABLE products (id mediumint(8) unsigned NOT NULL auto_increment,Name varchar(255) default NULL,Price varchar(255) default NULL, ImageUrl varchar(255) default NULL,PRIMARY KEY (id)) AUTO_INCREMENT=1;
   INSERT INTO products (Name,Price,ImageUrl) VALUES ("Laptop","100","c-1.png"),("Drone","200","c-2.png"),("VR","300","c-3.png"),("Tablet","50","c-5.png"),("Watch","90","c-6.png"),("Phone Covers","20","c-7.png"),("Phone","80","c-8.png"),("Laptop","150","c-4.png");  
```
```
kubectl apply -f db-configmap.yaml
```
Check the created configmap
```
kubectl get configmap
```

![image](https://github.com/daus89/cloud-resume-challenge/assets/129677949/048892de-4753-4538-84ce-09d02d4dc424)

2. Create kubernetes Secret to safely use it in the Database manifest  later.
Encode the password first
```
echo -n 'yourpassword' | base64
```
Create secret.yaml 
```
apiVersion: v1
kind: Secret
metadata:
  name: db-secret
type: Opaque
data:
  mysql-root-password: cGFzc3dvcmQ=  # base64 encoded password
```
Create the secret
```
kubectl apply -f secret.yaml
```


3. Create Database manifest mariadb-deployment.yaml, service and mount the configmap mariadb-initdb as volume, use previously created pvc azure-managed-disk
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mariadb
spec:
  selector:
    matchLabels:
      app: mariadb
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: mariadb
    spec:
      containers:
      - image: mariadb:10.5
        name: mariadb
        env:
        - name: MYSQL_USER
          value: root
        - name: MYSQL_ROOT_PASSWORD
          value: abc123
        - name: MYSQL_DATABASE
          value: ecommerce
        volumeMounts:
        - name: mariadb-persistent-storage
          mountPath: /var/lib/mysql
        - name: mariadb-initdb
          mountPath: /docker-entrypoint-initdb.d
      volumes:
      - name: mariadb-persistent-storage
        persistentVolumeClaim:
          claimName: azure-managed-disk
      - name: mariadb-initdb
        configMap:
          name: mariadb-initdb

---
apiVersion: v1
kind: Service
metadata:
  name: mysql-service
spec:
  ports:
  - port: 3306
  selector:
    app: mariadb
```
```
kubectl apply -f mariadb-deployment.yaml
```

4. Create application deployment manifest ecommerce-deployment.yaml to use the mariadb deployment
```
apiVersion: apps/v1
kind: Deployment
metadata:
  generation: 1
  labels:
    app: my-ecommerce
  name: my-ecommerce
  namespace: default
spec:
  progressDeadlineSeconds: 600
  replicas: 2
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      app: my-ecommerce
  strategy:
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
    type: RollingUpdate
  template:
    metadata:
      labels:
        app: my-ecommerce
    spec:
      containers:
      - image: daus89/myecommerce-app:v1
        imagePullPolicy: Always
        name: myecommerce-app
        ports:
        - containerPort: 80
        env:
        - name: DB_HOST
          value: mysql-service
        - name: DB_PORT
          value: "3306"
        - name: DB_NAME
          value: ecommerce
        - name: DB_USER
          value: "root"
        - name: DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: db-secret
              key: mysql-root-password
---
---
apiVersion: v1
kind: Service
metadata:
  name: my-app
spec:
  selector:
    app: my-ecommerce
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
  type: LoadBalancer
```

5. Access the application from public
Get the LoadBalancer IP
```
kubectl get service -o wide
```
![image](https://github.com/daus89/cloud-resume-challenge/assets/129677949/290c176f-89f5-4084-8359-3c7c27f8ee52)

Access the IP through browser

![image](https://github.com/daus89/cloud-resume-challenge/assets/129677949/6a89f7bd-457a-405b-afae-9aafa1716f91)

# Step 4
# Autoscale Your Application

Task: Automate scaling based on CPU usage to handle unpredictable traffic spikes.

Implement HPA: Create a Horizontal Pod Autoscaler targeting 50% CPU utilization, with a minimum of 2 and a maximum of 10 pods.
Apply HPA: Execute kubectl autoscale deployment ecom-web --cpu-percent=50 --min=2 --max=10.
Simulate Load: Use a tool like Apache Bench to generate traffic and increase CPU load.
Monitor Autoscaling: Observe the HPA in action with kubectl get hpa.
Outcome: The deployment automatically adjusts the number of pods based on CPU load, showcasing Kubernetes’ capability to maintain performance under varying loads.

1. Ensure  the deployment has appropriate CPU requests and limits. The HPA will only work if these are set correctly.
Add
```
  resources:
          requests:
            cpu: "100m"
          limits:
            cpu: "500m"
```
in deployment spec and reapply the deployment

```
kubectl apply -f ecommerce-deployment.yaml
```

2. Apply autoscale using HPA targeting 50% CPU utilization, with a minimum of 2 and a maximum of 10 pods.
```
kubectl autoscale deployment my-ecommerce --cpu-percent=50 --min=2 --max=10
```
3. Use Apache Bench to load test the deployment
First, install the Apache Bench (AB)
```
sudo apt-get update
sudo apt-get install apache2-utils
```
Second, run the AB against our public web.
```
ab -n 50000 -c 200 http://<app public IP>/
```
-n 50000 specifies the total number of requests to perform.
-c 200 specifies the number of multiple requests to perform at a time.

Use these kubectl commands to monitor the autoscale functionallity while running the AB test.
```
kubectl get hpa --watch
```
```
kubectl get pods --watch
```

![image](https://github.com/daus89/cloud-resume-challenge/assets/129677949/cfbbfc82-fcbc-477d-ac54-9ad92b71f492)

As you can see in window 1, the Apache Bench is running and the same same time in window 2, the pod is creating while in window 3 the deployment is increasing in replicas count.

# Step 5
# Implement Liveness and Readiness Probes

Task: Ensure the web application is restarted if it becomes unresponsive and doesn’t receive traffic until ready.
Define Probes: Add liveness and readiness probes to website-deployment.yaml, targeting an endpoint in your application that confirms its operational status.
Apply Changes: Update your deployment with the new configuration.
Test Probes: Simulate failure scenarios (e.g., manually stopping the application) and observe Kubernetes’ response.
Outcome: Kubernetes automatically restarts unresponsive pods and delays traffic to newly started pods until they’re ready, enhancing the application’s reliability and availability.

Liveliness: The kubelet uses liveness probes to know when to restart a container.
Readiness: The kubelet uses readiness probes to know when a container is ready to start accepting traffic.

1. To demonstrate the liveliness probe, add these into ecommerce-deployment.yaml file, under spec:
```
args:
    - /bin/sh
    - -c
    - touch /tmp/healthy; sleep 30; rm -f /tmp/healthy; sleep 600
    livenessProbe:
      exec:
        command:
        - cat
        - /tmp/healthy
      initialDelaySeconds: 5
      periodSeconds: 5
```

In the configuration file, you can see that the Pod has a single Container. The periodSeconds field specifies that the kubelet should perform a liveness probe every 5 seconds. The initialDelaySeconds field tells the kubelet that it should wait 5 seconds before performing the first probe. To perform a probe, the kubelet executes the command cat /tmp/healthy in the target container. If the command succeeds, it returns 0, and the kubelet considers the container to be alive and healthy. If the command returns a non-zero value, the kubelet kills the container and restarts it.

When the container starts, it executes this command:

/bin/sh -c "touch /tmp/healthy; sleep 30; rm -f /tmp/healthy; sleep 600"

For the first 30 seconds of the container's life, there is a /tmp/healthy file. So during the first 30 seconds, the command cat /tmp/healthy returns a success code. After 30 seconds, cat /tmp/healthy returns a failure code.

2. Describe the pod events to check the liveliness probe kick in.
```
kubectl describe pod <pod name>
```
![image](https://github.com/daus89/cloud-resume-challenge/assets/129677949/6d3e2ce9-bef4-4855-86bc-7bda115738eb)





## Challenges
1. Application is not accessible from public IP
Check service and make sure service endpoint pointing to application
```
kubectl get service -o wide
```
Check deployment for errors
```
kubectl get deployments
```



