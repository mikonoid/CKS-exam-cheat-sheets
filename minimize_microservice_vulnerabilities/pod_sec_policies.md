# Pod Security Policies (PSP)

## What is a Pod Security Policy (PSP)?
Pod Security Policies (PSPs) are cluster-wide policies that control the security settings of pods before they are admitted.

> ⚠️ **Deprecated in Kubernetes 1.21** and removed in 1.25. Use [Pod Security Admission (PSA)](https://kubernetes.io/docs/concepts/security/pod-security-admission/) instead.

## Enabling Pod Security Policies
PSPs are enabled using the `PodSecurityPolicy` admission controller:
```sh
--enable-admission-plugins=PodSecurityPolicy
```

## Example Pod Security Policy
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
  fsGroup:
    rule: MustRunAs
    ranges:
      - min: 1
        max: 65535
  volumes:
    - 'configMap'
    - 'emptyDir'
    - 'secret'
  hostNetwork: false
  hostIPC: false
  hostPID: false
```

## Key PSP Fields
| Field                     | Description |
|---------------------------|-------------|
| `privileged: false`       | Prevents running privileged containers |
| `allowPrivilegeEscalation: false` | Blocks privilege escalation (e.g., `setuid` binaries) |
| `runAsUser.rule`          | Enforces running as a non-root user |
| `seLinux.rule`            | Configures SELinux settings |
| `fsGroup.rule`            | Controls file system group ownership |
| `volumes`                 | Limits allowed volume types |
| `hostNetwork: false`      | Prevents pod from using host network |
| `hostPID: false`          | Prevents pod from using host PID namespace |

## Assigning a PSP to Users
To apply PSPs, create a **RoleBinding** to link it to a user/group/service account:
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: use-psp
roleRef:
  kind: ClusterRole
  name: psp-user
  apiGroup: rbac.authorization.k8s.io
subjects:
  - kind: ServiceAccount
    name: default
    namespace: my-namespace
```

## Replacing PSPs with Pod Security Admission (PSA)
PSPs are deprecated. Use **Pod Security Admission (PSA)** instead, which provides three security levels:
- **privileged** (unrestricted pods)
- **baseline** (minimally restrictive)
- **restricted** (strong security restrictions)

Example enforcing `restricted` PSA:
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: secure-namespace
  labels:
    pod-security.kubernetes.io/enforce: "restricted"
```

## Debugging PSP Issues
- Check available PSPs:
  ```sh
  kubectl get psp
  ```
- Check applied PSP for a pod:
  ```sh
  kubectl describe pod <pod-name>
  ```
- Check RoleBindings for PSP usage:
  ```sh
  kubectl get rolebindings --all-namespaces | grep PSP
  ```

---
**Reference**: [Kubernetes Pod Security Policies](https://kubernetes.io/docs/concepts/security/pod-security-policy/)
