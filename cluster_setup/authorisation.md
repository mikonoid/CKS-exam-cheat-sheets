__
# **Authorization in Kubernetes - CKS Exam Cheat Sheet**  

## **1. What is Authorization in Kubernetes?**
- Authorization **determines what a user or service can do** in a Kubernetes cluster.
- It happens **after authentication** and **before request execution**.
- Kubernetes supports **multiple authorization modes** that can be used together.

---

## **2. Authorization Modes in Kubernetes**
| **Mode** | **Description** | **Usage** |
|----------|----------------|-----------|
| **RBAC (Role-Based Access Control)** | Uses roles and role bindings to control access. | **Recommended for security** |
| **ABAC (Attribute-Based Access Control)** | Uses JSON policy files to define access rules. | Deprecated, **not recommended** |
| **Webhook Authorization** | Delegates authorization to an external service. | Used in custom integrations |
| **Node Authorization** | Limits kubelet access to resources on its node. | Used for securing kubelets |

🔹 **Enable Authorization Modes in API Server**
```yaml
--authorization-mode=RBAC,Node
```
✅ **RBAC should always be enabled for security.**

---

## **3. RBAC (Role-Based Access Control) - The Most Secure Option**
### **🔹 Check Current RBAC Roles and Bindings**
```bash
kubectl get roles,rolebindings,clusterroles,clusterrolebindings -A
```

### **🔹 RBAC Key Objects**
| **Object** | **Scope** | **Purpose** |
|------------|----------|-------------|
| **Role** | Namespace | Defines permissions within a namespace. |
| **ClusterRole** | Cluster-wide | Defines permissions across all namespaces. |
| **RoleBinding** | Namespace | Assigns a **Role** to a user, group, or service account. |
| **ClusterRoleBinding** | Cluster-wide | Assigns a **ClusterRole** to a user, group, or service account. |

---

## **4. Create and Apply RBAC Rules**
### **🔹 Create a Role for Read-Only Access to Pods**
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: default
  name: read-pods
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list"]
```

### **🔹 Bind the Role to a User**
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-pods-binding
  namespace: default
subjects:
- kind: User
  name: user1
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: read-pods
  apiGroup: rbac.authorization.k8s.io
```

✅ **Now `user1` can only read pods in the `default` namespace.**

---

## **5. Using ClusterRoles for Global Access**
### **🔹 Create a ClusterRole for Admin Access**
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: cluster-admin
rules:
- apiGroups: ["*"]
  resources: ["*"]
  verbs: ["*"]
```

### **🔹 Bind the ClusterRole to a User**
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: cluster-admin-binding
subjects:
- kind: User
  name: admin-user
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: cluster-admin
  apiGroup: rbac.authorization.k8s.io
```

✅ **This grants `admin-user` full access to the cluster.** ⚠️ **Use carefully!**

---

## **6. Check User Permissions**
### **🔹 Verify What Actions a User Can Perform**
```bash
kubectl auth can-i create pods --as=user1
```
✅ **Expected Output:** `no` (If user lacks permission)

### **🔹 Test a User’s Permissions Across the Cluster**
```bash
kubectl auth can-i list nodes --as=user1 --all-namespaces
```
✅ **Expected Output:** `no` (User has no cluster-wide access)

---

## **7. Restrict Access Using Least Privilege Principle**
### **🔹 Grant Read-Only Access to Services**
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: default
  name: read-services
rules:
- apiGroups: [""]
  resources: ["services"]
  verbs: ["get", "list"]
```
✅ **Users should only have permissions necessary for their role.**

---

## **8. Secure Kubernetes API with Webhook Authorization**
### **🔹 Configure API Server to Use Webhook Authorization**
```yaml
--authorization-mode=Webhook
--authorization-webhook-config-file=/etc/kubernetes/webhook-config.yaml
```

Example Webhook Configuration:
```yaml
apiVersion: v1
kind: Config
clusters:
- name: webhook-authz
  cluster:
    server: https://authz-webhook.example.com/authorize
users:
- name: webhook-authz
contexts:
- name: webhook-authz-context
  context:
    cluster: webhook-authz
    user: webhook-authz
current-context: webhook-authz-context
```
✅ **Useful for integrating with external IAM systems.**

---

## **9. Special Authorization Mode: Node Authorization**
- Used **only for kubelets**.
- Ensures **kubelets can only access resources on their own node**.
- Enable in the API server:
```yaml
--authorization-mode=Node
```
✅ **Prevents kubelets from unauthorized access.**

---

## **10. Restrict Access to Secrets**
### **🔹 Limit Who Can Access Secrets**
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: default
  name: restrict-secrets
rules:
- apiGroups: [""]
  resources: ["secrets"]
  verbs: ["get"]
```
✅ **Only allow specific users or pods to access secrets.**

### **🔹 Verify If a User Can Access Secrets**
```bash
kubectl auth can-i get secrets --as=user1
```
✅ **Expected Output:** `no`

---

## **11. Auditing Authorization Events**
🔹 **Enable Kubernetes Audit Logs**
```yaml
--audit-log-path=/var/log/kubernetes/audit.log
--audit-policy-file=/etc/kubernetes/audit-policy.yaml
```

🔹 **Example Audit Policy to Log Authorization Events**
```yaml
apiVersion: audit.k8s.io/v1
kind: Policy
rules:
- level: Metadata
  resources:
  - group: "*"
    resources: ["*"]
```

🔹 **Check Authorization Failures in Logs**
```bash
cat /var/log/kubernetes/audit.log | grep -i "forbidden"
```
✅ **Helps detect unauthorized access attempts.**

---

## **12. Test and Verify Authorization Controls**
| **Test** | **Command** | **Expected Result** |
|----------|------------|---------------------|
| Verify if a user can delete pods | `kubectl auth can-i delete pods --as=user1` | `no` |
| Check if a user has admin access | `kubectl auth can-i '*' '*' --as=admin-user` | `yes` |
| Test if a service account can list nodes | `kubectl auth can-i list nodes --as=system:serviceaccount:default:sa-user` | `no` |
| Review audit logs for failed authorization requests | `cat /var/log/kubernetes/audit.log | grep forbidden` | Shows failed attempts |

---

## **13. Summary**
✅ **Use RBAC for secure authorization (`--authorization-mode=RBAC`)**  
✅ **Follow the least privilege principle** when assigning roles  
✅ **Limit access to secrets and critical resources**  
✅ **Use Webhook Authorization for external IAM integration**  
✅ **Enable audit logging to track authorization events**  
