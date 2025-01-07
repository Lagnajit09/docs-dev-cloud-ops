
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
   - Create a new file named `voting.yaml` in the user directory.
   - Paste the provided YAML configuration into the file.
   - Save the file.

  ```yaml
apiVersion: v1
kind: Service
metadata:
  name: vote-service
  labels:
    app: vote
spec:
  selector:
    app: vote
  ports:
  - name: http
    port: 80
    targetPort: 80
  - name: udp
    port: 5000
    targetPort: 5000
  type: LoadBalancer
---
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: vote
  name: vote
spec:
  replicas: 2
  selector:
    matchLabels:
      app: vote
  template:
    metadata:
      labels:
        app: vote
    spec:
      containers:
      - image: dockersamples/examplevotingapp_vote
        name: vote
        ports:
        - containerPort: 80
          name: vote
---
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: worker
  name: worker
spec:
  replicas: 1
  selector:
    matchLabels:
      app: worker
  template:
    metadata:
      labels:
        app: worker
    spec:
      containers:
      - image: dockersamples/examplevotingapp_worker
        name: worker
---
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: db
  name: db
spec:
  replicas: 1
  selector:
    matchLabels:
      app: db
  template:
    metadata:
      labels:
        app: db
    spec:
      containers:
      - image: postgres:15-alpine
        name: postgres
        env:
        - name: POSTGRES_USER
          value: postgres
        - name: POSTGRES_PASSWORD
          value: postgres
        ports:
        - containerPort: 5432
          name: postgres
        volumeMounts:
        - mountPath: /var/lib/postgresql/data
          name: db-data
      volumes:
      - name: db-data
        emptyDir: {}
---
apiVersion: v1
kind: Service
metadata:
  labels:
    app: db
  name: db
spec:
  type: ClusterIP
  ports:
  - name: "db-service"
    port: 5432
    targetPort: 5432
  selector:
    app: db
---
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: redis
  name: redis
spec:
  replicas: 1
  selector:
    matchLabels:
      app: redis
  template:
    metadata:
      labels:
        app: redis
    spec:
      containers:
      - image: redis:alpine
        name: redis
        ports:
        - containerPort: 6379
          name: redis
        volumeMounts:
        - mountPath: /data
          name: redis-data
      volumes:
      - name: redis-data
        emptyDir: {}
---
apiVersion: v1
kind: Service
metadata:
  labels:
    app: redis
  name: redis
spec:
  type: ClusterIP
  ports:
  - name: "redis-service"
    port: 6379
    targetPort: 6379
  selector:
    app: redis
---
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: result
  name: result
spec:
  replicas: 1
  selector:
    matchLabels:
      app: result
  template:
    metadata:
      labels:
        app: result
    spec:
      containers:
      - image: dockersamples/examplevotingapp_result
        name: result
        ports:
        - containerPort: 80
          name: result
---
apiVersion: v1
kind: Service
metadata:
  labels:
    app: result
  name: result
spec:
  type: LoadBalancer
  ports:
  - name: http
    port: 80
    targetPort: 80
  - name: udp
    port: 5001
    targetPort: 5001
  selector:
    app: result
```

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
