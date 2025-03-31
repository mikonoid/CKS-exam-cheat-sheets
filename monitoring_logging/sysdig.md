```markdown
# Sysdig - Kubernetes Security & Monitoring

## Overview
Sysdig is a cloud-native security and monitoring tool that provides runtime threat detection, compliance enforcement, and forensic analysis for Kubernetes and containerized workloads.

## Installing Sysdig
### Install Sysdig Agent via Helm
```sh
helm repo add sysdig https://charts.sysdig.com
helm repo update
helm install sysdig-agent sysdig/sysdig --set sysdig.accessKey=YOUR_ACCESS_KEY
```

### Install Sysdig Secure for Kubernetes
```sh
helm install sysdig-secure sysdig/sysdig-secure --set sysdig.accessKey=YOUR_ACCESS_KEY
```

## Sysdig Configuration
### Config File Location
- `/etc/sysdig/sysdig.yaml` (for agent settings)
- `/etc/sysdig/rules.d/` (for Falco-compatible security rules)

### Example Configuration (sysdig.yaml)
```yaml
security:
  enabled: true
  falco_rules_file: /etc/sysdig/rules.d/custom-rules.yaml
  audit_log:
    enabled: true
```

## Writing Custom Sysdig Rules
Sysdig Secure uses Falco-compatible rules for runtime security.

### Example Rule: Detect Sensitive File Access
```yaml
- rule: Detect sensitive file access
  desc: Alert when a sensitive file is accessed inside a container
  condition: >
    (evt.type=open or evt.type=openat) and
    (fd.name startswith "/etc/shadow" or fd.name startswith "/etc/passwd")
  output: "Sensitive file access detected (user=%user.name file=%fd.name)"
  priority: CRITICAL
  tags: ["filesystem", "security"]
```

## Viewing Sysdig Alerts
### View Sysdig Secure Events
```sh
kubectl logs -l app=sysdig-agent -n sysdig
```

### View Sysdig Metrics in Kubernetes
```sh
kubectl top pods --all-namespaces
```

### Enable Sysdig Kubernetes Audit Log Monitoring
Modify `sysdig.yaml`:
```yaml
audit:
  k8s_audit_server_url: "https://your-kubernetes-api-server:443"
  enabled: true
```

## Best Practices
- **Enable Sysdig Secure** to monitor runtime security threats.
- **Integrate Sysdig with SIEM** tools like Splunk or ELK for centralized logging.
- **Define custom security rules** for Kubernetes runtime monitoring.
- **Leverage Kubernetes audit logs** to detect unauthorized API activity.
- **Enable network visibility** to analyze container-to-container traffic.

## References
- Sysdig Documentation: [https://docs.sysdig.com/](https://docs.sysdig.com/)
- Sysdig GitHub Repository: [https://github.com/draios/sysdig](https://github.com/draios/sysdig)
- Sysdig Secure Overview: [https://sysdig.com/products/secure/](https://sysdig.com/products/secure/)
- Kubernetes Audit Logging: [https://kubernetes.io/docs/tasks/debug/debug-cluster/audit/](https://kubernetes.io/docs/tasks/debug/debug-cluster/audit/)
```

