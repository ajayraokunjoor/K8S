# Setting Up a Kubernetes Cluster on Ubuntu 22.04

This guide walks through the step-by-step process of setting up a Kubernetes cluster manually on Ubuntu 22.04 (Jammy Jellyfish). It follows the practices taught in A Cloud Guru's Kubernetes course and is intended for hands-on learning.

## Infrastructure Setup
- **1 Master Node** (Ubuntu 22.04, Medium size)
- **2 Worker Nodes** (Ubuntu 22.04, Small size)

---

## Step 1: Install containerd (on all nodes)

### Disable Swap
```bash
sudo swapoff -a
```
> Kubernetes requires swap to be disabled for proper resource management.

### Load Required Kernel Modules
```bash
cat <<EOF | sudo tee /etc/modules-load.d/containerd.conf
overlay
br_netfilter
EOF

sudo modprobe overlay
sudo modprobe br_netfilter
```
> These modules are needed for networking and container support.

### Set Kernel Parameters
```bash
cat <<EOF | sudo tee /etc/sysctl.d/99-kubernetes-cri.conf
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
net.bridge.bridge-nf-call-ip6tables = 1
EOF

sudo sysctl --system
```
> These parameters ensure that traffic routing works as expected for Kubernetes networking.

### Install containerd
```bash
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg

echo \  
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \  
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \  
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt-get update && sudo apt-get install -y containerd
```
> Adds Docker's repo and installs containerd, a container runtime needed by Kubernetes.

### Configure containerd
```bash
sudo mkdir -p /etc/containerd
sudo containerd config default | sudo tee /etc/containerd/config.toml
sudo sed -i 's/SystemdCgroup = false/SystemdCgroup = true/g' /etc/containerd/config.toml
sudo systemctl restart containerd
sudo systemctl status containerd
```
> Generates default configuration, modifies it to use systemd for cgroup management, then restarts the service.

---

## Step 2: Install Kubernetes Components (on all nodes)

### Install Dependencies
```bash
sudo apt-get update && sudo apt-get install -y apt-transport-https ca-certificates curl gpg software-properties-common
```

### Add Kubernetes Repo
```bash
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.31/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
sudo chmod 644 /etc/apt/keyrings/kubernetes-apt-keyring.gpg

echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.31/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
sudo chmod 644 /etc/apt/sources.list.d/kubernetes.list

sudo apt-get update
```

### Install Kubernetes Tools
```bash
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
sudo systemctl enable --now kubelet
```
> Installs core Kubernetes tools and holds their version to prevent unintended upgrades.

---

## Step 3: Initialize Cluster (Master Node only)

### Disable Firewall (on all nodes)
```bash
sudo systemctl stop ufw
sudo ufw disable
```
> Disables firewall to prevent network issues between nodes.

### Initialize the Cluster
```bash
sudo kubeadm init --pod-network-cidr=192.168.0.0/16
```
> Initializes the control plane and sets the pod network range.

### Set Up kubectl Access
```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
> Allows the current user to use `kubectl` to manage the cluster.

---

## Step 4: Join Worker Nodes (Worker Nodes only)

### Run Join Command
Use the output from `kubeadm init`, or retrieve it:
```bash
kubeadm token create --print-join-command
```
Then run the join command:
```bash
sudo kubeadm join <IP>:6443 --token <TOKEN> --discovery-token-ca-cert-hash sha256:<HASH>
```
> Adds worker nodes to the cluster.

### Verify Nodes (Master Node)
```bash
kubectl get nodes
```
> All nodes should be listed as `NotReady` until networking is configured.

---

## Step 5: Set Up Networking (Master Node only)

### Install Calico CNI
```bash
kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.25.0/manifests/calico.yaml
```
> Installs Calico for networking between pods and nodes.

### Verify Cluster Status
```bash
kubectl get nodes
```
> Nodes should change to `Ready` status.

### Check Pod Status
```bash
kubectl get pods -n kube-system
```
> Ensure all Calico and core system pods are running.

---

## Final Notes
- This is a minimal manual setup for learning and development.
- For production, consider using TLS, RBAC, monitoring, logging, and HA setups.

Happy learning!

---

_This guide is based on A Cloud Guru's Kubernetes training._

