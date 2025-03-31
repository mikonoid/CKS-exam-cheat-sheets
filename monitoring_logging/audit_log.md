```markdown
# Kubernetes Audit Logs

## Overview
Kubernetes audit logging records cluster activity, helping detect security threats, unauthorized access, and compliance violations.

## Enabling Audit Logging
Audit logging must be enabled in the Kubernetes API server.

### Modify API Server Config
1. Edit the API server manifest (e.g., `/etc/kubernetes/manifests/kube-apiserver.yaml`).
2. Add these flags:
```yaml
--audit-policy-file=/etc/kubernetes/audit-policy.yaml
--audit-log-path=/var/log/kubernetes/audit.log
--audit-log-maxage=30
--audit-log-maxbackup=10
--audit-log-maxsize=100
```

3. Restart the API server.

## Defining an Audit Policy
Audit policies define what events to log and at what level.

### Example Audit Policy (`audit-policy.yaml`)
```yaml
apiVersion: audit.k8s.io/v1
kind: Policy
rules:
  - level: Metadata # Log request metadata (e.g., user, resource, action)
    verbs: ["create", "delete", "patch", "update"]
    resources:
      - group: ""
        resources: ["pods", "deployments", "secrets"]
  - level: RequestResponse # Log full request and response
    users: ["system:serviceaccount:kube-system:kube-proxy"]
    verbs: ["get", "list"]
```

## Viewing Audit Logs
### Inspect the Audit Log File
```sh
cat /var/log/kubernetes/audit.log | jq .
```

### Filter Logs for Security Events
```sh
cat /var/log/kubernetes/audit.log | jq 'select(.user.username == "system:anonymous")'
```

## Sending Audit Logs to External Systems
### Forward Logs to Fluentd
Modify `audit-policy.yaml`:
```yaml
- level: Request
  sink: fluentd
```

### Send Logs to Sysdig or Falco
Configure Sysdig/Falco to monitor `/var/log/kubernetes/audit.log`.

## Best Practices
- **Use a strict audit policy** to capture relevant security events.
- **Forward logs** to a central SIEM (e.g., ELK, Splunk) for analysis.
- **Monitor anonymous and privileged access** to detect threats.
- **Rotate logs** to prevent excessive disk usage.
- **Automate alerts** for suspicious API activity.

## References
- Kubernetes Audit Logging Docs: [https://kubernetes.io/docs/tasks/debug/debug-cluster/audit/](https://kubernetes.io/docs/tasks/debug/debug-cluster/audit/)
- Fluentd Kubernetes Logging: [https://docs.fluentd.org/input/kubernetes](https://docs.fluentd.org/input/kubernetes)
- Kubernetes Security Best Practices: [https://kubernetes.io/docs/concepts/security/overview/](https://kubernetes.io/docs/concepts/security/overview/)
```

