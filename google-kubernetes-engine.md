
# Deploying a Web Application Using Google Kubernetes Engine (GKE)

## Introduction

This guide explains how to deploy a Kubernetes cluster on Google Kubernetes Engine (GKE) and deploy a web application on it. By following these steps, you'll gain an understanding of creating a cluster via the command line and managing deployments.


## Step 1: Log in to Google Cloud Console

1. Go to the [Google Cloud Console](https://console.cloud.google.com/).
2. Accept the terms of service:
   - Select your country.
   - Agree to the terms and click **AGREE AND CONTINUE**.


## Step 2: Enable the Kubernetes Engine API

1. From the console menu, navigate to **Kubernetes Engine** > **Clusters**.
2. Click **ENABLE** when prompted to enable the Kubernetes Engine API.
3. Wait a few moments for the API to be enabled.


## Step 3: Create a Kubernetes Cluster

1. On the Kubernetes clusters page, click **CREATE**.
2. Choose **Standard: You manage your cluster** and click **CONFIGURE**.
   - If an Autopilot cluster is selected by default, click **SWITCH TO STANDARD CLUSTER**.
3. Configure the following settings under **Cluster basics**:
   - **Name**: `test-cluster`
   - **Location type**: Zonal (default)
   - **Zone**: `us-central1-c`
   - **Default node locations**: Check the box and select `us-central1-b`.
4. Under **NODE POOLS** > **default-pool**, configure the following:
   - **Number of nodes (per zone)**: 1
   - Enable **Cluster autoscaler**:
     - Minimum: 1
     - Maximum: 2
   - **Node locations**: Verify `us-central1-b` and `us-central1-c` are selected.
   - **Boot disk size**: Set to 10 GB.
5. Leave other options as default and click **CREATE**.


## Step 4: Connect to the Cluster Using Cloud Shell

1. Click the **Activate Cloud Shell** icon in the top-right of the console.
2. Click **CONTINUE** to activate Cloud Shell.
3. Run the following command to connect to the cluster:
   ```bash
   gcloud container clusters get-credentials test-cluster --zone us-central1-c
   ```
4. Authorize the Cloud Shell when prompted.


## Step 5: Deploy the Web Application

1. Create a YAML file for the deployment:
   - Open the Cloud Shell Editor.
   - Create a new file named `voting.yaml` in the `cloud_user` directory.
   - Paste the provided YAML configuration into the file.
   - Save the file.

2. Deploy the application:
   ```bash
   kubectl apply -f voting.yaml
   ```

3. Verify the deployment:
   - Check pods:
     ```bash
     kubectl get pods
     ```
   - Check deployment:
     ```bash
     kubectl get deployment
     ```
   - Check services:
     ```bash
     kubectl get svc
     ```

## Step 6: Access the Web Application

1. Wait for the external IP address to be assigned to the `vote-service` LoadBalancer.
2. Copy the external IP address and open it in a web browser.
3. Confirm the application is running by interacting with the voting widget.


## Step 7: Validate Pod IDs

1. Note the container ID displayed on the web page.
2. Match the container ID with one of the pods:
   ```bash
   kubectl get pods
   ```
3. Refresh the web page until the container ID changes to match the second pod.


## Conclusion

Congratulations! You have successfully deployed a web application on GKE using Kubernetes. You now understand how to:
- Create and manage a Kubernetes cluster.
- Deploy and scale applications.
- Access and validate services in GKE.

---
