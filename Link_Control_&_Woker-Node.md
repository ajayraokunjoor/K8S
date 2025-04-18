
# Setting Up kubectl on a Kubernetes Worker Node (Ubuntu)

This guide outlines the steps to configure `kubectl` on a Kubernetes worker node running Ubuntu, with explanations for each step.

## Prerequisites
- Ubuntu 22.04 LTS on the worker node.
- Network access to the control plane’s API server (port 6443).
- Access to the control node’s kubeconfig file.

## Steps

1. **Install kubectl**  
   ```bash
   sudo apt update
   sudo apt install -y kubectl
   ```  
   **Why**: `kubectl` is the Kubernetes command-line tool for managing the cluster (e.g., deploying apps, checking node status). Installing it enables local cluster management on the worker node.

2. **Copy kubeconfig from control node**  
   - On the control node, locate `/etc/kubernetes/admin.conf` or `~/.kube/config`.  
   - Transfer to the worker node using `scp`:  
     ```bash
     scp /etc/kubernetes/admin.conf user@worker-node:/home/user/.kube/config
     ```  
   **Why**: The kubeconfig file contains the API server address and authentication credentials. Copying it ensures the worker node can securely connect to the cluster.

3. **Set permissions**  
   ```bash
   mkdir -p ~/.kube
   sudo chown $(whoami):$(whoami) ~/.kube/config
   ```  
   **Why**: The transferred kubeconfig file may have incorrect ownership, causing permission errors. Setting the correct owner allows the user to access it securely.

4. **Verify kubectl**  
   ```bash
   kubectl version
   kubectl get nodes
   ```  
   **Why**: These commands confirm `kubectl` is installed, configured, and can connect to the cluster. `kubectl version` checks client/server versions, and `kubectl get nodes` lists cluster nodes, validating connectivity.

5. **Optional: Set KUBECONFIG**  
   ```bash
   export KUBECONFIG=~/.kube/config
   ```  
   - Persist in `~/.bashrc`:  
     ```bash
     echo 'export KUBECONFIG=~/.kube/config' >> ~/.bashrc
     ```  
   **Why**: The `KUBECONFIG` environment variable tells `kubectl` where to find the kubeconfig file. Setting it ensures consistent behavior, and persisting it saves time in future sessions.

## Notes
- Ensure the worker node can reach the control plane’s API server (port 6443).
- These steps enable the worker node to manage the Kubernetes cluster locally, providing flexibility for administrators.

