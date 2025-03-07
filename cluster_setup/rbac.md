
---

### **RBAC Cheat Sheet for CKS Exam**

#### **RBAC Key Concepts**

- **RBAC (Role-Based Access Control)** controls access to resources based on the roles assigned to users or service accounts.
- **Roles** define what actions can be performed on what resources.
- **RoleBindings** and **ClusterRoleBindings** associate roles with subjects (users or service accounts).

---

#### **RBAC Resources**

1. **Roles**
   - **Namespace-level resource**.
   - Defines access control within a specific namespace.
   - Can be used with `RoleBinding` to grant permissions.

   Example:
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
   ```

2. **ClusterRoles**
   - **Cluster-wide resource**.
   - Defines access control across the entire cluster (e.g., access to nodes, cluster-level resources).
   - Can be used with `ClusterRoleBinding` to grant permissions.

   Example:
   ```yaml
   apiVersion: rbac.authorization.k8s.io/v1
   kind: ClusterRole
   metadata:
     # Name of the ClusterRole
     name: cluster-admin
   rules:
   - apiGroups: [""]
     resources: ["pods", "services"]
     verbs: ["get", "list", "create", "delete"]
   ```

3. **RoleBindings**
   - Associates a **Role** with specific users or service accounts **within a namespace**.
   - A **RoleBinding** grants the permissions defined in a Role to users, service accounts, or groups.

   Example:
   ```yaml
   apiVersion: rbac.authorization.k8s.io/v1
   kind: RoleBinding
   metadata:
     name: pod-reader-binding
     namespace: default
   subjects:
   - kind: User
     name: "janedoe"
     apiGroup: rbac.authorization.k8s.io
   roleRef:
     kind: Role
     name: pod-reader
     apiGroup: rbac.authorization.k8s.io
   ```

4. **ClusterRoleBindings**
   - Associates a **ClusterRole** with specific users or service accounts **across the entire cluster**.
   - A **ClusterRoleBinding** grants the permissions defined in a ClusterRole.

   Example:
   ```yaml
   apiVersion: rbac.authorization.k8s.io/v1
   kind: ClusterRoleBinding
   metadata:
     name: cluster-admin-binding
   subjects:
   - kind: ServiceAccount
     name: default
     namespace: kube-system
   roleRef:
     kind: ClusterRole
     name: cluster-admin
     apiGroup: rbac.authorization.k8s.io
   ```

---

#### **RBAC Permissions (Verbs)**

These verbs define what actions are allowed on resources:

- **create**: Create a resource.
- **get**: Retrieve a resource.
- **list**: List resources.
- **update**: Modify a resource.
- **patch**: Partially modify a resource.
- **delete**: Delete a resource.
- **deletecollection**: Delete a collection of resources.
- **watch**: Watch for changes to a resource.
- **bind**: Used for binding users to roles.

---

#### **RBAC Best Practices**

1. **Principle of Least Privilege (PoLP)**
   - Grant the **minimum permissions** required for a user or service account to perform their tasks.
   - Avoid assigning **cluster-admin** roles unless absolutely necessary.

2. **Use Namespaced Roles When Possible**
   - Prefer using **Roles** over **ClusterRoles** when limiting access to a specific namespace.

3. **Use ServiceAccounts for Pods**
   - Assign specific **ServiceAccounts** to pods to enforce RBAC policies.

4. **Review and Audit RBAC Permissions Regularly**
   - Use tools like **kubectl auth can-i** to audit permissions.

   Example to check if a user has a specific permission:
   ```bash
   kubectl auth can-i get pods --as janedoe
   ```

---

#### **Common Commands for RBAC**

1. **List Roles and ClusterRoles**:
   ```bash
   kubectl get roles
   kubectl get clusterroles
   ```

2. **List RoleBindings and ClusterRoleBindings**:
   ```bash
   kubectl get rolebindings
   kubectl get clusterrolebindings
   ```

3. **Create a Role or ClusterRole**:
   ```bash
   kubectl apply -f role.yaml
   kubectl apply -f clusterrole.yaml
   ```

4. **Create a RoleBinding or ClusterRoleBinding**:
   ```bash
   kubectl apply -f rolebinding.yaml
   kubectl apply -f clusterrolebinding.yaml
   ```

5. **Check Permissions for a User or Service Account**:
   ```bash
   kubectl auth can-i <verb> <resource> --as <user>
   ```

   Example:
   ```bash
   kubectl auth can-i get pods --as janedoe
   ```

6. **Describe a Role or RoleBinding**:
   ```bash
   kubectl describe role <role-name> -n <namespace>
   kubectl describe rolebinding <rolebinding-name> -n <namespace>
   ```

---

#### **RBAC Troubleshooting Tips**

1. **Check if a user has the correct permissions**:
   - Use `kubectl auth can-i` to check what actions a user can perform on specific resources.
   - Example: Check if a user can `get` pods:
     ```bash
     kubectl auth can-i get pods --as <username>
     ```

2. **Review Resource Access Logs**:
   - Ensure proper logging is enabled to track access to Kubernetes resources.

3. **Debugging RBAC Issues**:
   - If a user or service account can't access resources, check for missing or incorrect RoleBindings/ClusterRoleBindings.
   - Ensure the correct API group is used in the `roleRef`.

---

#### **RBAC Security Considerations**

1. **Avoid Wildcards**
   - Avoid using wildcards (`*`) in roles (e.g., `verbs: ["*"]`), as it grants broad access.
   - Instead, specify only the required resources and verbs.

2. **Use Namespaces for Isolation**
   - Limit permissions to specific namespaces to prevent cross-namespace access.

3. **Review RBAC with Regular Audits**
   - Continuously monitor roles and bindings to ensure users have only the permissions they need.

---

### **RBAC Example: Pod Reader Role**

Create a **Role** to allow a user to read `pods` in the `default` namespace.

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
```

Bind the **Role** to a specific user (`janedoe`):

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: pod-reader-binding
  namespace: default
subjects:
- kind: User
  name: "janedoe"
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

---

