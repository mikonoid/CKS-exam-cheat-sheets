# Data Plane Isolation

## What is Data Plane Isolation?
Data Plane Isolation ensures that Kubernetes worker nodes and workloads are secured to prevent unauthorized access, lateral movement, and privilege escalation.

## Key Components of the Data Plane
- **Kubelet**: Manages container execution on worker nodes.
- **Container Runtime**: Runs containers (containerd, CRI-O, Docker).
- **CNI Plugins**: Handles networking (Calico, Cilium, Flannel).
- **Pods & Workloads**: User applications running in containers.

## Best Practices for Data Plane Isolation
### 1. **Harden Worker Nodes**
- Use a **minimal and hardened OS** (CIS benchmarked images).
- Disable unnecessary services:
  ```sh
  systemctl disable --now ufw firewalld
  ```
- Restrict SSH access to nodes (`AllowUsers` in `/etc/ssh/sshd_config`).
- Ensure nodes are **not publicly accessible**.

### 2. **Secure the Kubelet API**
- Restrict anonymous access:
  ```sh
  --anonymous-auth=false
  ```
- Enable TLS for communication (`--tls-cert-file`, `--tls-private-key-file`).
- Enforce authentication and authorization (`--authentication-token-webhook`, `--authorization-mode=Webhook`).
- Limit kubelet permissions using RBAC:
  ```yaml
  apiVersion: rbac.authorization.k8s.io/v1
  kind: ClusterRoleBinding
  metadata:
    name: kubelet-auth-binding
  roleRef:
    kind: ClusterRole
    name: system:kubelet-api-admin
    apiGroup: rbac.authorization.k8s.io
  subjects:
    - kind: User
      name: system:kubelet-api-user
  ```

### 3. **Enforce Network Policies**
- Restrict pod-to-pod communication using **NetworkPolicies**:
  ```yaml
  apiVersion: networking.k8s.io/v1
  kind: NetworkPolicy
  metadata:
    name: deny-all
  spec:
    podSelector: {}
    policyTypes:
      - Ingress
      - Egress
  ```
- Use **Cilium/Calico** for advanced security (e.g., eBPF-based enforcement).
- Block access to control plane services from worker nodes.

### 4. **Pod and Container Hardening**
- Enforce **Pod Security Standards (PSS)** or **Pod Security Admission (PSA)**.
- Use **seccomp**, **AppArmor**, or **SELinux**:
  ```yaml
  apiVersion: v1
  kind: Pod
  metadata:
    name: secure-pod
  spec:
    securityContext:
      seccompProfile:
        type: RuntimeDefault
  ```
- Drop unnecessary Linux capabilities:
  ```yaml
  securityContext:
    capabilities:
      drop:
        - ALL
  ```
- Use **read-only root filesystems**:
  ```yaml
  securityContext:
    readOnlyRootFilesystem: true
  ```

### 5. **Restrict Privileged Access**
- **Disable privileged containers** (`allowPrivilegeEscalation: false`).
- Block host namespace sharing:
  ```yaml
  securityContext:
    hostPID: false
    hostIPC: false
    hostNetwork: false
  ```
- Restrict volume mounts (`readonly` for sensitive directories).

### 6. **Monitor and Audit the Data Plane**
- Enable Kubernetes **Audit Logs**:
  ```yaml
  apiVersion: audit.k8s.io/v1
  kind: Policy
  rules:
    - level: Metadata
      resources:
        - group: ""
          resources: ["pods"]
  ```
- Deploy **Falco** for real-time threat detection.
- Use **Prometheus/Grafana** to monitor node performance.

## Verifying Data Plane Isolation
- Check active security policies:
  ```sh
  kubectl get networkpolicy -A
  ```
- Verify kubelet authentication:
  ```sh
  curl -k https://localhost:10250/pods
  ```
- Test pod-to-pod isolation:
  ```sh
  kubectl exec -it <pod> -- nc -zv <other-pod-IP> 80
  ```

## Summary
✅ Harden worker nodes and disable unnecessary services
✅ Secure the Kubelet API with authentication and RBAC
✅ Enforce NetworkPolicies to restrict pod communication
✅ Harden pods with seccomp, AppArmor, and minimal privileges
✅ Monitor and audit the data plane with logs and real-time detection

---
**Reference**: [Kubernetes Security Best Practices](https://kubernetes.io/docs/concepts/security/overview/)
