# Kubernetes Cluster Configuration
---

### **1. Preparing the VM**

#### **Update and Upgrade**
```bash
sudo apt-get update -y
sudo apt-get upgrade -y
```
- **Purpose**: Updates the package lists and upgrades all installed packages on your VM to the latest version. This ensures your VM has the latest security patches and compatible libraries.

#### **Install Required Tools**
```bash
sudo apt-get install -y apt-transport-https ca-certificates curl
```
- **Purpose**: Installs basic tools needed for downloading packages securely from external repositories:
  - `apt-transport-https`: Enables `apt` to fetch packages over HTTPS.
  - `ca-certificates`: Ensures secure communication with external repositories.
  - `curl`: Command-line tool for downloading files.

---

### **2. Disable Swap**
```bash
sudo swapoff -a
sudo sed -i '/ swap / s/^/#/' /etc/fstab
```
Why Kubernetes needs swap off:

- Predictable Performance:

   - Kubernetes needs precise control over memory allocation
   - Swap can make memory usage unpredictable
   - Container performance becomes inconsistent with swap


- Resource Management:

   - Kubernetes manages container resources (CPU, memory) precisely
   - Swap interferes with this management
   - Could lead to containers running when they should be killed
- `swapoff -a`: Disables swap immediately.
- `sed '/ swap / s/^/#/' /etc/fstab`: Comments out the swap entry in the `/etc/fstab` file, preventing swap from being enabled on reboot.

---

### **3. Load Kernel Modules**
```bash
sudo modprobe overlay
sudo modprobe br_netfilter
```
- These modules are crucial because:
   1. Container Runtime needs overlay filesystem
   2. Kubernetes networking requires br_netfilter for:
      -  Service networking
      -  Pod networking
      -  Network policies
      -  Load balancing
- `overlay`: Ensures file system overlays work efficiently for containers.
- `br_netfilter`: Ensures network traffic between pods is handled correctly.

---

### **4. Configure Networking Settings**
```bash
sudo tee /etc/sysctl.d/k8s.conf <<EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
EOF
sudo sysctl --system
```
```bash
net.bridge.bridge-nf-call-iptables = 1
# Enables iptables rules to work with bridge networking
# Allows Kubernetes to properly handle container networking
# Essential for pod-to-pod communication

net.bridge.bridge-nf-call-ip6tables = 1
# Same as above but for IPv6
# Ensures IPv6 traffic is also properly handled

net.ipv4.ip_forward = 1
# Enables IP forwarding
# Allows containers to communicate with the outside world
# Required for container networking to work properly
```
- **Command breakdown**:
  - `tee`: Writes configuration to `/etc/sysctl.d/k8s.conf`.
  - `sysctl --system`: Applies the new network settings immediately.

---

### **5. Install Container Runtime**
```bash
sudo apt-get install -y containerd
```
- **Purpose**: Installs `containerd`, the container runtime that Kubernetes uses to manage containers (e.g. pulling images, manage storage, start/stop containers, managing network interfaces.).
- Why is it required for Kubernetes?
   - Kubernetes doesn't handle containers directly
   - It needs a container runtime to:
      - Create containers from images
      - Manage container lifecycle
      - Handle container networking
      - Manage container storage
```bash
sudo mkdir -p /etc/containerd
sudo containerd config default | sudo tee /etc/containerd/config.toml
sudo systemctl restart containerd
sudo systemctl enable containerd
```
- **Commands explained**:
  - `containerd config default`: Generates a default configuration file for containerd.
  - `systemctl restart containerd`: Restarts the container runtime with the new configuration.
  - `systemctl enable containerd`: Ensures `containerd` starts automatically on boot.

---

### **6. Install Kubernetes Components**

#### Add Kubernetes Repository:
```bash
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
sudo apt-add-repository "deb http://apt.kubernetes.io/ kubernetes-xenial main"
sudo apt-get update -y
```
- **Purpose**: Adds Kubernetes's official package repository to your VM:
  - `apt-key add`: Adds Google’s GPG key to verify the packages.
  - `apt-add-repository`: Adds the Kubernetes package repository.

