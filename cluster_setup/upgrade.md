# Kubernetes Cluster Upgrade Cheat Sheet (CKS Exam)

## **1. Pre-Upgrade Checklist** 
Before upgrading, ensure:
- **Backup etcd**: Take a snapshot of etcd (for clusters using kubeadm).
- **Check current version**:
  ```bash
  kubectl version --short
  ```
- **Review Kubernetes release notes**: Identify deprecations & breaking changes.
- **Check compatibility**:
  ```bash
  kubectl get nodes -o wide
  ```
  Ensure worker nodes and control plane are compatible with the target version.
- **Drain workload from nodes**:
  ```bash
  kubectl drain <node-name> --ignore-daemonsets --delete-emptydir-data
  ```

---

## **2. Upgrade Kubernetes Control Plane (kubeadm)**

### ** Step 1: Upgrade `kubeadm`**
Upgrade `kubeadm` on the **control plane node**:
```bash
apt update && apt install -y kubeadm=<target-version>
```
Check the available versions:
```bash
apt-cache madison kubeadm
```

### ** Step 2: Plan the Upgrade**
Run a dry-run to check issues:
```bash
kubeadm upgrade plan
```

### ** Step 3: Apply the Upgrade**
Apply the upgrade on the control plane:
```bash
kubeadm upgrade apply v<target-version>
```

### ** Step 4: Upgrade `kubectl` and `kubelet`**
Update binaries:
```bash
apt install -y kubelet=<target-version> kubectl=<target-version>
systemctl restart kubelet
```

---

## **3. Upgrade Worker Nodes**

### **🔹 Step 1: Upgrade `kubeadm` on Worker Node**
```bash
apt update && apt install -y kubeadm=<target-version>
```
Check upgrade plan:
```bash
kubeadm upgrade node
```

### ** Step 2: Drain the Node**
```bash
kubectl drain <worker-node> --ignore-daemonsets --delete-emptydir-data
```

### ** Step 3: Upgrade Kubelet & Restart**
```bash
apt install -y kubelet=<target-version>
systemctl restart kubelet
kubectl uncordon <worker-node>
```

---

## **4. Post-Upgrade Checks**
- Verify all nodes are in `Ready` state:
  ```bash
  kubectl get nodes
  ```
- Check cluster functionality:
  ```bash
  kubectl get pods -A
  ```
- Validate API Server:
  ```bash
  kubectl cluster-info
  ```
- Restart CNI (if required, for networking issues).

---

## **5. Security Considerations During Upgrade** 🔒
- **Use Immutable Container Images**: Ensure workloads run **fixed versions** of images.
- **Audit Logs**: Monitor logs for issues post-upgrade.
- **Enable RBAC Policies**: Ensure security policies remain intact.
- **Monitor Network Policies**: Verify pod-to-pod and external communication after the upgrade.
- **Verify Pod Security Standards (PSS)**: Ensure security contexts are still enforced.

---

## **6. Rolling Back an Upgrade**
If the upgrade fails:
1. **Check etcd backup** and restore if needed:
   ```bash
   ETCDCTL_API=3 etcdctl snapshot restore <snapshot-file>
   ```
2. **Revert Kubelet and Kubectl**:
   ```bash
   apt install -y kubelet=<previous-version> kubectl=<previous-version>
   systemctl restart kubelet
   ```
3. **Rejoin Worker Nodes**:
   ```bash
   kubeadm reset
   kubeadm join <control-plane-ip>:6443 --token <token> --discovery-token-ca-cert-hash sha256:<hash>
   ```

---

## **7. Summary of Commands**
| Task | Command |
|------|---------|
| Backup etcd | `ETCDCTL_API=3 etcdctl snapshot save snapshot.db` |
| Check upgrade plan | `kubeadm upgrade plan` |
| Upgrade Control Plane | `kubeadm upgrade apply vX.Y.Z` |
| Upgrade Worker Node | `kubeadm upgrade node` |
| Restart Kubelet | `systemctl restart kubelet` |
| Drain Node | `kubectl drain <node>` |
| Uncordon Node | `kubectl uncordon <node>` |
| Check Cluster Status | `kubectl get nodes` |


