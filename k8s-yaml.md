# Kubernetes YAML Configuration Guide

This document explains how to create and understand different types of YAML files for Kubernetes resources. 

## Table of Contents
1. [Understanding YAML Basics](#understanding-yaml-basics)
2. [Creating a Pod YAML](#creating-a-pod-yaml)
3. [Creating a Deployment YAML](#creating-a-deployment-yaml)
4. [Creating a Service YAML](#creating-a-service-yaml)
5. [Creating a ConfigMap YAML](#creating-a-configmap-yaml)
6. [Creating a Secret YAML](#creating-a-secret-yaml)
7. [Creating a Persistent Volume and Persistent Volume Claim](#creating-a-persistent-volume-and-persistent-volume-claim)
8. [Best Practices](#best-practices)

---

## Understanding YAML Basics
YAML (Yet Another Markup Language) is a human-readable data serialization format often used for configuration files. Kubernetes uses YAML to define resources.

### Key Concepts
- **Indentation**: YAML uses spaces (not tabs) for hierarchy.
- **Key-Value Pairs**: Data is represented in `key: value` format.
- **Lists**: Use a hyphen `-` to define items in a list.
- **Comments**: Add comments using `#`.

**Example YAML Structure**:
```yaml
apiVersion: v1          # Specifies the API version
kind: Pod               # Type of Kubernetes resource
metadata:               # Metadata about the resource
  name: my-pod          # Name of the resource
  labels:               # Key-value pairs for identifying resources
    app: my-app         
spec:                   # Specifications for the resource
  containers:           # List of containers
    - name: my-container  # Container name
      image: nginx        # Image used for the container
      ports:
        - containerPort: 80 # Port the container exposes
```

---

## Creating a Pod YAML
A Pod is the smallest deployable unit in Kubernetes that runs one or more containers.

### **Pod YAML File**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
  labels:
    app: nginx
spec:
  containers:
    - name: nginx-container
      image: nginx:latest
      ports:
        - containerPort: 80
```

### **Explanation**:
1. **`apiVersion`**: `v1` indicates the Kubernetes API version for Pod.
2. **`kind`**: Specifies the resource type (`Pod`).
3. **`metadata`**: Contains the resource name (`my-pod`) and labels.
4. **`spec`**: Defines the specification of the Pod.
   - **`containers`**: A list of containers in the Pod.
   - **`image`**: Docker image to use (e.g., `nginx:latest`).
   - **`ports`**: The container exposes port 80.

### **Apply the Pod YAML**:
```bash
kubectl apply -f pod.yaml
```

### **Verify the Pod**:
```bash
kubectl get pods
```

---

## Creating a Deployment YAML
A Deployment manages multiple replicas of Pods and ensures that the desired state matches the current state.

### **Deployment YAML File**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
        - name: nginx-container
          image: nginx:latest
          ports:
            - containerPort: 80
```

### **Explanation**:
1. **`apiVersion`**: `apps/v1` indicates the API version for Deployment.
2. **`kind`**: Specifies the resource type (`Deployment`).
3. **`replicas`**: Number of Pod replicas (3 in this case).
4. **`selector`**: Matches the labels of the Pods.
5. **`template`**: Defines the Pod template (metadata and spec for the Pods).
6. **`containers`**: List of containers, including name, image, and ports.

### **Apply the Deployment YAML**:
```bash
kubectl apply -f deployment.yaml
```

### **Verify the Deployment**:
```bash
kubectl get deployments
kubectl get pods
```

---

## Creating a Service YAML
A Service exposes a set of Pods to external traffic or other resources.

### **Types of Services**:
- **ClusterIP**: Exposes service internally in the cluster.
- **NodePort**: Exposes service on a static port across each node.
- **LoadBalancer**: Exposes service externally using a cloud provider's load balancer.

### **Service YAML File** (NodePort Example)
```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  selector:
    app: nginx
  type: NodePort
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
      nodePort: 30007
```

### **Explanation**:
1. **`apiVersion`**: `v1` for Services.
2. **`kind`**: Specifies the resource type (`Service`).
3. **`selector`**: Matches Pods with the label `app: nginx`.
4. **`type`**: Defines the Service type (`NodePort`).
5. **`ports`**:
   - **`port`**: Service port (80).
   - **`targetPort`**: Container port (80).
   - **`nodePort`**: External port exposed on each node (e.g., 30007).

### **Apply the Service YAML**:
```bash
kubectl apply -f service.yaml
```

### **Verify the Service**:
```bash
kubectl get services
```

---

## Creating a ConfigMap YAML
A ConfigMap is used to store non-confidential configuration data as key-value pairs.

### **ConfigMap YAML File**
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  APP_ENV: production
  APP_DEBUG: "false"
```

### **Explanation**:
- **`data`**: Contains key-value pairs for configuration.

### **Apply the ConfigMap**:
```bash
kubectl apply -f configmap.yaml
```

---

## Creating a Secret YAML
A Secret is used to store sensitive data such as passwords or API tokens.

### **Secret YAML File**
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: my-secret
type: Opaque
data:
  username: dXNlcjEyMw==    # Base64-encoded value of "user123"
  password: cGFzc3dvcmQ=    # Base64-encoded value of "password"
```

### **Explanation**:
- **`type`**: `Opaque` is a general-purpose Secret type.
- **`data`**: Values must be base64-encoded.

### **Apply the Secret**:
```bash
kubectl apply -f secret.yaml
```

---

## Creating a Persistent Volume and Persistent Volume Claim
Persistent Volumes (PVs) provide persistent storage, and Persistent Volume Claims (PVCs) request storage from PVs.

### **Persistent Volume YAML**
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: my-pv
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: /data/pv-volume
```

### **Persistent Volume Claim YAML**
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
```

---

## Best Practices
1. Use clear and consistent naming conventions.
2. Separate configurations into different YAML files.
3. Use ConfigMaps and Secrets for configuration.
4. Avoid hardcoding values in YAML files.
5. Validate YAML files before applying:
   ```bash
   kubectl apply --dry-run=client -f <file.yaml>
   ```

---