#### Install Tools:
```bash
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
```
- **Purpose**:
  - `kubelet`: The agent running on worker nodes to manage pods.
  - `kubeadm`: A tool for bootstrapping a Kubernetes cluster.
  - `kubectl`: Command-line tool to manage the cluster.
  - `apt-mark hold`: Prevents automatic upgrades of these tools, ensuring cluster stability.

---

### **7. Initialize the Control Plane**

#### **Run `kubeadm init`**
```bash
sudo kubeadm init --pod-network-cidr=192.168.0.0/16
```
   - Initializes a Kubernetes control plane
   - The --pod-network-cidr=192.168.0.0/16 flag specifies the network range for the pod network (important for deploying a CNI like Calico or Flannel).

```bash
# What happens:
- Preflight checks (verifies system requirements)
- Pulls required container images
- Generates certificates for communication
- Initializes control plane components
- Creates kubeconfig file
- Generates token for joining worker nodes
```

---

#### **Configure `kubectl` Access**
```bash
mkdir -p $HOME/.kube
sudo cp /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
Purpose:
- Allows kubectl to communicate with cluster
- Stores cluster authentication details
- Sets up proper permissions
```bash
$HOME/.kube/config contains:
- Cluster information
- Authentication details
- Context settings
```
  - `admin.conf`: Configuration file generated during `kubeadm init`.
  - `mkdir -p`: Ensures the `.kube` directory exists.
  - `chown`: Changes file ownership to the current user for accessibility.

---

### **8. Install Pod Network Add-On**

#### **Apply Calico**
```bash
kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml
```
- Installs Calico network plugin
- Required for:
  - Pod-to-pod communication
  - Network policy enforcement
  - IP address management

What Calico provides:
```bash
# Network Features
- Pod networking
- Network policies
- Security rules
- IP address management
```

---

### **9. Add Worker Nodes to the Cluster**

#### **Join Command**
On each worker node, run:
```bash
sudo kubeadm join <control-plane-ip>:6443 --token <token> --discovery-token-ca-cert-hash sha256:<hash>
```
- **Purpose**: Joins the worker node to the control plane.
- **Values**:
  - `<control-plane-ip>`: IP address of the master node.
  - `<token>`: Authentication token generated during `kubeadm init`.
  - `<hash>`: Hash for verifying the control plane's certificate.

To regenerate the join command:
```bash
kubeadm token create --print-join-command
```

---

### **10. Verify the Cluster**

#### **Check Nodes**
```bash
kubectl get nodes
```
- **Purpose**: Lists all nodes in the cluster. The output should show:
  - Master node (`control-plane`).
  - Worker nodes.

#### **Check System Pods**
```bash
kubectl get pods -n kube-system
```
- **Purpose**: Verifies that Kubernetes system components (like `kube-dns`, `etcd`) are running.

---

### **11. Deploy a Sample Application**

#### **Create a Deployment**
```bash
kubectl create deployment nginx --image=nginx
```
- **Purpose**: Creates a deployment with one replica of the `nginx` web server.

#### **Expose the Deployment**
```bash
kubectl expose deployment nginx --type=NodePort --port=80
```
- **Purpose**: Exposes the `nginx` deployment as a service on a specific port.
  - `--type=NodePort`: Exposes the service on a port accessible from the cluster's nodes.

#### **Access the Application**
```bash
kubectl get svc nginx
```
- **Purpose**: Displays the NodePort assigned to the service. Access the application via:
  ```
  http://<worker-node-ip>:<node-port>
  ```

---

### **12. (Optional) Scaling the Deployment**
```bash
kubectl scale deployment nginx --replicas=3
```
- **Purpose**: Increases the number of `nginx` pods to handle more traffic.

---
