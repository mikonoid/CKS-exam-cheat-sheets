Here’s a **Service Accounts Cheat Sheet** for the **CKS Exam**, covering key concepts, security best practices, and commands to manage Kubernetes service accounts effectively.  

---

# **Service Accounts Cheat Sheet for CKS Exam**

## **1. What is a Service Account?**
A **ServiceAccount** (SA) in Kubernetes is an identity for processes running inside pods, enabling them to interact with the API server.  
By default:
- Every **pod** runs under the `default` service account in its namespace unless another is specified.
- Service accounts are used for **RBAC** to control pod permissions.

---

## **2. Service Account Resources**
Kubernetes provides three key resources for service accounts:

| Resource  | Description |
|-----------|------------|
| `ServiceAccount` | Defines an account that can be used by a pod. |
| `Secret` | Stores API tokens for a service account (deprecated in newer versions). |
| `TokenRequest` | Generates short-lived tokens for service accounts. |

---

## **3. Creating and Using a Service Account**

### **3.1 Create a Service Account**
```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: custom-sa
  namespace: default
```
Apply the YAML:
```bash
kubectl apply -f service-account.yaml
```
Check the service account:
```bash
kubectl get serviceaccount custom-sa -n default
```

---

### **3.2 Assign a Service Account to a Pod**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: test-pod
  namespace: default
spec:
  serviceAccountName: custom-sa
  containers:
  - name: nginx
    image: nginx
```
> 🚨 **Pods automatically inherit the default service account if `serviceAccountName` is not specified.**

---

### **3.3 Binding Service Account to RBAC**
To allow a pod running with `custom-sa` to get pods:

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: default
  name: pod-reader
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: pod-reader-binding
  namespace: default
subjects:
- kind: ServiceAccount
  name: custom-sa
  namespace: default
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```
Apply the YAML:
```bash
kubectl apply -f role.yaml
```
Check permissions:
```bash
kubectl auth can-i list pods --as=system:serviceaccount:default:custom-sa -n default
```

---

## **4. Managing Service Account Tokens**

### **4.1 Check Secrets Linked to a Service Account**
```bash
kubectl get secrets -n default
```
Newer Kubernetes versions use **TokenRequest API** instead of secrets.

### **4.2 Create a Token for a Service Account (Newer Method)**
```bash
kubectl create token custom-sa -n default
```

### **4.3 Manually Generate a Service Account Token (Deprecated)**
```bash
kubectl describe serviceaccount custom-sa -n default
kubectl get secret <sa-secret-name> -o jsonpath='{.data.token}' | base64 --decode
```

---

## **5. Service Account Security Best Practices**

✅ **Least Privilege Principle**  
- Assign the **minimum required permissions** using **RBAC**.  

✅ **Disable Auto-Mounting of Service Account Tokens**  
- By default, every pod gets a service account token, which can be a security risk.
- Disable auto-mounting:
  ```yaml
  apiVersion: v1
  kind: ServiceAccount
  metadata:
    name: no-token-sa
    namespace: default
  automountServiceAccountToken: false
  ```
  - To disable auto-mounting in a pod:
  ```yaml
  apiVersion: v1
  kind: Pod
  metadata:
    name: secure-pod
    namespace: default
  spec:
    serviceAccountName: default
    automountServiceAccountToken: false
    containers:
    - name: nginx
      image: nginx
  ```

✅ **Use Short-Lived Tokens (TokenRequest API)**  
```bash
kubectl create token custom-sa -n default
```
> 🔹 **This generates a temporary token instead of relying on secret-based tokens.**

✅ **Rotate Credentials Regularly**  
- Use Kubernetes **Projected Service Account Tokens** to enable auto-rotation.

✅ **Restrict API Server Access**  
- Apply **NetworkPolicies** to restrict communication between pods and the Kubernetes API server.

✅ **Avoid Using `cluster-admin` for Service Accounts**  
- Use **specific RBAC rules** instead of granting broad access.

✅ **Audit Service Account Usage**  
- Monitor logs for excessive permissions or unusual API requests.

---

## **6. Common Commands for Service Accounts**

| Command | Description |
|---------|------------|
| `kubectl get serviceaccounts` | List service accounts in the current namespace. |
| `kubectl describe serviceaccount <sa-name>` | Show details of a service account. |
| `kubectl create serviceaccount <sa-name>` | Create a service account. |
| `kubectl delete serviceaccount <sa-name>` | Delete a service account. |
| `kubectl get secrets | grep <sa-name>` | Get secrets associated with a service account. |
| `kubectl get pod <pod-name> -o jsonpath='{.spec.serviceAccountName}'` | Check which service account is assigned to a pod. |
| `kubectl create token <sa-name> -n <namespace>` | Generate a short-lived token. |
| `kubectl auth can-i <verb> <resource> --as=system:serviceaccount:<namespace>:<sa-name>` | Test permissions for a service account. |

---

## **7. Troubleshooting Service Account Issues**

### **🔍 Check if the Pod is Using the Correct Service Account**
```bash
kubectl get pod <pod-name> -o jsonpath='{.spec.serviceAccountName}'
```

### **🔍 Verify If a Service Account Has the Correct Permissions**
```bash
kubectl auth can-i get pods --as=system:serviceaccount:default:custom-sa
```

### **🔍 Check Pod Logs for Authentication Errors**
```bash
kubectl logs <pod-name>
```

### **🔍 Debug a Service Account Token Inside a Pod**
```bash
kubectl exec -it <pod-name> -- cat /var/run/secrets/kubernetes.io/serviceaccount/token
```

---

## **8. Example: Secure Service Account with Network Policy**
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-all-except-api
  namespace: default
spec:
  podSelector:
    matchLabels:
      run: restricted-pod
  policyTypes:
  - Egress
  egress:
  - to:
    - ipBlock:
        cidr: 10.0.0.1/32  # API Server IP
    ports:
    - protocol: TCP
      port: 443
```
This policy **restricts pod egress traffic**, only allowing access to the API server.

---

## **9. Service Account vs User Authentication**
| Feature | Service Account | User Account |
|---------|----------------|-------------|
| Use Case | Used by pods and controllers | Used by human users |
| Credentials | Managed by Kubernetes | Managed by IAM/Authentication provider |
| Token Type | JWT-based service account token | User-generated token |
| Expiry | Can be short-lived (`TokenRequest`) | Varies (based on OIDC or external auth) |

---

### **References**

 - https://kubernetes.io/docs/concepts/security/service-accounts/
 - https://itnext.io/cks-exam-series-7-serviceaccount-tokens-1158c93612d4
