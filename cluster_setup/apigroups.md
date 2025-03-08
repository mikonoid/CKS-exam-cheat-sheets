
---

# **Kubernetes API Groups Cheat Sheet for CKS Exam**

## **1. What are Kubernetes API Groups?**
- Kubernetes API is **organized into API groups** to manage different sets of resources.
- API groups **help structure APIs** and enable **versioning**.
- Each API group is accessed via **`apis/<group>/<version>`** in the Kubernetes API server.

---

## **2. API Group Structure**
API groups follow this format:
```
/apis/<api-group>/<version>
```
For example:
```
/apis/apps/v1
/apis/rbac.authorization.k8s.io/v1
/apis/networking.k8s.io/v1
```

Some **core Kubernetes APIs** do not have an API group:
```
/api/v1
```
> **Example**: `Pods`, `Services`, `ConfigMaps`, and `Secrets` belong to the **core** API (`api/v1`).

---

## **3. Key API Groups in Kubernetes**
| API Group | Example Resource | API Endpoint |
|-----------|----------------|--------------|
| `""` (core) | Pods, Services, ConfigMaps | `/api/v1` |
| `apps` | Deployments, StatefulSets, DaemonSets | `/apis/apps/v1` |
| `batch` | Jobs, CronJobs | `/apis/batch/v1` |
| `rbac.authorization.k8s.io` | Roles, ClusterRoles, RoleBindings | `/apis/rbac.authorization.k8s.io/v1` |
| `networking.k8s.io` | NetworkPolicies, Ingress | `/apis/networking.k8s.io/v1` |
| `policy` | PodSecurityPolicies (deprecated) | `/apis/policy/v1` |
| `storage.k8s.io` | StorageClasses, CSI drivers | `/apis/storage.k8s.io/v1` |
| `apiextensions.k8s.io` | CustomResourceDefinitions (CRDs) | `/apis/apiextensions.k8s.io/v1` |
| `admissionregistration.k8s.io` | ValidatingWebhookConfigurations | `/apis/admissionregistration.k8s.io/v1` |

---

## **4. Checking API Groups in a Cluster**
Use `kubectl` to list available API groups:
```bash
kubectl api-versions
```
This lists all active API groups and their versions.

List all available API resources and their groups:
```bash
kubectl api-resources
```

Filter by a specific API group:
```bash
kubectl api-resources --api-group=apps
```

Check if a specific API resource exists:
```bash
kubectl explain deployment --api-version=apps/v1
```

---

## **5. Managing API Groups in RBAC**
RBAC rules use **API groups** to define permissions.

**Example: Role allowing read-only access to Pods (core API)**:
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: default
  name: pod-reader
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list", "watch"]
```
> `apiGroups: [""]` means **core API (api/v1)**.

---

**Example: ClusterRole allowing modifications to Deployments (apps API group)**:
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: deployment-editor
rules:
- apiGroups: ["apps"]
  resources: ["deployments"]
  verbs: ["get", "list", "update", "patch", "delete"]
```
> `apiGroups: ["apps"]` refers to the **apps/v1** API group.

---

## **6. Enforcing Security on API Groups**
### **🔹 Best Practices for Secure API Access**
✅ **Limit RBAC permissions**:  
- Do **not grant cluster-wide permissions** unless necessary.
- Use **Roles instead of ClusterRoles** when possible.

✅ **Control access to Custom Resources (CRDs)**:  
- Avoid giving users permissions to **`apiextensions.k8s.io`** unless needed.
- Example: **Restricting CRD modifications**:
  ```yaml
  apiVersion: rbac.authorization.k8s.io/v1
  kind: ClusterRole
  metadata:
    name: crd-restrict
  rules:
  - apiGroups: ["apiextensions.k8s.io"]
    resources: ["customresourcedefinitions"]
    verbs: ["get", "list"]
  ```

✅ **Secure network access to the API Server**:  
- Use **NetworkPolicies** to restrict API access.
- Example: **Restrict access to the API Server**:
  ```yaml
  apiVersion: networking.k8s.io/v1
  kind: NetworkPolicy
  metadata:
    name: api-server-restrict
    namespace: kube-system
  spec:
    podSelector:
      matchLabels:
        component: kube-apiserver
    ingress:
    - from:
      - podSelector:
          matchLabels:
            role: trusted
  ```

✅ **Monitor API usage with audit logs**:  
Enable **audit logging** to track API calls:
```yaml
apiVersion: audit.k8s.io/v1
kind: Policy
rules:
- level: Metadata
  resources:
  - group: "*"
    resources: ["*"]
```

✅ **Disable unused API groups**:  
- If certain API groups are not needed, **disable them via the API server**:
  ```bash
  kube-apiserver --runtime-config=apiextensions.k8s.io/v1=false
  ```

---

## **7. Common Commands for API Groups**

| Command | Description |
|---------|------------|
| `kubectl api-versions` | List available API groups and versions. |
| `kubectl api-resources` | Show all API resources and their groups. |
| `kubectl explain <resource> --api-version=<group/version>` | Describe a resource in a specific API group. |
| `kubectl get --raw /apis` | Get all available API groups in raw format. |
| `kubectl auth can-i <verb> <resource> --api-group=<group>` | Check RBAC permissions for an API group. |

Example: Check if a user can create deployments:
```bash
kubectl auth can-i create deployments --api-group=apps
```

---

## **8. Troubleshooting API Group Issues**
🔍 **Check if an API resource is available**
```bash
kubectl explain <resource>
```

🔍 **Verify API access with RBAC**
```bash
kubectl auth can-i get pods --api-group=""
kubectl auth can-i list deployments --api-group=apps
```

🔍 **List disabled API groups**
```bash
kubectl get --raw /apis | jq .
```

🔍 **Check API server logs for errors**
```bash
kubectl logs -n kube-system -l component=kube-apiserver
```

---

## **9. Example: Custom API Group with CRDs**
Kubernetes allows defining **Custom Resource Definitions (CRDs)**, which create new API groups.

Example: Creating a **CRD for "Database" resources**:
```yaml
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: databases.mycompany.com
spec:
  group: mycompany.com
  versions:
  - name: v1
    served: true
    storage: true
  scope: Namespaced
  names:
    plural: databases
    singular: database
    kind: Database
```
After applying:
```bash
kubectl apply -f database-crd.yaml
kubectl get crd
kubectl get databases.mycompany.com
```

---

### **10. Summary**
- Kubernetes APIs are structured into **API groups**.
- Some resources belong to the **core API (`api/v1`)**, while others have specific **API groups (`apis/<group>/v1`)**.
- Use **RBAC** to secure access to API groups.
- Audit API usage with **logs and policies**.
- Restrict access to API groups **with NetworkPolicies and RBAC**.
- **Disable unused API groups** for better security.

