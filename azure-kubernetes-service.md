# Deploying a Web Application Using Azure Kubernetes Service (AKS)

## Introduction

This guide explains how to deploy a Kubernetes cluster on Azure Kubernetes Service (AKS) and deploy a test application. Following these steps will give you a practical understanding of managing AKS clusters and applications.

## Step 1: Log in to Azure Portal

1. Go to the [Azure Portal](https://portal.azure.com/).

## Step 2: Create an AKS Cluster

1. In the Azure Portal, search for and select **Kubernetes Services** in the search box.
2. On the **Kubernetes services** page, click **Create** and choose **Kubernetes cluster**.
3. On the **Create Kubernetes cluster > Basics** page, configure the following:
   - **Subscription**: Use any subscription.
   - **Resource group**: Select the resource group.
   - **Cluster preset configuration**: Select **Dev/Test**.
   - **Kubernetes cluster name**: Enter `cluster1`.
   - **Region**: Select **East US**.
4. Click **Next**.

## Step 3: Configure the System Node Pool

1. In the node pool list, click the system node pool named `agentpool`.
2. Configure the following settings:
   - **Node size**: Choose **Standard D2s v3**.
   - **Scale method**: Select **Autoscale**.
     - **Minimum node count**: 1
     - **Maximum node count**: 3
3. Click **Update**.
4. Click **Review + Create**.
5. Wait for the cluster creation to complete (this may take up to 5 minutes).



## Step 4: Deploy a Test Application

1. Once the cluster is created, click **Go to resource**.
2. From the top menu, click **+ Create** and select **Apply a YAML** from the dropdown.
3. Copy the `manifest.yaml` content provided in the instructions and paste it into the file.
4. Click **Add** to deploy the application.

```yaml
# manifest.yaml

apiVersion: v1
kind: Service
metadata:
  name: myaksapp-service
  labels:
    app: myaksapp
spec:
  selector:
    app: myaksapp
  ports:
   - name: http
     port: 80
     targetPort: 80
  type: LoadBalancer
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: web-config
data:
  index.html: |
            <!doctype html>
            <html lang="en">
            <head>
            <meta charset="utf-8">
            <title>My AKS App</title>
            </head>
            <body>
            <h1 style="text-align: center">Hello from Azure Kubernetes Service!</h1>
            </body>
            </html>
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myaksapp-deployment
  labels:
    app: myaksapp
spec:
  replicas: 2
  selector:
    matchLabels:
      app: myaksapp
  template:
    metadata:
      labels:
        app: myaksapp
    spec:
      volumes:
         - name: config-volume
           configMap:
             name: web-config
      containers:
      - name: myaksapp-con
        image: nginx:stable
        volumeMounts:
        - name: config-volume
          mountPath: /usr/share/nginx/html
        ports:
        - name: http
          containerPort: 80
```


## Step 5: Verify the Application Deployment

1. In the left-hand menu, under **Kubernetes resources**, click **Workloads**.
2. Verify that the deployment `myaksapp-deployment` is listed with 2 replicas.



## Step 6: Test the Application

1. In the left-hand menu, under **Kubernetes resources**, click **Services and ingresses**.
2. Locate the `myaksapp-service` entry and note the **External IP**.
3. Click the External IP link for `myaksapp-service` to open the test application in a browser.



## Conclusion

Congratulations! You have successfully deployed a Kubernetes cluster using Azure Kubernetes Service (AKS) and launched a test application. You now understand how to:
- Create and configure AKS clusters.
- Deploy and verify applications on AKS.
- Access services using external IPs.

---
