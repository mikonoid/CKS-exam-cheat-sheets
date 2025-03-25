# Kubesec Scan

## What is Kubesec?
Kubesec is a security tool that analyzes Kubernetes YAML manifests for security risks and compliance issues. It helps identify weaknesses in Pod, Deployment, and other resource configurations.

## Why Use Kubesec for CKS?
 - Detects **misconfigurations** in Kubernetes resources.
 - Provides **risk scores** based on security best practices.
 - Ensures compliance with **CIS Benchmarks and Kubernetes security guidelines**.
 - Helps prevent **privilege escalation and insecure configurations**.

---
## Installing Kubesec
### 1. **Run Kubesec as a Standalone Tool**
```sh
curl -LO https://github.com/controlplaneio/kubesec/releases/latest/download/kubesec_linux_amd64
chmod +x kubesec_linux_amd64
mv kubesec_linux_amd64 /usr/local/bin/kubesec
```

### 2. **Use Kubesec as a Web Service**
```sh
curl -s -X POST --data-binary @deployment.yaml https://v2.kubesec.io/scan
```

---
## Scanning Kubernetes Manifests
### 1. **Scan a YAML File**
```sh
kubesec scan deployment.yaml
```

### 2. **Scan a Manifest from Standard Input**
```sh
cat deployment.yaml | kubesec scan -
```

### 3. **Scan a Running Pod**
```sh
kubectl get pod mypod -o yaml | kubesec scan -
```

---
## Understanding Kubesec Scores
- **Score ≥ 5** → Secure (Good security practices applied)
- **Score 0 to 4** → Moderate risk (Some security gaps detected)
- **Score < 0** → High risk (Critical misconfigurations present)

---
## Common Security Issues Detected by Kubesec
### 1. **Running as Root** (High Risk)
- Avoid running containers with root privileges.

#### ❌ Insecure Example:
```yaml
securityContext:
  runAsUser: 0
```

#### ✅ Secure Fix:
```yaml
securityContext:
  runAsNonRoot: true
  runAsUser: 1000
```

### 2. **Missing Read-Only Root Filesystem**
- Prevents unauthorized modifications to the container filesystem.

#### ❌ Insecure Example:
```yaml
securityContext:
  readOnlyRootFilesystem: false
```

#### ✅ Secure Fix:
```yaml
securityContext:
  readOnlyRootFilesystem: true
```

### 3. **Privileged Mode Enabled** (Very High Risk)
- Containers should not run in privileged mode.

#### ❌ Insecure Example:
```yaml
securityContext:
  privileged: true
```

#### ✅ Secure Fix:
```yaml
securityContext:
  privileged: false
```

### 4. **Allowing Dangerous Capabilities**
- Drop unnecessary Linux capabilities.

#### ❌ Insecure Example:
```yaml
securityContext:
  capabilities:
    add:
      - SYS_ADMIN
```

#### ✅ Secure Fix:
```yaml
securityContext:
  capabilities:
    drop:
      - ALL
```

### 5. **Lack of Network Restrictions**
- Use **NetworkPolicies** to restrict traffic.

#### ❌ Insecure Example (No restrictions):
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-all
spec:
  podSelector: {}
  ingress:
    - {}
```

#### ✅ Secure Fix (Deny all by default):
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny
spec:
  podSelector: {}
  policyTypes:
    - Ingress
    - Egress
```

---
## Best Practices for Secure Manifests
✅ Always **set non-root users** for containers.
✅ Enforce **read-only root filesystems**.
✅ Drop unnecessary **Linux capabilities**.
✅ Use **NetworkPolicies** to restrict access.
✅ Automate **Kubesec scans** in CI/CD pipelines.
✅ Regularly **update and review Kubernetes manifests** for security risks.

---
**Reference**: [Kubesec Documentation](https://kubesec.io/)
