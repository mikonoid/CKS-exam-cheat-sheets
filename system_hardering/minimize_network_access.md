# CKS Cheat Sheet: Minimize External Access to the Network

## 1. Use UFW (Uncomplicated Firewall) to Restrict Network Access
- Install UFW if not already installed:
  ```bash
  sudo apt update && sudo apt install ufw -y
  ```
- Enable UFW and set default deny rules:
  ```bash
  sudo ufw default deny incoming
  sudo ufw default allow outgoing
  ```

## 2. Allow Essential Kubernetes Traffic
- Allow only necessary ports for Kubernetes components:
  ```bash
  sudo ufw allow 6443/tcp  # Kubernetes API server
  sudo ufw allow 2379:2380/tcp  # etcd server
  sudo ufw allow 10250/tcp  # Kubelet API
  sudo ufw allow 10259/tcp  # kube-scheduler
  sudo ufw allow 10257/tcp  # kube-controller-manager
  ```

## 3. Restrict Node-to-Node Communication
- Allow only necessary inter-node communication (e.g., Flannel, Calico, or other CNI plugins):
  ```bash
  sudo ufw allow 6783/tcp  # WeaveNet
  sudo ufw allow 8285/udp  # Flannel
  sudo ufw allow 8472/udp  # Flannel
  ```
- Block unnecessary external access:
  ```bash
  sudo ufw deny 30000:32767/tcp  # Block NodePort services
  ```

## 4. Enable Logging and Monitor Firewall Activity
- Enable logging to track denied connections:
  ```bash
  sudo ufw logging on
  ```
- Check UFW status and rules:
  ```bash
  sudo ufw status verbose
  ```

## 5. Allow Specific External Access (if required)
- Allow external access only to designated services:
  ```bash
  sudo ufw allow proto tcp from any to any port 80,443  # Allow HTTP/HTTPS
  ```

## 6. Apply and Verify Firewall Rules
- Enable UFW after configuring rules:
  ```bash
  sudo ufw enable
  ```
- Verify rules:
  ```bash
  sudo ufw status numbered
  ```
- If needed, remove rules:
  ```bash
  sudo ufw delete [rule_number]
  ```

## 7. Use UFW with Kubernetes Worker Nodes
- Restrict access to Kubernetes worker nodes:
  ```bash
  sudo ufw allow 10250/tcp  # Kubelet API
  sudo ufw allow 10255/tcp  # Read-only Kubelet API (disable in production)
  sudo ufw deny 30000:32767/tcp  # Block NodePort services
  ```

## 8. Advanced: Restrict SSH and External Access
- Allow SSH only from trusted IPs:
  ```bash
  sudo ufw allow from 192.168.1.0/24 to any port 22
  ```
- Block all other SSH access:
  ```bash
  sudo ufw deny 22
  ```

By implementing these **UFW firewall rules**, you can **minimize external access** to your Kubernetes cluster and improve security. 🚀

