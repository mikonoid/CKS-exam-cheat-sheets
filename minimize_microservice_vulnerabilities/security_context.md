```markdown
#  Security Context

## 1. Security Context Overview
Security Context defines privilege and access control settings for pods or containers in Kubernetes.

### Set Security Context in a Pod
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: secure-pod
spec:
  securityContext:
    runAsUser: 1000
    runAsGroup: 3000
    fsGroup: 2000
  containers:
  - name: secure-container
    image: busybox
    command: ["sleep", "3600"]
    securityContext:
      allowPrivilegeEscalation: false
      runAsNonRoot: true
      capabilities:
        drop:
          - ALL
      readOnlyRootFilesystem: true
```

## 2. Key Security Context Fields

| Field | Description |
|-------|-------------|
| `runAsUser` | Runs container as a specific user ID (UID) |
| `runAsGroup` | Specifies primary group ID (GID) |
| `fsGroup` | Defines file system group for volumes |
| `runAsNonRoot` | Ensures container does not run as root |
| `allowPrivilegeEscalation` | Prevents privilege escalation (e.g., via setuid binaries) |
| `privileged` | Grants full host privileges (should be `false`) |
| `capabilities` | Drops or adds Linux capabilities to containers |
| `readOnlyRootFilesystem` | Prevents writing to root filesystem |
| `seccompProfile` | Applies seccomp security profiles (e.g., `RuntimeDefault`) |

## 3. Restrict Privileges with PodSecurityPolicy (Deprecated) & Pod Security Admission

### Example: Enforce Security with Pod Security Admission (PSA)
```yaml
apiVersion: policy/v1beta1
kind: PodSecurityPolicy
metadata:
  name: restricted-psp
spec:
  privileged: false
  allowPrivilegeEscalation: false
  requiredDropCapabilities:
    - ALL
  runAsUser:
    rule: MustRunAsNonRoot
  seLinux:
    rule: RunAsAny
  supplementalGroups:
    rule: MustRunAs
    ranges:
      - min: 1000
        max: 2000
  fsGroup:
    rule: MustRunAs
    ranges:
      - min: 1000
        max: 2000
  readOnlyRootFilesystem: true
```

**Note**: PodSecurityPolicy (PSP) is deprecated; use Pod Security Admission (PSA) instead.

## 4. Enforce Security with Pod Security Standards (PSS)

### Example: Enforcing `restricted` Policy
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: secure-namespace
  labels:
    pod-security.kubernetes.io/enforce: "restricted"
    pod-security.kubernetes.io/audit: "baseline"
    pod-security.kubernetes.io/warn: "baseline"
```

## 5. Best Practices
- Always **use non-root users** (`runAsNonRoot: true`)
- **Disable privilege escalation** (`allowPrivilegeEscalation: false`)
- **Use read-only root filesystems** (`readOnlyRootFilesystem: true`)
- **Limit Linux capabilities** (`capabilities.drop: ALL`)
- **Apply seccomp profiles** (`seccompProfile: RuntimeDefault`)
- **Enforce security policies** with PSA or admission controllers
- **Use RBAC** to restrict unauthorized actions
- **Scan container images** for vulnerabilities before deployment
```

