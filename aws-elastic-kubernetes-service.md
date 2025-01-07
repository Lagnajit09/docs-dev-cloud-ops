# Deploying a Web Application on AWS Elastic Kubernetes Service (EKS)

## Table of Contents
- [Introduction](#introduction)
- [Environment Setup](#environment-setup)
  - [Creating an Admin User](#creating-an-admin-user)
  - [Setting up the EC2 Instance](#setting-up-the-ec2-instance)
  - [Installing Required Tools](#installing-required-tools)
- [Cluster Creation and Configuration](#cluster-creation-and-configuration)
- [Application Deployment](#application-deployment)
- [High Availability Testing](#high-availability-testing)
- [Cleanup and Best Practices](#cleanup-and-best-practices)

## Introduction

This guide provides detailed instructions for deploying a web application using Amazon Elastic Kubernetes Service (EKS). EKS is a managed Kubernetes service that simplifies the deployment, management, and scaling of containerized applications using Kubernetes.

## Environment Setup

### Creating an Admin User

1. Navigate to AWS IAM Console
2. Create a new user named `eks-user`:
   ```bash
   # Required permissions
   - AdministratorAccess policy
   ```
   > **Important**: Store the access keys securely. They will be required for AWS CLI configuration.

### Setting up the EC2 Instance

Create an EC2 instance to serve as your administrative workstation:

1. Launch a new EC2 instance with these specifications:
   - Name: `eks-admin`
   - AMI: Amazon Linux 2
   - Instance Type: t3.medium
   - Auto-assign Public IP: Enable
   - Key Pair: Create new key pair named `admin-key`

### Installing Required Tools

1. **AWS CLI v2**:
   ```bash
   curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
   unzip awscliv2.zip
   sudo ./aws/install --bin-dir /usr/bin --install-dir /usr/bin/aws-cli --update
   ```

2. **Configure AWS CLI**:
   ```bash
   aws configure
   # Enter:
   # - AWS Access Key ID
   # - AWS Secret Access Key
   # - Default region: us-east-1
   # - Default output format: json
   ```

3. **Install kubectl**:
   ```bash
   curl -O https://s3.us-west-2.amazonaws.com/amazon-eks/1.25.6/2023-01-30/bin/linux/amd64/kubectl
   chmod +x ./kubectl
   mkdir -p $HOME/bin && cp ./kubectl $HOME/bin/kubectl && export PATH=$PATH:$HOME/bin
   ```

4. **Install eksctl**:
   ```bash
   curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
   sudo mv /tmp/eksctl /usr/local/bin
   ```

## Cluster Creation and Configuration

1. Create the EKS cluster:
   ```bash
   eksctl create cluster \
     --name myeks \
     --region us-east-1 \
     --zones=us-east-1a,us-east-1b \
     --nodegroup-name eks-workers \
     --node-type t3.medium \
     --nodes 2 \
     --nodes-min 2 \
     --nodes-max 4 \
     --managed
   ```

   > **Note**: Cluster creation typically takes 15-20 minutes.

2. Configure kubectl for cluster access:
   ```bash
   aws eks update-kubeconfig --name myeks --region us-east-1
   ```

3. Verify cluster status:
   ```bash
   eksctl get cluster
   ```

## Application Deployment

1. Create the application manifest (`app.yaml`):
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
     name: vote-deployment
     labels:
       app: vote
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
           - name: vote
             image: dockersamples/examplevotingapp_vote:before
             ports:
               - name: http
                 containerPort: 80
               - name: udp
                 containerPort: 5000
   ```

2. Deploy the application:
   ```bash
   kubectl apply -f app.yaml
   ```

3. Verify deployment:
   ```bash
   kubectl get deployment
   kubectl get pods
   kubectl get svc
   ```

   > **Note**: Wait for the LoadBalancer's EXTERNAL-IP to be assigned before accessing the application.

## High Availability Testing

Test the cluster's high availability capabilities:

1. Identify the worker nodes in EC2 console
2. Terminate both worker nodes simultaneously
3. Observe automatic node replacement
4. Verify application availability during node replacement

> **Important**: The cluster should maintain availability during node replacement, demonstrating the self-healing capabilities of EKS.

## Cleanup and Best Practices

To avoid unnecessary charges, clean up resources when they're no longer needed:

1. Delete the application:
   ```bash
   kubectl delete -f app.yaml
   ```

2. Delete the cluster:
   ```bash
   eksctl delete cluster --name myeks --region us-east-1
   ```

Best Practices:
- Regularly monitor cluster health and performance
- Implement proper logging and monitoring
- Use resource requests and limits
- Keep EKS and kubectl versions up to date
- Implement proper security groups and network policies
- Use namespace segregation for different environments
- Implement proper backup strategies for persistent data

## Additional Considerations

1. **Security**:
   - Implement RBAC policies
   - Use AWS KMS for secrets encryption
   - Configure network policies
   - Regular security audits

2. **Scaling**:
   - Configure Horizontal Pod Autoscaling
   - Implement Cluster Autoscaler
   - Monitor resource utilization

3. **Monitoring**:
   - Set up CloudWatch monitoring
   - Configure logging
   - Implement alerting mechanisms

4. **Cost Optimization**:
   - Use Spot Instances where appropriate
   - Implement proper resource requests and limits
   - Regular cost monitoring and optimization

5. **Backup and Disaster Recovery**:
   - Regular backup of cluster state
   - Cross-region disaster recovery plan
   - Testing restoration procedures
