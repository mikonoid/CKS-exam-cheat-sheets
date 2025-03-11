# Certified Kubernetes Security Specialist (CKS) - kubectl Proxy & Port Forwarding Cheat Sheet

## Overview
Kubernetes provides mechanisms to access cluster services securely using `kubectl proxy` and `kubectl port-forward`. These methods help in debugging, accessing internal services, and securely routing traffic.

---

## `kubectl proxy`
`kubectl proxy` starts a local HTTP proxy that allows access to the Kubernetes API server from your local machine.

### Syntax:
```bash
kubectl proxy --address=0.0.0.0 --port=8001 --accept-hosts='.*'
```

### Options:
- `--port=8001` → Port to expose the API.
- `--address=0.0.0.0` → Allows external connections.
- `--accept-hosts='.*'` → Accepts requests from any host.

### Example:
To access the Kubernetes API:
```bash
curl http://127.0.0.1:8001/api/v1/namespaces/default/pods
```

### Security Considerations:
- Limit access by binding to `127.0.0.1`.
- Use RBAC to restrict API access.
- Use a firewall or VPN for secure access.

---

## `kubectl port-forward`
`kubectl port-forward` forwards traffic from a local port to a port on a pod, allowing access to internal services.

### Syntax:
```bash
kubectl port-forward <pod-name> <local-port>:<pod-port>
```

### Example:
Forward traffic from local port 8080 to port 80 on the pod `my-pod`:
```bash
kubectl port-forward my-pod 8080:80
```

To forward a service:
```bash
kubectl port-forward svc/my-service 9090:80
```

### Security Considerations:
- Forward ports only when necessary.
- Use RBAC to restrict access.
- Close the session when not in use.

---

## Key Differences
| Feature            | `kubectl proxy` | `kubectl port-forward` |
|--------------------|----------------|------------------------|
| Purpose           | API access      | Direct pod/service access |
| Use case         | Interacting with API server | Accessing internal services |
| Security Concern | Exposes API server | Exposes specific service |
| Scope             | Cluster-wide API | Single pod/service |


