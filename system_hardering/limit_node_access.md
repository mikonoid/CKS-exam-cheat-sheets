# CKS Cheat Sheet: Limiting Node Access

## 1. **Use Node Restrictions Admission Controller**
Ensure that the `NodeRestriction` admission controller is enabled to prevent compromised kubelets from modifying other nodes:

```yaml
--enable-admission-plugins=NodeRestriction
```

## 2. **Restrict SSH Access to Nodes**
- **Disable direct SSH access** to worker nodes.
- Use **bastion hosts** instead of allowing SSH access to all nodes.
- Restrict SSH access with security groups or firewall rules:
  
  ```bash
  sudo ufw allow from <bastion-ip> to any port 22
  ```

## 3. **Minimize Cloud Provider IAM Permissions**
Ensure nodes have the **minimum required IAM permissions**. Example for AWS:

- Restrict EC2 IAM roles assigned to worker nodes.
- Use least privilege policies for accessing resources.

## 4. **Use Network Policies to Restrict Node Access**
Apply **NetworkPolicies** to limit pod-to-node communication:

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-node-access
  namespace: default
spec:
  podSelector: {}
  policyTypes:
  - Egress
  egress:
  - to:
    - ipBlock:
        cidr: 10.0.0.0/16  # Adjust to your cluster's node CIDR
    except:
    - 10.0.1.0/24  # Allow access to necessary control plane nodes
```

## 5. **Limit API Server Access to Nodes**
Restrict API server communication to nodes using **firewall rules** or **Security Groups**:

- Limit access to only the control plane components.
- Disable public API access if not needed.

## 6. **Harden kubelet Security**
- **Disable anonymous authentication:**
  
  ```yaml
  --anonymous-auth=false
  ```
- **Enable authentication and authorization:**
  
  ```yaml
  --authorization-mode=Webhook
  --client-ca-file=/etc/kubernetes/pki/ca.crt
  ```
- **Restrict kubelet API access:**
  
  ```yaml
  --read-only-port=0
  --protect-kernel-defaults=true
  ```

## 7. **Use RBAC to Restrict Node Access**
Restrict node access using **RBAC** roles:

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: kube-system
  name: node-reader
rules:
- apiGroups: [""]
  resources: ["nodes"]
  verbs: ["get", "list"]
```

Bind this role only to specific service accounts that require access:

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  namespace: kube-system
  name: node-reader-binding
subjects:
- kind: User
  name: node-auditor
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: node-reader
  apiGroup: rbac.authorization.k8s.io
```

## 8. **Use Pod Security Policies or Pod Security Admission**
Enforce **Pod Security Policies** (deprecated) or **Pod Security Admission** to restrict node access:

```yaml
apiVersion: policy/v1beta1
kind: PodSecurityPolicy
metadata:
  name: restricted-psp
spec:
  privileged: false
  hostNetwork: false
  hostIPC: false
  hostPID: false
```

## 9. **Enable Seccomp and AppArmor**
Harden nodes by enforcing **Seccomp** and **AppArmor**:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: secure-pod
  annotations:
    container.apparmor.security.beta.kubernetes.io/my-container: "runtime/default"
    seccomp.security.alpha.kubernetes.io/pod: "runtime/default"
