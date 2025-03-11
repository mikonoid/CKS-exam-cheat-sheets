# Certified Kubernetes Security Specialist (CKS) - Auditing Cheat Sheet

## Kubernetes Auditing Overview
Kubernetes auditing provides visibility into cluster activity, helping to track API requests and detect suspicious behavior. The audit system records details such as who did what, when, and where within the cluster.

### Key Audit Components
- **Audit Policy**: Defines what events should be recorded.
- **Audit Backend**: Where the audit logs are stored (file, webhook, etc.).
- **Audit Levels**:
  - `None`: No logging.
  - `Metadata`: Logs metadata (who, what, when, etc.) but not request/response body.
  - `Request`: Logs metadata + request body.
  - `RequestResponse`: Logs metadata + request & response body.

---

## Example: Audit Policy for Monitoring Secret Deletions in `prod` Namespace
### Create the audit policy file:
```yaml
# /etc/kubernetes/prod-audit.yaml
apiVersion: audit.k8s.io/v1
kind: Policy
rules:
- level: Metadata
  namespaces: ["prod"]
  verbs: ["delete"]
  resources:
  - group: ""
    resources: ["secrets"]
```

### Configure API Server to Enable Auditing
Modify the `kube-apiserver` configuration (typically in a manifest file for static pods, e.g., `/etc/kubernetes/manifests/kube-apiserver.yaml`):
```yaml
- --audit-policy-file=/etc/kubernetes/prod-audit.yaml
- --audit-log-path=/var/log/prod-secrets.log
- --audit-log-maxage=30
```

### Add Required Volumes and Mounts
Modify the API Server pod specification to include the audit policy and log file:
```yaml
volumes:
  - name: audit
    hostPath:
      path: /etc/kubernetes/prod-audit.yaml
      type: File
  - name: audit-log
    hostPath:
      path: /var/log/prod-secrets.log
      type: FileOrCreate

volumeMounts:
  - mountPath: /etc/kubernetes/prod-audit.yaml
    name: audit
    readOnly: true
  - mountPath: /var/log/prod-secrets.log
    name: audit-log
    readOnly: false
```

### Verify Audit Logs
After applying the changes and restarting `kube-apiserver`, check the audit logs:
```bash
cat /var/log/prod-secrets.log
```

### Best Practices for Kubernetes Auditing
- **Use minimal logging** to reduce storage overhead while capturing critical actions.
- **Rotate and archive logs** using tools like `logrotate`.
- **Send logs to a centralized SIEM** for real-time monitoring and alerting.
- **Ensure audit logs are tamper-proof**, e.g., by using a dedicated logging backend.



